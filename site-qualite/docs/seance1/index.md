# Séance 1 — Fondamentaux de la qualité et premiers tests

!!! info "Objectifs de la séance"
    À l'issue de cette séance, vous serez capables de :

    - **Définir** la qualité logicielle selon la norme ISO/IEC 25010
    - **Distinguer** erreur, défaut et panne (vocabulaire ISTQB)
    - **Expliquer** la pyramide des tests et les principes FIRST
    - **Écrire** des tests unitaires structurés avec pytest (Python) et JUnit 5 (Java)
    - **Utiliser** le pattern AAA (Arrange-Act-Assert) et la paramétrisation

---

## 1. Pourquoi tester ?

### Le coût des bugs en production

L'histoire du logiciel regorge de défaillances spectaculaires causées par des tests insuffisants :

| Incident | Année | Conséquence | Cause |
|----------|-------|-------------|-------|
| **Therac-25** | 1985–87 | 6 patients irradiés, 3 décès | Condition de course non testée |
| **Ariane 5 vol 501** | 1996 | Fusée détruite (370 M$) | Dépassement d'entier non géré |
| **Knight Capital** | 2012 | 440 M$ perdus en 45 min | Déploiement défectueux sans tests |
| **Heartbleed** | 2014 | Fuite de données massives | Absence de vérification de bornes dans OpenSSL |
| **Boeing 737 MAX** | 2018–19 | 346 décès | Défaut MCAS, tests insuffisants |

!!! warning "Règle du ×10"
    Le coût de correction d'un défaut est multiplié par **10** à chaque phase du cycle de vie. Un bug trouvé en production coûte 100 à 1 000 fois plus cher qu'un bug trouvé pendant le développement (Boehm & Basili, 2001[^1]).

