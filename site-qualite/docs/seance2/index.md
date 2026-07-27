# Séance 2 — Conception de tests et couverture de code

!!! info "Objectifs de la séance"
    À l'issue de cette séance, vous serez capables de :

    - **Appliquer** les techniques boîte noire : partitions d'équivalence, valeurs limites, tables de décision
    - **Concevoir** des tests basés sur les transitions d'états
    - **Mesurer** la couverture de code (instructions et branches) avec coverage.py et JaCoCo
    - **Utiliser** les doublures de test (mocks, stubs, spies) avec `unittest.mock` et Mockito

---

## 1. Approches de test : boîte noire vs. boîte blanche

### Boîte noire (Black-box testing)

Le testeur ne connaît **pas** le code source. Il teste uniquement à partir des **spécifications** : entrées → sorties attendues.

Techniques principales :

- Partitions d'équivalence
- Valeurs limites
- Tables de décision
- Tests de transitions d'états

### Boîte blanche (White-box testing)

Le testeur connaît la **structure interne** du code. Il conçoit les tests pour couvrir des chemins spécifiques.

Techniques principales :

- Couverture d'instructions (C0)
- Couverture de branches (C1)
- Couverture de conditions (C2 / MC-DC)

!!! tip "Complémentarité"
    Les deux approches sont **complémentaires**, pas opposées. La boîte noire garantit que les exigences sont satisfaites ; la boîte blanche garantit que le code est entièrement exercé.

---

## 2. Partitions d'équivalence

### Principe

Diviser l'espace des entrées en **classes d'équivalence** : des groupes de valeurs qui déclenchent le même comportement. Tester une valeur par classe suffit.

### Méthode en 4 étapes (Myers, 1979[^1])

[^1]: Myers, G.J. (1979). *The Art of Software Testing*. Wiley. ISBN: 978-0471043287.

1. **Identifier** les paramètres d'entrée
2. **Déterminer** les classes valides et invalides pour chaque paramètre
3. **Choisir** un représentant par classe
4. **Combiner** pour former les cas de test

### Exemple : `valider_age(age: int) -> bool`

Spécification : accepte les âges dans `[13, 120]`.

| Classe | Plage | Validité | Représentant |
|--------|-------|----------|-------------|
| C1 | `age < 13` | Invalide | `age = 5` |
| C2 | `13 ≤ age ≤ 120` | Valide | `age = 25` |
| C3 | `age > 120` | Invalide | `age = 150` |

!!! warning "Piège fréquent"
    Ne pas oublier les cas spéciaux : valeurs négatives, zéro, très grands nombres. Ce sont souvent des sous-classes invalides à part entière.

---

## 3. Valeurs limites (Boundary Value Analysis)

### Principe

Les défauts se concentrent aux **frontières** des classes d'équivalence (Beizer, 1990[^2]). Pour chaque borne, tester :

[^2]: Beizer, B. (1990). *Software Testing Techniques*. 2nd ed. Van Nostrand Reinhold. ISBN: 978-0442206727.

- La borne elle-même
- Juste en dessous
- Juste au-dessus

### Application à `valider_age`

| Valeur | Position | Attendu |
|--------|----------|---------|
| 12 | Juste sous la borne min | `False` |
| **13** | **Borne minimale** | `True` |
| 14 | Juste au-dessus de la borne min | `True` |
| 119 | Juste sous la borne max | `True` |
| **120** | **Borne maximale** | `True` |
| 121 | Juste au-dessus de la borne max | `False` |

---

## 4. Tables de décision

### Principe

Lorsqu'un comportement dépend de **plusieurs conditions combinées**, une table de décision liste toutes les combinaisons pertinentes et le résultat attendu pour chacune.

### Exemple : remise en librairie

Règles métier :

- Étudiant **et** 13–25 ans → remise de **15 %**
- Étudiant **et** 26–30 ans → remise de **10 %**
- Client fidèle (non étudiant) **et** montant ≥ 50 € → remise de **5 %**
- Sinon → **0 %**

