# Ressources

## Bibliographie du cours

### Ouvrages fondamentaux

| Réf. | Ouvrage | Usage dans le cours |
|------|---------|---------------------|
| [1] | Beck, K. (2002). *Test-Driven Development: By Example*. Addison-Wesley. ISBN: 978-0321146533 | TDD, cycle Red-Green-Refactor (Séance 3) |
| [2] | Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley. ISBN: 978-0134757599 | Catalogue de refactorings (Séance 3) |
| [3] | Cohn, M. (2009). *Succeeding with Agile*. Addison-Wesley. ISBN: 978-0321579362 | Pyramide des tests (Séance 1) |
| [4] | Myers, G.J. (1979). *The Art of Software Testing*. Wiley. ISBN: 978-0471043287 | Partitions d'équivalence (Séance 2) |
| [5] | Beizer, B. (1990). *Software Testing Techniques*. 2nd ed. Van Nostrand Reinhold. ISBN: 978-0442206727 | Valeurs limites (Séance 2) |
| [6] | Meszaros, G. (2007). *xUnit Test Patterns*. Addison-Wesley. ISBN: 978-0131495050 | Taxonomie des doublures de test (Séance 2) |

### Articles scientifiques

| Réf. | Article | DOI |
|------|---------|-----|
| [7] | Boehm, B. & Basili, V. (2001). Software Defect Reduction Top 10 List. *IEEE Computer*, 34(1), 135–137. | [10.1109/2.962984](https://doi.org/10.1109/2.962984) |
| [8] | DeMillo, R.A., Lipton, R.J. & Sayward, F.G. (1978). Hints on Test Data Selection. *IEEE Computer*, 11(4), 34–41. | [10.1109/C-M.1978.218136](https://doi.org/10.1109/C-M.1978.218136) |

### Normes et référentiels

| Réf. | Norme | Usage |
|------|-------|-------|
| [9] | ISO/IEC 25010:2011 — Systems and software Quality Requirements and Evaluation (SQuaRE) | Modèle de qualité (Séance 1) |
| [10] | ISTQB — International Software Testing Qualifications Board, *Glossaire* | Vocabulaire du test (Séance 1) |
| [11] | IEEE (2014). *SWEBOK v3.0 — Software Engineering Body of Knowledge* | Référentiel de compétences |

---

## Documentation des outils

### Python

| Outil | Documentation |
|-------|---------------|
| pytest | [docs.pytest.org](https://docs.pytest.org/) |
| pytest-cov | [pytest-cov.readthedocs.io](https://pytest-cov.readthedocs.io/) |
| coverage.py | [coverage.readthedocs.io](https://coverage.readthedocs.io/) |
| unittest.mock | [docs.python.org/3/library/unittest.mock.html](https://docs.python.org/3/library/unittest.mock.html) |
| Hypothesis | [hypothesis.readthedocs.io](https://hypothesis.readthedocs.io/) |
| mutmut | [mutmut.readthedocs.io](https://mutmut.readthedocs.io/) |
| ruff | [docs.astral.sh/ruff](https://docs.astral.sh/ruff/) |
| mypy | [mypy.readthedocs.io](https://mypy.readthedocs.io/) |

### Java

| Outil | Documentation |
|-------|---------------|
| JUnit 5 | [junit.org/junit5/docs/current/user-guide](https://junit.org/junit5/docs/current/user-guide/) |
| Mockito | [site.mockito.org](https://site.mockito.org/) |
| JaCoCo | [www.jacoco.org/jacoco](https://www.jacoco.org/jacoco/) |
| Maven | [maven.apache.org](https://maven.apache.org/) |

### CI/CD

| Outil | Documentation |
|-------|---------------|
| GitHub Actions | [docs.github.com/en/actions](https://docs.github.com/en/actions) |
| SonarQube | [docs.sonarsource.com](https://docs.sonarsource.com/) |

---

## Code complet du fil rouge

### `src/librairie.py`

```python
from dataclasses import dataclass


@dataclass
class Livre:
    isbn: str
    titre: str
    prix: float
    stock: int


class LibraireService:
    def __init__(self, catalogue: dict[str, Livre]):
        self.catalogue = catalogue

    def get_prix(self, isbn: str) -> float:
        return self.catalogue[isbn].prix

    def est_disponible(self, isbn: str, quantite: int = 1) -> bool:
        livre = self.catalogue.get(isbn)
        return livre is not None and livre.stock >= quantite

    def calculer_total(self, isbn: str, quantite: int) -> float:
        livre = self.catalogue[isbn]
        return livre.prix * quantite


def valider_age(age: int) -> bool:
    return 13 <= age <= 120


def appliquer_tva(prix_ht: float, taux: float) -> float:
    return prix_ht * (1 + taux)


def reduction_etudiant(age: int, est_etudiant: bool) -> float:
    if not est_etudiant:
        return 0.0
    if 13 <= age <= 25:
        return 0.50
    if 26 <= age <= 30:
        return 0.30
    return 0.0
```

### `src/panier.py`

```python
from enum import Enum
from dataclasses import dataclass, field
from src.librairie import Livre


class EtatPanier(Enum):
    VIDE = "vide"
    ACTIF = "actif"
    VALIDE = "valide"
    PAYE = "paye"


@dataclass
class Panier:
    articles: dict[str, int] = field(default_factory=dict)
    etat: EtatPanier = EtatPanier.VIDE

    def ajouter(self, livre: Livre, quantite: int = 1) -> None:
        if self.etat in (EtatPanier.VALIDE, EtatPanier.PAYE):
            raise RuntimeError("Panier non modifiable")
        if quantite <= 0:
            raise ValueError("Quantité doit être > 0")
        self.articles[livre.isbn] = self.articles.get(livre.isbn, 0) + quantite
        self.etat = EtatPanier.ACTIF

    def retirer(self, isbn: str, quantite: int = 1) -> None:
        if self.etat in (EtatPanier.VALIDE, EtatPanier.PAYE):
            raise RuntimeError("Panier non modifiable")
        if quantite <= 0:
            raise ValueError("Quantité doit être > 0")
        if isbn not in self.articles:
            raise KeyError(f"ISBN {isbn} absent du panier")
        if quantite > self.articles[isbn]:
            raise ValueError("Quantité à retirer > quantité dans le panier")
        self.articles[isbn] -= quantite
        if self.articles[isbn] == 0:
            del self.articles[isbn]
        if not self.articles:
            self.etat = EtatPanier.VIDE

    def vider(self) -> None:
        if self.etat in (EtatPanier.VALIDE, EtatPanier.PAYE):
            raise RuntimeError("Panier non modifiable")
        self.articles.clear()
        self.etat = EtatPanier.VIDE

    def valider(self) -> None:
        if self.etat != EtatPanier.ACTIF:
            raise RuntimeError(
                f"Validation impossible depuis l'état {self.etat.value}"
            )
        self.etat = EtatPanier.VALIDE

    def payer(self) -> None:
        if self.etat != EtatPanier.VALIDE:
            raise RuntimeError(
                f"Paiement impossible depuis l'état {self.etat.value}"
            )
        self.etat = EtatPanier.PAYE

    def nombre_articles(self) -> int:
        return sum(self.articles.values())
```

### `src/fizzbuzz.py`

```python
def fizzbuzz(n: int) -> str:
    sortie = ""
    if n % 3 == 0:
        sortie += "Fizz"
    if n % 5 == 0:
        sortie += "Buzz"
    return sortie or str(n)
```

### `src/repository.py`

```python
import sqlite3
from src.librairie import Livre


class LivreRepository:
    def __init__(self, conn: sqlite3.Connection):
        self.conn = conn

    def sauvegarder(self, livre: Livre) -> None:
        self.conn.execute(
            "INSERT INTO livres (isbn, titre, prix, stock) VALUES (?, ?, ?, ?)",
            (livre.isbn, livre.titre, livre.prix, livre.stock)
        )
        self.conn.commit()

    def trouver(self, isbn: str) -> Livre | None:
        cur = self.conn.execute(
            "SELECT isbn, titre, prix, stock FROM livres WHERE isbn = ?",
            (isbn,)
        )
        row = cur.fetchone()
        return Livre(*row) if row else None
```

---

## Cas industriels étudiés en cours

| Incident | Année | Impact | Leçon |
|----------|-------|--------|-------|
| **Therac-25** | 1985–87 | 6 patients irradiés, 3 décès | Tests de conditions de course |
| **Ariane 5 vol 501** | 1996 | Fusée détruite (370 M$) | Tests de dépassement d'entier |
| **Knight Capital** | 2012 | 440 M$ perdus en 45 min | Tests de déploiement |
| **Heartbleed** | 2014 | Fuite de données massive | Vérification de bornes |
| **Boeing 737 MAX** | 2018–19 | 346 décès | Tests système insuffisants |
| **Toyota UA** | 2009–11 | Rappel de 9 millions de véhicules | Complexité logicielle non maîtrisée |

---

*Retour à l'[accueil](index.md).*
