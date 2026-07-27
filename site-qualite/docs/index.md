# Qualité et Tests Logiciels

**Cours L2 — EFREI Paris — 2025–2026**

Bienvenue sur le site du cours *Qualité et Tests Logiciels*. Ce cours de 15 heures (3 séances de 5h) vous apprendra à écrire du logiciel fiable en maîtrisant les techniques de test, de la théorie aux outils professionnels.

---

## Fil rouge : la librairie en ligne

Tout au long du cours, vous construirez et testerez une **application de librairie en ligne** composée de deux modules :

- **`librairie.py`** — gestion du catalogue (prix, disponibilité, calcul de total)
- **`panier.py`** — panier d'achat avec machine à états (Vide → Actif → Validé → Payé)

Chaque TP enrichit cette application avec de nouvelles techniques de test.

---

## Organisation du cours

| Séance | Thème | Durée | TPs |
|--------|-------|-------|-----|
| **Séance 1** | Fondamentaux de la qualité et premiers tests | 5h | TP1 (pytest + AAA) · TP2 (JUnit 5) |
| **Séance 2** | Conception de tests et couverture de code | 5h | TP3 (Panier) · TP4 (Coverage) |
| **Séance 3** | TDD, CI et pratiques avancées | 5h | TP5 (FizzBuzz TDD) · TP6 (GitHub Actions) |

---

## Objectifs d'apprentissage

À l'issue de ce cours, vous serez capables de :

1. **Expliquer** les 8 caractéristiques qualité de la norme ISO/IEC 25010
2. **Écrire** des tests unitaires structurés (pattern AAA) avec pytest et JUnit 5
3. **Concevoir** des cas de test à partir de partitions d'équivalence, valeurs limites et tables de décision
4. **Mesurer** la couverture de code (instructions, branches) et interpréter les résultats
5. **Appliquer** le cycle TDD (Red-Green-Refactor) sur un kata complet
6. **Configurer** un pipeline d'intégration continue avec GitHub Actions

---

## Évaluation

| Composante | Poids | Séance |
|------------|-------|--------|
| TP noté (test du Panier) | 40 % | Séance 2 |
| QCM final | 60 % | Séance 3 |

Détails dans la section [Évaluation](evaluation.md).

---

## Prérequis

- Programmation Python (variables, fonctions, classes, modules)
- Notions de base en Java (classes, méthodes, imports)
- Utilisation du terminal (ligne de commande)
- Git (clone, commit, push) — utile pour le TP6

---

## Environnement technique

| Outil | Version | Usage |
|-------|---------|-------|
| Python | 3.12+ | Langage principal |
| pytest | 8.x | Framework de test Python |
| coverage.py | 7.x | Mesure de couverture Python |
| Java | 17+ | Langage secondaire |
| JUnit 5 | 5.10+ | Framework de test Java |
| JaCoCo | 0.8+ | Mesure de couverture Java |
| Git | 2.x | Gestion de versions |
| GitHub Actions | — | Intégration continue |

---

*Dr. Yvan GUIFO FODJO — EFREI Paris — yvan.guifo-fodjo@efrei.fr*
