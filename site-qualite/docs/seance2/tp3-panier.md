# TP3 — Conception de tests pour le Panier

!!! info "Informations pratiques"
    **Durée** : 1h30 · **Langage** : Python 3.12+ · **Outils** : pytest 8.x
    
    **Objectifs** :

    - Appliquer les partitions d'équivalence et les valeurs limites
    - Tester les transitions d'une machine à états
    - Écrire une suite de tests complète pour la classe `Panier`
    - Utiliser `pytest.raises` pour les cas d'erreur

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

??? tip "Indice"
    Créez un `Panier()`, un `Livre`, appelez `ajouter()`, puis vérifiez que l'état passe à `ACTIF` et que le livre est bien dans `articles`.

??? example "Solution"
    ```python
    import pytest
    from src.librairie import Livre
    from src.panier import Panier, EtatPanier

    @pytest.fixture
    def livre_petit_prince():
        return Livre("978-2-07-036024-1", "Le Petit Prince", 7.50, 100)

    @pytest.fixture
    def livre_etranger():
        return Livre("978-2-07-040850-9", "L'Étranger", 6.90, 50)

    def test_ajouter_panier_vide_passe_a_actif(livre_petit_prince):
        panier = Panier()
        panier.ajouter(livre_petit_prince)

        assert panier.etat == EtatPanier.ACTIF
        assert panier.articles["978-2-07-036024-1"] == 1
    ```

### 1.2 Ajouter plusieurs fois le même livre

??? example "Solution"
    ```python
    def test_ajouter_meme_livre_cumule_quantites(livre_petit_prince):
        panier = Panier()
        panier.ajouter(livre_petit_prince, quantite=2)
        panier.ajouter(livre_petit_prince, quantite=3)

        assert panier.articles["978-2-07-036024-1"] == 5
    ```

### 1.3 Ajouter des livres différents

??? example "Solution"
    ```python
    def test_ajouter_livres_differents(livre_petit_prince, livre_etranger):
        panier = Panier()
        panier.ajouter(livre_petit_prince)
        panier.ajouter(livre_etranger, quantite=2)

        assert len(panier.articles) == 2
        assert panier.nombre_articles() == 3
    ```

---

## Étape 2 — Tester les cas d'erreur de `ajouter()`

### 2.1 Quantité invalide (≤ 0)

??? example "Solution"
    ```python
    def test_ajouter_quantite_zero_leve_ValueError(livre_petit_prince):
        panier = Panier()
        with pytest.raises(ValueError, match="Quantité doit être > 0"):
            panier.ajouter(livre_petit_prince, quantite=0)

    def test_ajouter_quantite_negative_leve_ValueError(livre_petit_prince):
        panier = Panier()
        with pytest.raises(ValueError):
            panier.ajouter(livre_petit_prince, quantite=-1)
    ```

### 2.2 Ajout sur un panier validé ou payé

??? example "Solution"
    ```python
    def test_ajouter_panier_valide_leve_RuntimeError(livre_petit_prince):
        panier = Panier()
        panier.ajouter(livre_petit_prince)
        panier.valider()

        with pytest.raises(RuntimeError, match="non modifiable"):
            panier.ajouter(livre_petit_prince)

    def test_ajouter_panier_paye_leve_RuntimeError(livre_petit_prince):
        panier = Panier()
        panier.ajouter(livre_petit_prince)
        panier.valider()
        panier.payer()

        with pytest.raises(RuntimeError):
            panier.ajouter(livre_petit_prince)
    ```

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