| # | Étudiant | Âge | Fidèle | Montant | Remise |
|---|----------|-----|--------|---------|--------|
| 1 | Oui | 20 | — | — | 15 % |
| 2 | Oui | 28 | — | — | 10 % |
| 3 | Oui | 35 | — | — | 0 % |
| 4 | Non | — | Oui | 60 € | 5 % |
| 5 | Non | — | Oui | 30 € | 0 % |
| 6 | Non | — | Non | 100 € | 0 % |

Chaque ligne de la table devient un cas de test.

---

## 5. Tests de transitions d'états

### Principe

Pour les systèmes dont le comportement dépend de l'**état courant**, on modélise une **machine à états** et on teste :

1. Chaque **transition valide** (l'action dans un état donné produit le bon état suivant)
2. Chaque **transition invalide** (l'action dans un mauvais état est rejetée)

### Machine à états du Panier

```
   ┌──────────────────────────────────────────┐
   │                                          │
   ▼        ajouter()         valider()       │  payer()
 VIDE ──────────────► ACTIF ──────────► VALIDÉ ──────────► PAYÉ
                       │  ▲
                       │  │  ajouter()
                       │  │  retirer()
                       ▼  │
                     (reste ACTIF)
                       │
               vider() │
                       ▼
                     VIDE
```

### Transitions à tester

| État initial | Action | État final | Remarque |
|-------------|--------|-----------|----------|
| VIDE | `ajouter()` | ACTIF | Transition valide |
| ACTIF | `ajouter()` | ACTIF | Ajout supplémentaire |
| ACTIF | `retirer()` (reste des articles) | ACTIF | Retrait partiel |
| ACTIF | `vider()` | VIDE | Panier vidé |
| ACTIF | `valider()` | VALIDÉ | Passage en validation |
| VALIDÉ | `payer()` | PAYÉ | Finalisation |
| VALIDÉ | `ajouter()` | — | **Rejeté** (`RuntimeError`) |
| PAYÉ | `ajouter()` | — | **Rejeté** (`RuntimeError`) |
| VIDE | `valider()` | — | **Rejeté** (panier vide) |

---

## 6. Couverture de code

### Qu'est-ce que la couverture ?

La couverture de code mesure **quel pourcentage du code source** est exécuté pendant les tests. C'est un indicateur quantitatif (boîte blanche) de l'exhaustivité des tests.

### Niveaux de couverture

| Niveau | Nom | Mesure | Symbole |
|--------|-----|--------|---------|
| C0 | **Couverture d'instructions** | % de lignes exécutées | Statement coverage |
| C1 | **Couverture de branches** | % de branches (`if`/`else`) empruntées | Branch coverage |
| C2 | **Couverture de conditions** | % de sous-conditions évaluées True/False | Condition / MC-DC |

### Exemple

```python
def categorie_age(age: int) -> str:
    if age < 13:           # Branche A
        return "enfant"
    elif age < 18:         # Branche B
        return "ado"
    else:                  # Branche C
        return "adulte"
```

Pour atteindre 100 % de couverture C1, il faut au minimum 3 tests : un pour chaque branche (ex : `age=10`, `age=15`, `age=25`).

### Couverture ≠ Qualité

!!! warning "La couverture est un indicateur, pas un objectif"
    - 100 % de couverture ne garantit **pas** l'absence de bugs
    - Un code couvert à 100 % peut avoir des assertions trop faibles
    - **Seuil recommandé** : ≥ 80–85 % en couverture de branches (C1)
    - La couverture montre le code **non testé**, c'est sa principale valeur

---

## 7. Outils de couverture

### Python : coverage.py

```bash
# Installer
pip install pytest-cov

# Lancer avec couverture
pytest --cov=src --cov-branch --cov-report=term-missing

# Rapport HTML
pytest --cov=src --cov-branch --cov-report=html

# Seuil minimal
pytest --cov=src --cov-branch --cov-fail-under=85
```

### Java : JaCoCo

Ajouter dans `pom.xml` :

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.12</version>
            <executions>
                <execution>
                    <goals><goal>prepare-agent</goal></goals>
                </execution>
                <execution>
                    <id>report</id>
                    <phase>test</phase>
                    <goals><goal>report</goal></goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

```bash
mvn test
# Rapport dans target/site/jacoco/index.html
```

---

## 8. Les doublures de test (Test Doubles)

### Pourquoi des doublures ?

En test unitaire, on isole le composant sous test. Si ce composant dépend d'un service externe (base de données, API, paiement), on le remplace par une **doublure** qui simule son comportement.

### Taxonomie de Meszaros (2007)[^3]

[^3]: Meszaros, G. (2007). *xUnit Test Patterns: Refactoring Test Code*. Addison-Wesley. ISBN: 978-0131495050.

| Type | Comportement | Usage |
|------|-------------|-------|
| **Dummy** | Objet passé mais jamais utilisé | Remplir un paramètre obligatoire |
| **Stub** | Retourne des valeurs prédéfinies | Simuler une réponse de service |
| **Fake** | Implémentation simplifiée mais fonctionnelle | Base de données en mémoire |
| **Mock** | Vérifie les interactions (appels, arguments) | Vérifier qu'un paiement a été appelé |
| **Spy** | Enregistre les appels reçus pour vérification a posteriori | Logger les appels |

### Python : `unittest.mock`

```python
from unittest.mock import Mock, patch

# Créer un mock
banque_mock = Mock()
banque_mock.debiter.return_value = True

# L'utiliser
service = ServicePaiement(banque=banque_mock)
resultat = service.payer(montant=42.0, carte="4111...")

# Vérifier l'interaction
banque_mock.debiter.assert_called_once_with(42.0, "4111...")
assert resultat is True
```

### `@patch` pour remplacer un module

```python
from unittest.mock import patch
from datetime import datetime

@patch("src.promotion.datetime")
def test_promotion_active_apres_18h(mock_datetime):
    mock_datetime.now.return_value = datetime(2026, 6, 28, 19, 30)
    assert est_promotion_active() is True
```

### Java : Mockito

```java
import static org.mockito.Mockito.*;

// Créer un mock
Banque banqueMock = mock(Banque.class);
when(banqueMock.debiter(42.0, "4111...")).thenReturn(true);

// L'utiliser
ServicePaiement service = new ServicePaiement(banqueMock);
boolean resultat = service.payer(42.0, "4111...");

// Vérifier
verify(banqueMock).debiter(42.0, "4111...");
assertTrue(resultat);
```

---

## Récapitulatif de la séance

| Concept | Points clés |
|---------|-------------|
| Boîte noire | Tester depuis les spécifications (partitions, limites, tables, états) |
| Boîte blanche | Tester la structure interne (couverture C0/C1/C2) |
| Partitions | Diviser les entrées en classes → 1 test par classe |
| Valeurs limites | Tester aux frontières (borne, borne−1, borne+1) |
| Tables de décision | Lister les combinaisons de conditions → résultats attendus |
| Transitions d'états | Tester les transitions valides et invalides d'une machine à états |
| Couverture | C0 (instructions), C1 (branches), C2 (conditions). Seuil : ≥ 85 % |
| Test doubles | Dummy, Stub, Fake, Mock, Spy (Meszaros, 2007) |
| `unittest.mock` | `Mock()`, `@patch`, `return_value`, `assert_called_once_with` |
| Mockito | `mock()`, `when().thenReturn()`, `verify()` |

---

*Passez aux travaux pratiques : [TP3 — Conception de tests pour le Panier](tp3-panier.md) puis [TP4 — Couverture de code](tp4-coverage.md).*
