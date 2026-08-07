# TP1 — Premiers tests avec pytest et le pattern AAA

!!! info "Informations pratiques"
    **Durée** : 1h30 · **Langage** : Python 3.12+ · **Outils** : pytest 8.x
    
    **Objectifs** :

    - Installer et configurer pytest
    - Écrire des tests structurés selon le pattern AAA
    - Utiliser la paramétrisation et `pytest.approx`
    - Tester le module `librairie.py` du fil rouge

!!! danger "Comment utiliser les indices — À lire avant de commencer"
    Chaque exercice propose **2 niveaux d'aide imbriqués** :

    - **Indice 1 — Direction** : ouvrez-le après avoir cherché seul au moins 5 minutes.
    - **Indice 2 — Approche** : ouvrez-le si l'indice 1 ne suffit pas et après 5 minutes supplémentaires.

    **Les corrigés complets sont fournis dans un document séparé, distribué après le rendu du TP.** Cette organisation vous garantit un apprentissage authentique : impossible de "regarder la réponse" avant d'avoir vraiment cherché.

---

## Étape 0 — Mise en place du projet

### 0.1 Créer l'arborescence

Créez la structure suivante dans votre répertoire de travail :

```
librairie-en-ligne/
├── src/
│   └── librairie.py
└── tests/
    └── test_librairie.py
```

??? tip "Commandes terminal"
    ```bash
    mkdir -p librairie-en-ligne/src librairie-en-ligne/tests
    cd librairie-en-ligne
    touch src/__init__.py tests/__init__.py
    touch src/librairie.py tests/test_librairie.py
    ```

### 0.2 Installer pytest

```bash
pip install pytest
```

Vérifiez l'installation :

```bash
pytest --version
```

### 0.3 Code de départ — `src/librairie.py`

Copiez ce code dans `src/librairie.py` :

```python
from dataclasses import dataclass


@dataclass
class Livre:
    isbn: str
    titre: str
    prix: float
    stock: int


class LibraireService:
    """Service de gestion du catalogue de la librairie."""

    def __init__(self, catalogue: dict[str, Livre]):
        self.catalogue = catalogue

    def get_prix(self, isbn: str) -> float:
        """Retourne le prix d'un livre par son ISBN.

        Raises:
            KeyError: si l'ISBN n'existe pas dans le catalogue.
        """
        return self.catalogue[isbn].prix

    def est_disponible(self, isbn: str, quantite: int = 1) -> bool:
        """Vérifie si un livre est disponible en quantité suffisante."""
        livre = self.catalogue.get(isbn)
        return livre is not None and livre.stock >= quantite

    def calculer_total(self, isbn: str, quantite: int) -> float:
        """Calcule le prix total pour un nombre d'exemplaires donné."""
        livre = self.catalogue[isbn]
        return livre.prix * quantite


def valider_age(age: int) -> bool:
    """Valide qu'un âge est dans la plage acceptable [13, 120]."""
    return 13 <= age <= 120


def appliquer_tva(prix_ht: float, taux: float) -> float:
    """Applique un taux de TVA à un prix hors taxe."""
    return prix_ht * (1 + taux)
```

---

## Étape 1 — Votre premier test

### 1.1 Écrire un test simple

Ouvrez `tests/test_librairie.py` et écrivez votre premier test :

```python
from src.librairie import Livre, LibraireService


def test_get_prix_livre_existant():
    # Arrange
    catalogue = {
        "978-2-07-036024-1": Livre(
            isbn="978-2-07-036024-1",
            titre="Le Petit Prince",
            prix=7.50,
            stock=100
        )
    }
    service = LibraireService(catalogue)

    # Act
    prix = service.get_prix("978-2-07-036024-1")

    # Assert
    assert prix == 7.50
```

### 1.2 Lancer le test

```bash
pytest tests/test_librairie.py -v
```

Résultat attendu :

```
tests/test_librairie.py::test_get_prix_livre_existant PASSED
```

!!! success "Bravo !"
    Vous venez d'écrire et de faire passer votre premier test unitaire. Notez la structure AAA claire avec les commentaires `# Arrange`, `# Act`, `# Assert`.

---

## Étape 2 — Tester `calculer_total`

### 2.1 Cas nominal

Écrivez un test qui vérifie que `calculer_total` retourne `prix × quantité` :

!!! warning "Règle d'apprentissage"
    N'ouvrez l'indice 2 qu'après avoir essayé au moins 5 minutes avec l'indice 1. Les corrigés complets sont dans un document séparé distribué après le rendu du TP.

??? tip "Indice 1 — Direction (essayez d'abord seul)"
    Reprenez la structure AAA du test précédent. Ce qui change : la méthode appelée (`calculer_total` au lieu de `get_prix`), un paramètre `quantite`, et la valeur attendue.

    ??? tip "Indice 2 — Approche (si toujours bloqué)"
        Créez un catalogue avec un livre à 7.50 €. Appelez `service.calculer_total(isbn, quantite=2)`. Comparez le résultat à `15.0`.

### 2.2 Cas d'erreur — ISBN inexistant

Que se passe-t-il si on demande le total pour un ISBN qui n'existe pas ?

??? tip "Indice 1 — Direction"
    Pytest fournit un mécanisme dédié pour tester qu'une exception est bien levée. Cherchez `pytest.raises` dans la documentation.

    ??? tip "Indice 2 — Approche"
        Utilisez `with pytest.raises(KeyError):` puis appelez la méthode qui doit lever l'exception à l'intérieur du bloc.

---

## Étape 3 — Tester `est_disponible`

Écrivez une série de tests pour la méthode `est_disponible`. Vous devez couvrir les cas suivants :