[^1]: Boehm, B. & Basili, V. (2001). Software Defect Reduction Top 10 List. *IEEE Computer*, 34(1), 135–137. DOI: [10.1109/2.962984](https://doi.org/10.1109/2.962984)

---

## 2. La qualité logicielle — ISO/IEC 25010

La norme **ISO/IEC 25010:2011** définit un modèle de qualité structuré en **8 caractéristiques**, chacune divisée en sous-caractéristiques.

### Les 8 caractéristiques

| # | Caractéristique | Description | Exemple concret |
|---|----------------|-------------|-----------------|
| 1 | **Adéquation fonctionnelle** | Le logiciel fait-il ce qu'on lui demande ? | La calculatrice calcule correctement |
| 2 | **Performance** | Temps de réponse, débit, utilisation des ressources | Page chargée en < 2 secondes |
| 3 | **Compatibilité** | Coexistence et interopérabilité avec d'autres systèmes | Fonctionne sur Chrome, Firefox et Safari |
| 4 | **Convivialité** (Usability) | Facilité d'apprentissage et d'utilisation | Interface intuitive, accessible |
| 5 | **Fiabilité** | Fonctionnement correct dans le temps, tolérance aux pannes | Disponibilité 99,9 % |
| 6 | **Sécurité** | Confidentialité, intégrité, authenticité | Protection contre les injections SQL |
| 7 | **Maintenabilité** | Facilité de modification, modularité, testabilité | Code modulaire, bien documenté |
| 8 | **Portabilité** | Facilité d'adaptation à d'autres environnements | Fonctionne sur Linux, Windows, macOS |

!!! tip "À retenir"
    La qualité ne se réduit pas à « ça marche ». Un logiciel fonctionnel mais lent, vulnérable ou impossible à maintenir n'est **pas** un logiciel de qualité.

### Qualité interne vs. externe

La norme distingue deux perspectives :

- **Qualité interne** : propriétés du code source (lisibilité, modularité, couverture de tests). Visible par les développeurs.
- **Qualité externe** : comportement observé par l'utilisateur (performance, fiabilité, ergonomie).

La qualité interne *conditionne* la qualité externe : un code mal structuré finira par produire des bugs visibles.

---

## 3. Vocabulaire ISTQB : la chaîne erreur → défaut → panne

L'**ISTQB** (International Software Testing Qualifications Board) standardise le vocabulaire du test logiciel. La chaîne causale fondamentale est :

```
Erreur (humaine) → Défaut (dans le code) → Panne (à l'exécution)
```

### Définitions précises

| Terme | Synonymes | Définition | Exemple |
|-------|-----------|------------|---------|
| **Erreur** (*error*, *mistake*) | Faute humaine | Action humaine produisant un résultat incorrect | Le développeur écrit `<` au lieu de `<=` |
| **Défaut** (*defect*, *bug*, *fault*) | Bug | Imperfection dans le code résultant d'une erreur | `if age < 18` au lieu de `if age <= 18` |
| **Panne** (*failure*) | Défaillance | Comportement incorrect observable à l'exécution | Un mineur de 18 ans est accepté alors qu'il ne devrait pas |

!!! example "Illustration concrète"
    1. **Erreur** : le développeur confond « strictement inférieur » et « inférieur ou égal »
    2. **Défaut** : le code contient `if age < 18` au lieu de `if age <= 18`
    3. **Panne** : l'application accepte un utilisateur de 17 ans dans une section réservée aux adultes

    Un défaut n'entraîne pas toujours une panne. Si personne ne déclenche le chemin de code concerné, le bug reste dormant — mais il est toujours là.

### Pourquoi cette distinction est importante

Comprendre cette chaîne permet de cibler les actions correctives :

- **Réduire les erreurs** → revues de code, formation, pair programming
- **Détecter les défauts** → tests, analyse statique, revues
- **Prévenir les pannes** → tests automatisés, CI/CD, monitoring

---

## 4. Vérification vs. Validation (V&V)

Deux activités complémentaires, souvent confondues :

| | Vérification | Validation |
|--|-------------|------------|
| **Question** | « Construisons-nous le produit **correctement** ? » | « Construisons-nous le **bon** produit ? » |
| **Focus** | Conformité aux spécifications techniques | Satisfaction des besoins utilisateur |
| **Moyens** | Tests unitaires, revues de code, analyse statique | Tests d'acceptation, démos, feedback utilisateur |
| **Analogie** | Vérifier que le pont supporte 10 tonnes | Vérifier que le pont est au bon endroit |

!!! tip "Mnémonique de Boehm (1981)"
    - **Vérification** : le produit est-il *bien construit* ?
    - **Validation** : est-ce le *bon produit* ?

---

## 5. La pyramide des tests

La **pyramide des tests** (Mike Cohn, 2009[^2]) est un modèle d'architecture de la suite de tests qui recommande une répartition en trois niveaux :

[^2]: Cohn, M. (2009). *Succeeding with Agile: Software Development Using Scrum*. Addison-Wesley. ISBN: 978-0321579362.

```
         ╱  E2E  ╲          5–10 %
        ╱──────────╲
       ╱ Intégration ╲      15–20 %
      ╱────────────────╲
     ╱  Tests unitaires  ╲  70–80 %
    ╱──────────────────────╲
```

### Les trois niveaux

| Niveau | Portée | Vitesse | Coût | Exemple |
|--------|--------|---------|------|---------|
| **Unitaire** | Une fonction ou classe isolée | Très rapide (ms) | Faible | `test_calculer_total()` |
| **Intégration** | Interaction entre composants | Moyen (s) | Moyen | Service + base de données |
| **E2E (bout en bout)** | Parcours utilisateur complet | Lent (min) | Élevé | Scénario d'achat complet via navigateur |

### Anti-patterns

!!! danger "Le cornet de glace (Ice Cream Cone)"
    Pyramide inversée : beaucoup de tests E2E, peu de tests unitaires. Résultat : suite lente, fragile et coûteuse à maintenir.

!!! danger "Le sablier (Hourglass)"
    Beaucoup de tests unitaires et E2E, mais quasi aucun test d'intégration. Résultat : les composants fonctionnent isolément mais échouent ensemble.

---

## 6. Les principes FIRST

Les tests unitaires de qualité respectent les cinq principes **FIRST** :

| Lettre | Principe | Signification |
|--------|----------|---------------|
| **F** | *Fast* | Exécution rapide (millisecondes). Un test lent sera ignoré. |
| **I** | *Independent* | Chaque test est autonome, ne dépend pas de l'ordre d'exécution. |
| **R** | *Repeatable* | Même résultat à chaque exécution, quel que soit l'environnement. |
| **S** | *Self-validating* | Résultat binaire : PASS ou FAIL. Pas de vérification manuelle. |
| **T** | *Timely* | Écrit au bon moment — idéalement avant ou en même temps que le code. |

---

## 7. Le pattern AAA (Arrange-Act-Assert)

Tout test unitaire bien structuré suit le pattern **AAA** :

```python
def test_calculer_total_deux_exemplaires():
    # Arrange — préparer les données
    catalogue = {
        "978-2-07-036024-1": Livre(
            isbn="978-2-07-036024-1",
            titre="Le Petit Prince",
            prix=7.50,
            stock=100
        )
    }
    service = LibraireService(catalogue)

    # Act — exécuter l'action à tester
    total = service.calculer_total("978-2-07-036024-1", quantite=2)

    # Assert — vérifier le résultat
    assert total == 15.0
```

### Les trois phases en détail

| Phase | Rôle | Bonnes pratiques |
|-------|------|------------------|
| **Arrange** | Préparer le contexte : objets, données, mocks | Minimal : juste le nécessaire pour ce test |
| **Act** | Exécuter l'action ou la méthode testée | **Une seule** action par test |
| **Assert** | Vérifier que le résultat est conforme | Assertions claires, message explicite si échec |

!!! tip "Règle d'or"
    **Un test = un comportement.** Si vous avez besoin de plusieurs `assert` indépendants, c'est probablement plusieurs tests.

---

## 8. Nommer ses tests

Un bon nom de test documente le comportement attendu. Convention recommandée :

```
test_<methode>_<scenario>_<resultat_attendu>
```

Exemples :

- `test_calculer_total_quantite_positive_retourne_prix_fois_quantite`
- `test_est_disponible_stock_zero_retourne_false`
- `test_get_prix_isbn_inexistant_leve_KeyError`

---

## 9. Le code du fil rouge — Séance 1

### `librairie.py`

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
        """Calcule le prix total pour un nombre d'exemplaires donné.

        Raises:
            KeyError: si l'ISBN n'existe pas.
        """
        livre = self.catalogue[isbn]
        return livre.prix * quantite
```

### Fonctions utilitaires

```python
def valider_age(age: int) -> bool:
    """Valide qu'un âge est dans la plage acceptable [13, 120]."""
    return 13 <= age <= 120


def appliquer_tva(prix_ht: float, taux: float) -> float:
    """Applique un taux de TVA à un prix hors taxe.

    Args:
        prix_ht: prix hors taxe (>= 0)
        taux: taux de TVA (ex: 0.20 pour 20%)

    Returns:
        Le prix TTC.
    """
    return prix_ht * (1 + taux)
```

---

## 10. pytest — le framework de test Python

### Installation

```bash
pip install pytest
```

### Conventions

- Les fichiers de test commencent par `test_` (ex : `test_librairie.py`)
- Les fonctions de test commencent par `test_`
- Les assertions utilisent le mot-clé `assert` natif de Python

### Exécution

```bash
# Lancer tous les tests
pytest

# Mode verbeux
pytest -v

# Un fichier spécifique
pytest tests/test_librairie.py

# Un test spécifique
pytest tests/test_librairie.py::test_calculer_total_deux_exemplaires
```

### Paramétrisation avec `@pytest.mark.parametrize`

La paramétrisation permet de tester plusieurs cas avec le même code de test :

```python
import pytest
from librairie import appliquer_tva

@pytest.mark.parametrize("prix_ht, taux, attendu", [
    (100.0, 0.20, 120.0),   # TVA standard 20%
    (100.0, 0.055, 105.5),   # TVA réduite 5,5%
    (0.0, 0.20, 0.0),        # Prix nul
    (50.0, 0.0, 50.0),       # Taux nul
])
def test_appliquer_tva(prix_ht, taux, attendu):
    assert appliquer_tva(prix_ht, taux) == pytest.approx(attendu)
```

### `pytest.approx` pour les flottants

Les nombres à virgule flottante ne se comparent pas avec `==` à cause des erreurs d'arrondi :

```python
>>> 0.1 + 0.2 == 0.3
False  # Surprise !

>>> 0.1 + 0.2
0.30000000000000004
```

La solution : `pytest.approx` qui tolère une petite marge d'erreur :

```python
assert appliquer_tva(100.0, 0.055) == pytest.approx(105.5)
assert appliquer_tva(100.0, 0.055) == pytest.approx(105.5, rel=1e-9)
```

---

## 11. JUnit 5 — le framework de test Java

### Structure d'un test JUnit 5

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class LibraireServiceTest {

    private LibraireService service;

    @BeforeEach
    void setUp() {
        Map<String, Livre> catalogue = new HashMap<>();
        catalogue.put("978-2-07-036024-1",
            new Livre("978-2-07-036024-1", "Le Petit Prince", 7.50, 100));
        service = new LibraireService(catalogue);
    }

    @Test
    @DisplayName("calculerTotal retourne prix × quantité")
    void calculerTotal_quantitePositive_retourneProduit() {
        // Arrange (fait dans setUp)
        // Act
        double total = service.calculerTotal("978-2-07-036024-1", 2);
        // Assert
        assertEquals(15.0, total, 0.001);
    }

    @Test
    @DisplayName("getPrix lève une exception pour un ISBN inexistant")
    void getPrix_isbnInexistant_leveException() {
        assertThrows(NoSuchElementException.class,
            () -> service.getPrix("000-0-00-000000-0"));
    }
}
```

### Annotations essentielles

| Annotation | Rôle |
|------------|------|
| `@Test` | Marque une méthode comme test |
| `@BeforeEach` | Exécuté avant chaque test (setup) |
| `@AfterEach` | Exécuté après chaque test (teardown) |
| `@DisplayName("...")` | Nom lisible pour les rapports |
| `@ParameterizedTest` | Test paramétré (plusieurs jeux de données) |
| `@CsvSource({...})` | Source de données CSV pour `@ParameterizedTest` |
| `@Disabled("raison")` | Désactive temporairement un test |

### Tests paramétrés en Java

```java
@ParameterizedTest
@CsvSource({
    "100.0, 0.20, 120.0",
    "100.0, 0.055, 105.5",
    "0.0, 0.20, 0.0",
    "50.0, 0.0, 50.0"
})
@DisplayName("appliquerTva avec différents taux")
void appliquerTva_parametres(double prixHt, double taux, double attendu) {
    assertEquals(attendu, appliquerTva(prixHt, taux), 0.001);
}
```

### Assertions principales

| Assertion | Usage |
|-----------|-------|
| `assertEquals(attendu, réel)` | Vérifie l'égalité |
| `assertTrue(condition)` | Vérifie que la condition est vraie |
| `assertFalse(condition)` | Vérifie que la condition est fausse |
| `assertThrows(Exception.class, () -> ...)` | Vérifie qu'une exception est levée |
| `assertNull(objet)` | Vérifie que l'objet est null |
| `assertNotNull(objet)` | Vérifie que l'objet n'est pas null |

---

## Récapitulatif de la séance

| Concept | Points clés |
|---------|-------------|
| ISO 25010 | 8 caractéristiques de qualité (fonctionnalité → portabilité) |
| ISTQB | Erreur → Défaut → Panne |
| V&V | Vérification (bien construit) vs. Validation (bon produit) |
| Pyramide | 70-80 % unitaires, 15-20 % intégration, 5-10 % E2E |
| FIRST | Fast, Independent, Repeatable, Self-validating, Timely |
| AAA | Arrange → Act → Assert |
| pytest | `assert`, `@pytest.mark.parametrize`, `pytest.approx` |
| JUnit 5 | `@Test`, `@BeforeEach`, `@ParameterizedTest`, `@CsvSource` |

---

*Passez aux travaux pratiques : [TP1 — Premiers tests avec pytest](tp1-pytest-aaa.md) puis [TP2 — JUnit 5](tp2-junit5.md).*