??? example "Solution complète"
    ```python
    def test_retirer_partiel(livre_petit_prince):
        panier = Panier()
        panier.ajouter(livre_petit_prince, quantite=3)
        panier.retirer("978-2-07-036024-1", quantite=1)

        assert panier.articles["978-2-07-036024-1"] == 2
        assert panier.etat == EtatPanier.ACTIF

    def test_retirer_totalite_article(livre_petit_prince, livre_etranger):
        panier = Panier()
        panier.ajouter(livre_petit_prince)
        panier.ajouter(livre_etranger)
        panier.retirer("978-2-07-036024-1")

        assert "978-2-07-036024-1" not in panier.articles
        assert panier.etat == EtatPanier.ACTIF  # reste L'Étranger

    def test_retirer_dernier_article_panier_vide(livre_petit_prince):
        panier = Panier()
        panier.ajouter(livre_petit_prince)
        panier.retirer("978-2-07-036024-1")

        assert panier.etat == EtatPanier.VIDE
        assert panier.nombre_articles() == 0

    def test_retirer_isbn_absent_leve_KeyError():
        panier = Panier()
        with pytest.raises(KeyError):
            panier.retirer("ISBN-INEXISTANT")

    def test_retirer_quantite_excessive_leve_ValueError(livre_petit_prince):
        panier = Panier()
        panier.ajouter(livre_petit_prince, quantite=2)
        with pytest.raises(ValueError, match="Quantité à retirer"):
            panier.retirer("978-2-07-036024-1", quantite=5)

    def test_retirer_panier_valide_leve_RuntimeError(livre_petit_prince):
        panier = Panier()
        panier.ajouter(livre_petit_prince)
        panier.valider()
        with pytest.raises(RuntimeError):
            panier.retirer("978-2-07-036024-1")
    ```

---

## Étape 4 — Tester les transitions d'états

C'est l'exercice le plus important de ce TP. Vous devez tester le parcours complet de la machine à états.

### 4.1 Parcours nominal complet

??? example "Solution"
    ```python
    def test_parcours_complet_vide_actif_valide_paye(livre_petit_prince):
        panier = Panier()
        assert panier.etat == EtatPanier.VIDE

        panier.ajouter(livre_petit_prince)
        assert panier.etat == EtatPanier.ACTIF

        panier.valider()
        assert panier.etat == EtatPanier.VALIDE

        panier.payer()
        assert panier.etat == EtatPanier.PAYE
    ```

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

??? example "Solution (extraits)"
    ```python
    def test_valider_panier_vide_leve_RuntimeError():
        panier = Panier()
        with pytest.raises(RuntimeError, match="Validation impossible"):
            panier.valider()

    def test_payer_panier_vide_leve_RuntimeError():
        panier = Panier()
        with pytest.raises(RuntimeError, match="Paiement impossible"):
            panier.payer()

    def test_payer_panier_actif_leve_RuntimeError(livre_petit_prince):
        panier = Panier()
        panier.ajouter(livre_petit_prince)
        with pytest.raises(RuntimeError):
            panier.payer()

    def test_valider_panier_deja_valide_leve_RuntimeError(livre_petit_prince):
        panier = Panier()
        panier.ajouter(livre_petit_prince)
        panier.valider()
        with pytest.raises(RuntimeError):
            panier.valider()

    def test_payer_panier_deja_paye_leve_RuntimeError(livre_petit_prince):
        panier = Panier()
        panier.ajouter(livre_petit_prince)
        panier.valider()
        panier.payer()
        with pytest.raises(RuntimeError):
            panier.payer()
    ```

---

## Étape 5 — Tester `vider()` et `nombre_articles()`

??? example "Solution"
    ```python
    def test_vider_panier_actif(livre_petit_prince, livre_etranger):
        panier = Panier()
        panier.ajouter(livre_petit_prince, quantite=3)
        panier.ajouter(livre_etranger)
        panier.vider()

        assert panier.etat == EtatPanier.VIDE
        assert panier.nombre_articles() == 0
        assert len(panier.articles) == 0

    def test_vider_panier_valide_leve_RuntimeError(livre_petit_prince):
        panier = Panier()
        panier.ajouter(livre_petit_prince)
        panier.valider()
        with pytest.raises(RuntimeError):
            panier.vider()

    def test_nombre_articles_cumule(livre_petit_prince, livre_etranger):
        panier = Panier()
        panier.ajouter(livre_petit_prince, quantite=3)
        panier.ajouter(livre_etranger, quantite=2)
        assert panier.nombre_articles() == 5
    ```

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
