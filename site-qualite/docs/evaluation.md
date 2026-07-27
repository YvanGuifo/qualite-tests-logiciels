# Évaluation

## Modalités

| Composante | Poids | Séance | Format |
|------------|-------|--------|--------|
| **TP noté** | 40 % | Séance 2 | Test du module Panier, individuel, 1h |
| **QCM final** | 60 % | Séance 3 | 30–40 questions, 90 min |

---

## TP noté — Grille de notation

Le TP noté porte sur l'écriture d'une suite de tests pour un module fourni le jour de l'épreuve (similaire au Panier du fil rouge).

| Critère | Points | Détails |
|---------|--------|---------|
| **Structure AAA et nommage** | 4 pts | Chaque test suit Arrange-Act-Assert. Noms explicites : `test_methode_scenario_resultat` |
| **Cas nominaux et limites** | 5 pts | Scénarios fonctionnels couverts, valeurs limites testées (bornes incluses/exclues) |
| **Paramétrisation** | 3 pts | Usage correct de `@pytest.mark.parametrize` pour factoriser les cas similaires |
| **Détection de défauts** | 4 pts | Les tests détectent les bugs intentionnellement introduits dans le code fourni |
| **Couverture ≥ 85 %** | 3 pts | Couverture de branches mesurée par `pytest --cov --cov-branch` |
| **Respect des principes FIRST** | 1 pt | Tests rapides, indépendants, répétables, auto-validants |
| **Total** | **20 pts** | |

### Conseils pour le TP noté

!!! tip "Stratégie recommandée"
    1. **Lisez** d'abord le code fourni et sa documentation (10 min)
    2. **Identifiez** les cas nominaux, les cas d'erreur et les valeurs limites (5 min)
    3. **Écrivez** les tests cas par cas en commençant par les cas nominaux (30 min)
    4. **Mesurez** la couverture et complétez les branches manquantes (10 min)
    5. **Vérifiez** le nommage et la structure AAA (5 min)

---

## QCM final

### Format

- **30 à 40 questions** à choix multiples
- **90 minutes**
- Répartition par niveau de Bloom :
    - 30 % — **Comprendre** (définitions, concepts, distinctions)
    - 50 % — **Appliquer** (lire du code de test, identifier le résultat)
    - 20 % — **Analyser** (diagnostiquer un test défaillant, choisir la bonne technique)

### Thèmes couverts

| Séance | Thèmes |
|--------|--------|
| 1 | ISO 25010, chaîne erreur→défaut→panne, pyramide des tests, FIRST, AAA, pytest, JUnit 5 |
| 2 | Partitions d'équivalence, valeurs limites, tables de décision, transitions d'états, couverture C0/C1, mocks |
| 3 | TDD (cycle, lois), refactoring, tests d'intégration, CI/CD, analyse statique, mutation testing |

### Exemples de questions types

!!! example "Niveau Comprendre"
    *Quelle est la différence entre un défaut et une panne selon l'ISTQB ?*

    A. Un défaut est dans le code, une panne est observable à l'exécution ✅

    B. Un défaut est une erreur humaine, une panne est dans le code

    C. Il n'y a pas de différence

    D. Un défaut est toujours détecté par les tests

!!! example "Niveau Appliquer"
    *Quel est le résultat de ce test ?*
    ```python
    def test_exemple():
        assert appliquer_tva(100.0, 0.20) == 120.0
    ```
    A. PASSED ✅ B. FAILED C. ERROR D. Impossible à déterminer

!!! example "Niveau Analyser"
    *Un test passe toujours, même quand on introduit un bug dans le code testé. Quel est le problème le plus probable ?*

    A. Le test ne contient pas d'assertion significative ✅

    B. pytest est mal configuré

    C. Le bug est dans un autre fichier

    D. Le test est trop rapide

---

*Retour à l'[accueil](index.md).*
