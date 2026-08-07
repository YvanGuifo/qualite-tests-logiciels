# TP3 — Conception de tests pour le Panier

!!! info "Informations pratiques"
    **Durée** : 1h30 · **Langage** : Python 3.12+ · **Outils** : pytest 8.x
    
    **Objectifs** :

    - Appliquer les partitions d'équivalence et les valeurs limites
    - Tester les transitions d'une machine à états
    - Écrire une suite de tests complète pour la classe `Panier`
    - Utiliser `pytest.raises` pour les cas d'erreur

!!! danger "Comment utiliser les indices — À lire avant de commencer"
    Chaque exercice propose **2 niveaux d'aide imbriqués** :

    - **Indice 1 — Direction** : ouvrez-le après avoir cherché seul au moins 5 minutes.
    - **Indice 2 — Approche** : ouvrez-le si l'indice 1 ne suffit pas et après 5 minutes supplémentaires.

    **Les corrigés complets sont fournis dans un document séparé, distribué après le rendu.** Le TP noté (Séance 2) portera précisément sur des tests de Panier ; s'entraîner sans consulter de solution est la seule préparation efficace.

---

## Étape 0 — Code de départ

### 0.1 Enrichir le projet

Ajoutez le fichier `src/panier.py` à votre projet `librairie-en-ligne/` :

```
librairie-en-ligne/
├── src/
│   ├── __init__.py
│   ├── librairie.py      (du TP1)
│   └── panier.py         (nouveau)
└── tests/
    ├── __init__.py
    ├── test_librairie.py  (du TP1)
    └── test_panier.py     (nouveau)
```

### 0.2 Code — `src/panier.py`

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
        """Ajoute un livre au panier.

        Raises:
            RuntimeError: si le panier est validé ou payé.
            ValueError: si la quantité est <= 0.
        """
        if self.etat in (EtatPanier.VALIDE, EtatPanier.PAYE):
            raise RuntimeError("Panier non modifiable")
        if quantite <= 0:
            raise ValueError("Quantité doit être > 0")
        self.articles[livre.isbn] = self.articles.get(livre.isbn, 0) + quantite
        self.etat = EtatPanier.ACTIF

    def retirer(self, isbn: str, quantite: int = 1) -> None:
        """Retire un livre du panier.

        Raises:
            RuntimeError: si le panier est validé ou payé.
            KeyError: si l'ISBN n'est pas dans le panier.
            ValueError: si la quantité est <= 0 ou > quantité actuelle.
        """
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
        """Vide entièrement le panier."""
        if self.etat in (EtatPanier.VALIDE, EtatPanier.PAYE):
            raise RuntimeError("Panier non modifiable")
        self.articles.clear()
        self.etat = EtatPanier.VIDE

    def valider(self) -> None:
        """Valide le panier pour le paiement.

        Raises:
            RuntimeError: si le panier est vide ou déjà validé/payé.
        """
        if self.etat != EtatPanier.ACTIF:
            raise RuntimeError(
                f"Validation impossible depuis l'état {self.etat.value}"
            )
        self.etat = EtatPanier.VALIDE

    def payer(self) -> None:
        """Marque le panier comme payé.

        Raises:
            RuntimeError: si le panier n'est pas validé.
        """
        if self.etat != EtatPanier.VALIDE:
            raise RuntimeError(
                f"Paiement impossible depuis l'état {self.etat.value}"
            )
        self.etat = EtatPanier.PAYE

    def nombre_articles(self) -> int:
        """Retourne le nombre total d'articles dans le panier."""
        return sum(self.articles.values())
