# Séance 3 — TDD, intégration continue et pratiques avancées

!!! info "Objectifs de la séance"
    À l'issue de cette séance, vous serez capables de :

    - **Appliquer** le cycle TDD Red-Green-Refactor sur un kata complet
    - **Refactorer** du code en toute confiance grâce aux tests
    - **Écrire** des tests d'intégration avec une base de données en mémoire
    - **Configurer** un pipeline CI avec GitHub Actions
    - **Connaître** les pratiques avancées : analyse statique, tests de mutation, fuzzing

---

## 1. Test-Driven Development (TDD)

### Le cycle Red-Green-Refactor

Le TDD (Beck, 2002[^1]) inverse l'ordre habituel : on écrit le test **avant** le code.

[^1]: Beck, K. (2002). *Test-Driven Development: By Example*. Addison-Wesley. ISBN: 978-0321146533.

```
     ┌─────────────────────────────────┐
     │                                 │
     ▼                                 │
   RED ──────► GREEN ──────► REFACTOR ─┘
  (test        (code          (nettoyer
  échoue)      minimal)       le code)
```

1. **RED** — Écrire un test qui échoue (le comportement n'existe pas encore)
2. **GREEN** — Écrire le **minimum** de code pour que le test passe
3. **REFACTOR** — Améliorer le code sans changer le comportement (les tests garantissent la non-régression)

### Les 3 lois de Robert C. Martin

1. Ne pas écrire de code de production tant qu'un test unitaire ne l'exige pas
2. Ne pas écrire plus d'un test que ce qui est nécessaire pour échouer
3. Ne pas écrire plus de code de production que ce qui est nécessaire pour faire passer le test

### Pourquoi TDD ?

- **Conception émergente** : le code est conçu pour être testable dès le départ
- **Filet de sécurité** : chaque ligne de code est couverte par un test
- **Documentation vivante** : les tests décrivent le comportement attendu
- **Confiance pour refactorer** : on peut restructurer sans crainte

---

## 2. Refactoring

### Définition

Le **refactoring** (Fowler, 2018[^2]) consiste à modifier la structure interne du code **sans changer son comportement observable**. Les tests existants garantissent la non-régression.

[^2]: Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley. ISBN: 978-0134757599.

### Catalogue de refactorings courants

| Refactoring | Quand l'utiliser | Avant | Après |
|------------|------------------|-------|-------|
| **Rename Variable/Method** | Nom pas clair | `x = p * q` | `total = prix * quantite` |
| **Extract Method** | Méthode trop longue | Bloc de 50 lignes | 3 méthodes courtes |
| **Inline Method** | Méthode triviale qui obscurcit | `def get_x(): return self.x` | Accès direct |
| **Extract Variable** | Expression complexe | `if a > 0 and b < 10 and c:` | `est_valide = ...` |
| **Replace Magic Number** | Nombre sans signification | `if age < 18:` | `if age < AGE_MINIMUM:` |
| **Replace Conditional with Polymorphism** | `if/elif` sur un type | Cascade de `if` | Classes spécialisées |

### Exemple : Extract Method

Avant :

```python
def total_panier(self, panier, catalogue):
    sous_total = 0
    for isbn, qte in panier.articles.items():
        sous_total += catalogue[isbn].prix * qte

    if sous_total > 100:
        remise = sous_total * 0.10
    elif sous_total > 50:
        remise = sous_total * 0.05
    else:
        remise = 0

    if sous_total > 0 and sous_total < 35:
        frais = 4.99
    else:
        frais = 0

    return sous_total - remise + frais
```

Après refactoring :

```python
def total_panier(self, panier, catalogue):
    sous_total = self._sous_total(panier, catalogue)
    remise = self._appliquer_remise(sous_total)
    frais = self._frais_port(sous_total)
    return sous_total - remise + frais

def _sous_total(self, panier, catalogue):
    return sum(
        catalogue[isbn].prix * qte
        for isbn, qte in panier.articles.items()
    )

def _appliquer_remise(self, sous_total):
    if sous_total > 100:
        return sous_total * 0.10
    elif sous_total > 50:
        return sous_total * 0.05
    return 0

def _frais_port(self, sous_total):
    if 0 < sous_total < 35:
        return 4.99
    return 0
```

!!! tip "Règle d'or du refactoring"
    Ne **jamais** refactorer sans tests. Les tests sont votre filet de sécurité.

---

## 3. Tests d'intégration

### Différence avec les tests unitaires

| Aspect | Test unitaire | Test d'intégration |
|--------|--------------|-------------------|
| **Portée** | Une fonction/classe isolée | Plusieurs composants ensemble |
| **Dépendances** | Mockées/stubbées | Réelles (ou quasi-réelles) |
| **Vitesse** | Très rapide (ms) | Plus lent (s) |
| **Exemple** | `test_sauvegarder_livre` avec mock | `test_sauvegarder_puis_retrouver` avec vraie BDD |

### Base de données en mémoire

Pour tester les interactions avec une base de données sans installer un serveur, on utilise une base **en mémoire** :

- **Python** : SQLite avec `sqlite3.connect(":memory:")`
- **Java** : H2 avec `jdbc:h2:mem:test`

### Python — SQLite in-memory

```python
import sqlite3
from src.librairie import Livre


class LivreRepository:
    """Accès aux données des livres en base de données."""

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

### Fixture de base de données

```python
import pytest
import sqlite3

@pytest.fixture
def db():
    """Base de données SQLite en mémoire avec le schéma livres."""
    conn = sqlite3.connect(":memory:")
    conn.execute("""
        CREATE TABLE livres (
            isbn TEXT PRIMARY KEY,
            titre TEXT NOT NULL,
            prix REAL NOT NULL,
            stock INTEGER NOT NULL
        )
    """)
    yield conn
    conn.close()
```

### Test d'intégration

```python
def test_sauvegarder_puis_retrouver(db):
    repo = LivreRepository(db)
    livre = Livre("978-2-07-036024-1", "Le Petit Prince", 7.50, 100)

    repo.sauvegarder(livre)
    retrouve = repo.trouver("978-2-07-036024-1")

    assert retrouve is not None
    assert retrouve.titre == "Le Petit Prince"
    assert retrouve.prix == 7.50

def test_trouver_isbn_inexistant(db):
    repo = LivreRepository(db)

    resultat = repo.trouver("ISBN-ABSENT")

    assert resultat is None
```

### Java — H2 in-memory

```java
try (Connection conn = DriverManager.getConnection(
        "jdbc:h2:mem:test;MODE=PostgreSQL;DB_CLOSE_DELAY=-1")) {
    conn.createStatement().execute(
        "CREATE TABLE livres (isbn VARCHAR PRIMARY KEY, " +
        "titre VARCHAR, prix DECIMAL, stock INT)"
    );
    LivreRepository repo = new LivreRepository(conn);

    repo.sauvegarder(new Livre("978-2-07-036024-1",
        "Le Petit Prince", 7.50, 100));
    Livre retrouve = repo.trouver("978-2-07-036024-1");

    assertNotNull(retrouve);
    assertEquals("Le Petit Prince", retrouve.titre());
}
```

---

## 4. Intégration continue (CI)

### Qu'est-ce que la CI ?

L'intégration continue consiste à **exécuter automatiquement** les tests à chaque push ou pull request. Si les tests échouent, l'intégration est bloquée.

### GitHub Actions

GitHub Actions permet de définir des **workflows** dans des fichiers YAML sous `.github/workflows/`.

### Workflow minimal

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install pytest pytest-cov
      - run: pytest --cov=src --cov-branch --cov-fail-under=85
```

### Workflow avec matrice

Pour tester sur plusieurs versions de Python et systèmes d'exploitation :

```yaml
jobs:
  tests:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
        python-version: ['3.11', '3.12']
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install pytest pytest-cov
      - run: pytest --cov=src --cov-branch --cov-fail-under=85
```

### Badge de statut

Ajoutez un badge dans votre `README.md` :

```markdown
![CI](https://github.com/<user>/<repo>/actions/workflows/ci.yml/badge.svg)
```

---

## 5. Analyse statique

L'analyse statique détecte les problèmes **sans exécuter** le code :

| Outil | Langage | Détecte |
|-------|---------|---------|
| **ruff** | Python | Style, imports, erreurs courantes (très rapide) |
| **pylint** | Python | Conventions, complexité, code mort |
| **mypy** | Python | Erreurs de typage statique |
| **SpotBugs** | Java | Patterns de bugs connus |
| **Checkstyle** | Java | Conventions de codage |
| **ESLint** | JavaScript | Erreurs et conventions JS |
| **SonarQube** | Multi-langage | Qualité globale, dette technique |

```bash
# Python
pip install ruff mypy
ruff check src/
mypy src/
```

---

## 6. Tests de mutation

### Principe

Les tests de mutation (DeMillo et al., 1978[^3]) vérifient que vos tests **détectent réellement** les bugs. L'outil injecte des mutations dans le code (ex : `>` → `>=`, `+` → `-`) et vérifie que les tests échouent.

[^3]: DeMillo, R.A., Lipton, R.J. & Sayward, F.G. (1978). Hints on Test Data Selection: Help for the Practicing Programmer. *IEEE Computer*, 11(4), 34–41. DOI: [10.1109/C-M.1978.218136](https://doi.org/10.1109/C-M.1978.218136)

- **Mutant tué** : les tests détectent la mutation (bien)
- **Mutant survivant** : les tests ne voient pas la différence (tests trop faibles)

### Outils

| Outil | Langage | Commande |
|-------|---------|----------|
| **mutmut** | Python | `mutmut run --paths-to-mutate=src/` |
| **PIT** (Pitest) | Java | Plugin Maven |

```bash
pip install mutmut
mutmut run --paths-to-mutate=src/
mutmut results
```

---

## 7. Fuzzing et tests property-based

### Fuzzing

Le fuzzing génère des entrées **aléatoires** ou **semi-aléatoires** pour découvrir des crashes inattendus.

### Hypothesis (Python)

Hypothesis génère automatiquement des données de test basées sur des **stratégies** :

```python
from hypothesis import given, strategies as st
from src.librairie import appliquer_tva

@given(
    prix=st.floats(min_value=0, max_value=1_000_000, allow_nan=False),
    taux=st.floats(min_value=0, max_value=1, allow_nan=False)
)
def test_tva_jamais_inferieur_au_prix_ht(prix, taux):
    resultat = appliquer_tva(prix, taux)
    assert resultat >= prix
```

!!! tip "Propriétés vs. exemples"
    Les tests classiques vérifient des **exemples** (`appliquer_tva(100, 0.20) == 120`).
    Les tests property-based vérifient des **propriétés** (`pour tout prix et taux valides, le TTC ≥ le HT`). Hypothesis génère des centaines de cas automatiquement.

---

## Récapitulatif de la séance

| Concept | Points clés |
|---------|-------------|
| TDD | Red → Green → Refactor. Test d'abord, code ensuite (Beck, 2002) |
| Refactoring | Modifier la structure sans changer le comportement (Fowler, 2018) |
| Tests d'intégration | Composants réels ensemble, BDD en mémoire (SQLite, H2) |
| CI | GitHub Actions, workflow YAML, matrice OS×Python, badge |
| Analyse statique | ruff, mypy, pylint, SonarQube — sans exécution |
| Mutation | mutmut / PIT — vérifier que les tests détectent les bugs |
| Fuzzing | Hypothesis — propriétés + données aléatoires |

---

*Passez aux travaux pratiques : [TP5 — Kata FizzBuzz en TDD](tp5-tdd-fizzbuzz.md) puis [TP6 — CI avec GitHub Actions](tp6-ci-github-actions.md).*