| # | Scénario | Résultat attendu |
|---|----------|------------------|
| 1 | Livre en stock (stock=10, quantité=1) | `True` |
| 2 | Stock exact (stock=5, quantité=5) | `True` |
| 3 | Stock insuffisant (stock=2, quantité=5) | `False` |
| 4 | ISBN inexistant | `False` |
| 5 | Stock à zéro | `False` |

??? tip "Indice 1 — Direction"
    Un test = un scénario du tableau ci-dessus. Nommez chaque test explicitement (`test_est_disponible_stock_suffisant`, `test_est_disponible_stock_exact`, etc.).

    ??? tip "Indice 2 — Approche"
        Pour chaque test, créez un `catalogue` avec un `Livre` dont le `stock` correspond au scénario, instanciez le `LibraireService`, puis assertez le résultat de `est_disponible`. Pensez à un test sans le livre du tout pour le cas "ISBN inexistant".

---

## Étape 4 — Paramétrisation avec `@pytest.mark.parametrize`

### 4.1 Pourquoi paramétrer ?

Vous avez remarqué que les tests de l'étape 3 se ressemblent beaucoup. La paramétrisation permet de factoriser le code de test en définissant les cas sous forme de tableau.

### 4.2 Paramétrer `valider_age`

La fonction `valider_age` accepte les âges dans l'intervalle `[13, 120]`. Écrivez un test paramétré couvrant les cas suivants :

| Âge | Attendu | Catégorie |
|-----|---------|-----------|
| 12 | `False` | Juste en dessous de la borne min |
| 13 | `True` | Borne minimale |
| 25 | `True` | Valeur nominale |
| 120 | `True` | Borne maximale |
| 121 | `False` | Juste au-dessus de la borne max |
| 0 | `False` | Valeur extrême basse |
| -1 | `False` | Valeur négative |

??? tip "Indice 1 — Direction"
    Le décorateur `@pytest.mark.parametrize` prend une chaîne de noms de paramètres et une liste de tuples. Chaque tuple = un scénario du tableau.

    ??? tip "Indice 2 — Approche"
        Le décorateur prend deux arguments : (1) une chaîne `"nom1, nom2"` déclarant les paramètres, (2) une liste de tuples, un tuple par ligne du tableau. La fonction reçoit ensuite les paramètres et fait une seule assertion.

---

## Étape 5 — `pytest.approx` pour la TVA

### 5.1 Le problème des flottants

Essayez ceci dans un interpréteur Python :

```python
>>> 0.1 + 0.2 == 0.3
False
```

Les nombres à virgule flottante (IEEE 754) ne représentent pas tous les réels exactement. Pour les comparaisons, pytest fournit `pytest.approx`.

### 5.2 Test paramétré de `appliquer_tva`

Écrivez un test paramétré pour `appliquer_tva` avec les cas suivants :

| Prix HT | Taux | Attendu TTC |
|---------|------|-------------|
| 100.0 | 0.20 | 120.0 |
| 100.0 | 0.055 | 105.5 |
| 0.0 | 0.20 | 0.0 |
| 50.0 | 0.0 | 50.0 |
| 9.99 | 0.20 | 11.988 |

??? tip "Indice 1 — Direction"
    Réutilisez `@pytest.mark.parametrize` mais avec 3 paramètres cette fois. L'assertion doit utiliser une comparaison **tolérante** aux flottants.

    ??? tip "Indice 2 — Approche"
        Remplacez `assert result == attendu` par `assert result == pytest.approx(attendu)`. Ne comparez jamais des flottants avec `==` strict.

---

## Étape 6 — Fixtures pytest (bonus)

### 6.1 Qu'est-ce qu'une fixture ?

Une **fixture** est une fonction qui fournit des données ou un contexte réutilisable pour les tests. Elle remplace le copier-coller du code Arrange.

### 6.2 Créer une fixture pour le catalogue

```python
import pytest
from src.librairie import Livre, LibraireService

@pytest.fixture
def catalogue():
    """Fixture fournissant un catalogue avec deux livres."""
    return {
        "978-2-07-036024-1": Livre(
            "978-2-07-036024-1", "Le Petit Prince", 7.50, 100
        ),
        "978-2-07-040850-9": Livre(
            "978-2-07-040850-9", "L'Étranger", 6.90, 50
        ),
    }

@pytest.fixture
def service(catalogue):
    """Fixture fournissant un LibraireService initialisé."""
    return LibraireService(catalogue)
```

### 6.3 Utiliser les fixtures dans les tests

```python
def test_get_prix_petit_prince(service):
    assert service.get_prix("978-2-07-036024-1") == 7.50

def test_get_prix_etranger(service):
    assert service.get_prix("978-2-07-040850-9") == 6.90

def test_est_disponible_petit_prince(service):
    assert service.est_disponible("978-2-07-036024-1", quantite=50) is True
```

!!! tip "Avantage des fixtures"
    Le code de préparation est écrit **une seule fois** et partagé par tous les tests qui en ont besoin. Si le catalogue change, une seule modification suffit.

---

## Vérification finale

Lancez toute la suite de tests :

```bash
pytest tests/test_librairie.py -v
```

!!! success "Critères de réussite"
    - [ ] Tous les tests passent (vert)
    - [ ] Chaque test suit le pattern AAA
    - [ ] Les noms de tests sont explicites
    - [ ] `valider_age` est testé avec `@pytest.mark.parametrize` (7+ cas)
    - [ ] `appliquer_tva` utilise `pytest.approx`
    - [ ] Au moins une fixture est définie et utilisée

---

*Continuez avec le [TP2 — JUnit 5](tp2-junit5.md) pour découvrir les tests unitaires en Java.*
