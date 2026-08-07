# TP6 — Intégration continue avec GitHub Actions

!!! info "Informations pratiques"
    **Durée** : 1h · **Outils** : Git, GitHub, GitHub Actions
    
    **Objectifs** :

    - Créer un dépôt GitHub pour le projet
    - Configurer un workflow CI avec GitHub Actions
    - Déclencher automatiquement les tests à chaque push
    - Ajouter un badge de statut au README

!!! info "Nature de ce TP"
    Ce TP est majoritairement procédural : suivez les étapes dans l'ordre. Chaque commande et fichier de configuration est fourni parce qu'il n'existe pas d'apprentissage à réinventer une syntaxe YAML propriétaire. **L'apprentissage porte sur la compréhension de chaque section**, pas sur sa réécriture aveugle.

    Pour chaque étape, essayez d'abord de **prédire** ce que va faire la commande avant de la lancer, puis vérifiez sur GitHub.

---

## Étape 0 — Prérequis

Assurez-vous d'avoir :

- Un compte [GitHub](https://github.com)
- Git installé et configuré (`git config --global user.name "..." && git config --global user.email "..."`)
- Le projet `librairie-en-ligne/` avec les tests des TP précédents

---

## Étape 1 — Initialiser le dépôt Git

### 1.1 Créer le dépôt local

```bash
cd librairie-en-ligne/
git init
```

### 1.2 Créer un `.gitignore`

```bash
cat > .gitignore << 'EOF'
__pycache__/
*.pyc
.pytest_cache/
htmlcov/
.coverage
*.egg-info/
dist/
build/
.mypy_cache/
EOF
```

### 1.3 Créer un `requirements.txt`

```bash
cat > requirements.txt << 'EOF'
pytest>=8.0
pytest-cov>=5.0
EOF
```

### 1.4 Premier commit

```bash
git add .
git commit -m "feat: projet librairie avec tests unitaires"
```

### 1.5 Créer le dépôt distant sur GitHub

1. Allez sur [github.com/new](https://github.com/new)
2. Nom : `librairie-en-ligne`
3. Visibilité : Public ou Private
4. Ne cochez **aucune** option d'initialisation (pas de README, pas de .gitignore)
5. Cliquez **Create repository**

```bash
git remote add origin https://github.com/<votre-username>/librairie-en-ligne.git
git branch -M main
git push -u origin main
```

---

## Étape 2 — Créer le workflow CI

### 2.1 Créer le fichier workflow

```bash
mkdir -p .github/workflows
```

Créez le fichier `.github/workflows/ci.yml` :

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
      - name: Checkout du code
        uses: actions/checkout@v4

      - name: Installer Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Installer les dépendances
        run: pip install -r requirements.txt

      - name: Lancer les tests avec couverture
        run: pytest --cov=src --cov-branch --cov-fail-under=85 -v
```

### 2.2 Comprendre le workflow

| Section | Rôle |
|---------|------|
| `name: CI` | Nom affiché dans l'onglet Actions de GitHub |
| `on: push/pull_request` | Déclenché à chaque push sur `main` ou ouverture de PR |
| `runs-on: ubuntu-latest` | Machine virtuelle Ubuntu utilisée |
| `actions/checkout@v4` | Clone le dépôt dans la VM |
| `actions/setup-python@v5` | Installe la version de Python spécifiée |
| `pip install` | Installe les dépendances |
| `pytest --cov...` | Lance les tests avec couverture et seuil à 85 % |

### 2.3 Pousser et observer

```bash
git add .github/
git commit -m "ci: ajouter workflow GitHub Actions"
git push
```

Allez sur votre dépôt GitHub → onglet **Actions**. Vous verrez le workflow s'exécuter.

??? tip "Que faire si le workflow échoue ?"
    1. Cliquez sur le run en échec
    2. Consultez les logs de l'étape qui a échoué
    3. Corrigez localement, commitez et poussez à nouveau
    4. Le workflow se relancera automatiquement

---

## Étape 3 — Workflow avec matrice

Pour tester votre code sur plusieurs environnements, utilisez une **stratégie matricielle** :

```yaml
name: CI Matrix

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

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

      - run: pip install -r requirements.txt

      - run: pytest --cov=src --cov-branch --cov-fail-under=85 -v
```

!!! note "4 exécutions parallèles"
    La matrice `2 OS × 2 Python` génère **4 jobs** parallèles :

    - Ubuntu + Python 3.11
    - Ubuntu + Python 3.12
    - macOS + Python 3.11
    - macOS + Python 3.12

??? tip "Indice"
    Remplacez le contenu de `.github/workflows/ci.yml` par la version matricielle et poussez.

---

## Étape 4 — Ajouter un badge de statut

### 4.1 Créer le README

Créez un fichier `README.md` à la racine :

```markdown
# Librairie en ligne

![CI](https://github.com/<votre-username>/librairie-en-ligne/actions/workflows/ci.yml/badge.svg)

Projet fil rouge du cours **Qualité et Tests Logiciels** — EFREI Paris.

## Lancer les tests

```bash
pip install -r requirements.txt
pytest --cov=src --cov-branch -v
```
```

### 4.2 Pousser

```bash
git add README.md
git commit -m "docs: ajouter README avec badge CI"
git push
```

Le badge affiche automatiquement :

- **passing** (vert) si le dernier workflow a réussi
- **failing** (rouge) si le dernier workflow a échoué

---

## Étape 5 — Exercice : provoquer un échec CI

### 5.1 Introduire un bug volontaire

Modifiez `src/librairie.py` pour introduire un bug :

```python
def calculer_total(self, isbn: str, quantite: int) -> float:
    livre = self.catalogue[isbn]
    return livre.prix + quantite  # Bug : + au lieu de *
```

### 5.2 Pousser et observer

```bash
git add src/librairie.py
git commit -m "test: introduire un bug volontaire"
git push
```

Observez dans l'onglet **Actions** : le workflow doit échouer. Les tests détectent le bug.

### 5.3 Corriger et vérifier

Corrigez le bug, commitez et poussez :

```bash
# Corriger : remettre * au lieu de +
git add src/librairie.py
git commit -m "fix: corriger calcul du total (retour à la multiplication)"
git push
```

Le workflow repasse au vert.

!!! success "La CI a joué son rôle"
    Le pipeline CI a détecté automatiquement le bug introduit. En conditions réelles, cela empêcherait un code défectueux d'être fusionné dans la branche principale.

---

## Étape 6 — Exercice bonus : Pull Request

### 6.1 Créer une branche

```bash
git checkout -b feature/reduction-etudiant
```

### 6.2 Ajouter une fonctionnalité

Ajoutez dans `src/librairie.py` :

```python
def reduction_etudiant(age: int, est_etudiant: bool) -> float:
    """Calcule le pourcentage de réduction pour un étudiant.

    - Étudiant et 13-25 ans : 50 %
    - Étudiant et 26-30 ans : 30 %
    - Sinon : 0 %
    """
    if not est_etudiant:
        return 0.0
    if 13 <= age <= 25:
        return 0.50
    if 26 <= age <= 30:
        return 0.30
    return 0.0
```

### 6.3 Écrire les tests

Ajoutez dans `tests/test_librairie.py` :

```python
from src.librairie import reduction_etudiant

@pytest.mark.parametrize("age, est_etudiant, attendu", [
    (20, True, 0.50),
    (13, True, 0.50),
    (25, True, 0.50),
    (26, True, 0.30),
    (30, True, 0.30),
    (31, True, 0.0),
    (20, False, 0.0),
    (12, True, 0.0),
])
def test_reduction_etudiant(age, est_etudiant, attendu):
    assert reduction_etudiant(age, est_etudiant) == attendu
```

### 6.4 Pousser et créer la PR

```bash
git add .
git commit -m "feat: ajouter réduction étudiant avec tests"
git push -u origin feature/reduction-etudiant
```

Sur GitHub, créez une **Pull Request** vers `main`. Le workflow CI se lance automatiquement sur la PR.

---

## Vérification finale

!!! success "Critères de réussite"
    - [ ] Dépôt GitHub créé avec le code du projet
    - [ ] Workflow CI fonctionnel (`.github/workflows/ci.yml`)
    - [ ] Badge de statut visible dans le README
    - [ ] Le workflow a été vu en échec puis en succès (exercice du bug)
    - [ ] La matrice OS × Python fonctionne (4 jobs)
    - [ ] (Bonus) PR créée avec la CI qui tourne

---

*Retour au [cours de la Séance 3](index.md) ou à l'[accueil](../index.md).*