```

---

## Étape 1 — Tester `ajouter()` (cas nominaux)

Écrivez les tests suivants dans `tests/test_panier.py` :

### 1.1 Ajouter un livre à un panier vide

??? tip "Indice 1 — Direction"
    Deux assertions minimum : (1) l'état du panier après ajout, (2) le contenu du dictionnaire `articles`. Utilisez une fixture pour les livres réutilisés.

    ??? tip "Indice 2 — Approche"
        Structure du test : créez un `Panier()` (part de VIDE), appelez `ajouter(livre)`, puis assertez `panier.etat == EtatPanier.ACTIF` et `panier.articles[isbn] == 1`.

### 1.2 Ajouter plusieurs fois le même livre

??? tip "Indice 1 — Direction"
    Appelez `ajouter` deux fois avec le même livre mais des quantités différentes. La classe doit **cumuler**, pas écraser.

    ??? tip "Indice 2 — Approche"
        Après `ajouter(livre, 2)` puis `ajouter(livre, 3)`, la quantité dans `articles[isbn]` doit valoir 5.

### 1.3 Ajouter des livres différents

??? tip "Indice 1 — Direction"
    Deux ISBN distincts, donc deux entrées dans `articles`. Vérifiez à la fois le nombre d'entrées (`len`) et le total d'articles.

    ??? tip "Indice 2 — Approche"
        `nombre_articles()` retourne la somme des quantités, pas le nombre d'ISBN distincts.

---

## Étape 2 — Tester les cas d'erreur de `ajouter()`

### 2.1 Quantité invalide (≤ 0)

??? tip "Indice 1 — Direction"
    Deux cas à couvrir : quantité **exactement 0** (borne) et quantité **négative**. Le paramètre `match=` de `pytest.raises` permet de vérifier le message.

    ??? tip "Indice 2 — Approche"
        `with pytest.raises(ValueError, match="Quantité doit être > 0"):` puis appelez `ajouter` avec `quantite=0` à l'intérieur du bloc.

### 2.2 Ajout sur un panier validé ou payé

??? tip "Indice 1 — Direction"
    Il faut d'abord amener le panier dans l'état VALIDE (ou PAYE), puis tenter d'y ajouter un livre. Cela nécessite plusieurs appels de méthodes dans l'Arrange.

    ??? tip "Indice 2 — Approche"
        Pour l'état VALIDE : `ajouter(...)` → `valider()` → tenter `ajouter(...)`. Pour PAYE : ajouter le `payer()` en plus.

---

## Étape 3 — Tester `retirer()`

Écrivez des tests pour les scénarios suivants :

| # | Scénario | Résultat attendu |
|---|----------|------------------|
| 1 | Retirer 1 sur 3 | Quantité passe à 2, état reste ACTIF |
| 2 | Retirer tout un article | Article supprimé, panier reste ACTIF si d'autres articles |
| 3 | Retirer le dernier article | Panier passe à VIDE |
| 4 | ISBN absent | `KeyError` |
| 5 | Quantité > stock panier | `ValueError` |
| 6 | Retirer d'un panier validé | `RuntimeError` |

??? tip "Indice 1 — Direction"
    Chaque ligne du tableau = 1 test. Attention aux **effets de bord** : quand la quantité tombe à 0, l'ISBN doit disparaître du dict ; si `articles` devient vide, l'état repasse à VIDE.

    ??? tip "Indice 2 — Approche"
        Trois patterns d'assertion selon le cas :
        - Cas nominal : vérifier la quantité restante et l'état
        - Cas "disparition" : `assert isbn not in panier.articles`
        - Cas d'erreur : `with pytest.raises(...)` autour de l'appel

---

## Étape 4 — Tester les transitions d'états

C'est l'exercice le plus important de ce TP. Vous devez tester le parcours complet de la machine à états.

### 4.1 Parcours nominal complet

??? tip "Indice 1 — Direction"
    Un seul test qui suit tout le chemin VIDE → ACTIF → VALIDE → PAYE. Assertez l'état **après chaque transition**, pas seulement à la fin.

    ??? tip "Indice 2 — Approche"
        Enchaînez : `Panier()` → assert VIDE → `ajouter` → assert ACTIF → `valider` → assert VALIDE → `payer` → assert PAYE.

### 4.2 Transitions invalides

Testez toutes les transitions **interdites** :

| Depuis | Action | Exception attendue |
|--------|--------|--------------------|
| VIDE | `valider()` | `RuntimeError` |
| VIDE | `payer()` | `RuntimeError` |
| ACTIF | `payer()` | `RuntimeError` |
| VALIDÉ | `ajouter()` | `RuntimeError` |
| VALIDÉ | `retirer()` | `RuntimeError` |
| VALIDÉ | `valider()` | `RuntimeError` |
| PAYÉ | `ajouter()` | `RuntimeError` |
| PAYÉ | `valider()` | `RuntimeError` |
| PAYÉ | `payer()` | `RuntimeError` |

??? tip "Indice 1 — Direction"
    Pour chaque transition interdite, votre Arrange doit amener le panier dans l'état de départ. L'Act tente la transition interdite. L'Assert utilise `pytest.raises(RuntimeError)`.

    ??? tip "Indice 2 — Approche"
        Nommez chaque test explicitement : `test_<action>_panier_<etat>_leve_RuntimeError`. Le `match=` du `pytest.raises` documente le message attendu et rend le test plus strict.

---

## Étape 5 — Tester `vider()` et `nombre_articles()`

??? tip "Indice 1 — Direction"
    3 tests : `vider` fonctionne (retour à VIDE, tout est effacé), `vider` échoue si panier verrouillé, `nombre_articles` fait bien une somme.

    ??? tip "Indice 2 — Approche"
        Pour `vider`, testez 3 assertions à la fois (état, `nombre_articles()`, `len(articles)`). Pour `nombre_articles`, un panier avec 2 ISBN et 3+2 doit retourner 5.

---

## Vérification finale

```bash
pytest tests/test_panier.py -v
```

!!! success "Critères de réussite"
    - [ ] Au moins **20 tests** pour le Panier
    - [ ] Tous les états testés (VIDE, ACTIF, VALIDÉ, PAYÉ)
    - [ ] Toutes les transitions valides couvertes
    - [ ] Toutes les transitions invalides couvertes (9 cas `RuntimeError`)
    - [ ] Cas d'erreur : `ValueError`, `KeyError`, `RuntimeError`
    - [ ] Pattern AAA respecté dans chaque test
    - [ ] Fixtures utilisées pour éviter la duplication

---

*Continuez avec le [TP4 — Couverture de code](tp4-coverage.md).*
