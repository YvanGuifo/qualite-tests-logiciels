# TP5 — Kata FizzBuzz en TDD

!!! info "Informations pratiques"
    **Durée** : 1h30 · **Langage** : Python 3.12+ · **Outils** : pytest
    
    **Objectifs** :

    - Pratiquer le cycle TDD strict (Red → Green → Refactor)
    - Écrire le code **minimum** pour faire passer chaque test
    - Refactorer en confiance grâce au filet de tests
    - Documenter chaque cycle TDD

---

## Règles du kata FizzBuzz

Écrire une fonction `fizzbuzz(n)` qui, pour un entier `n` donné :

- Retourne `"Fizz"` si `n` est divisible par **3**
- Retourne `"Buzz"` si `n` est divisible par **5**
- Retourne `"FizzBuzz"` si `n` est divisible par **3 et 5**
- Retourne la **représentation en chaîne** de `n` sinon

!!! warning "Règle TDD stricte"
    Vous devez suivre les 3 lois de Robert C. Martin :

    1. **Ne pas écrire de code de production** tant qu'un test ne l'exige pas
    2. **Ne pas écrire plus d'un test** que ce qui est nécessaire pour échouer
    3. **Ne pas écrire plus de code** que ce qui est nécessaire pour faire passer le test

---

## Mise en place

Créez deux fichiers :

```
fizzbuzz/
├── src/
│   └── fizzbuzz.py    (vide au départ !)
└── tests/
    └── test_fizzbuzz.py
```

---

## Cycle 1 — Le cas le plus simple : `fizzbuzz(1)`

### RED — Écrire le test

```python
from src.fizzbuzz import fizzbuzz

def test_fizzbuzz_1_retourne_1():
    assert fizzbuzz(1) == "1"
```

Lancez : `pytest tests/test_fizzbuzz.py -v`

```
E   ImportError: cannot import name 'fizzbuzz' from 'src.fizzbuzz'
```

Le test échoue (RED). C'est normal et attendu.

### GREEN — Code minimum

```python
# src/fizzbuzz.py
def fizzbuzz(n):
    return "1"
```

!!! warning "Oui, c'est volontaire"
    En TDD strict, on écrit le code **le plus simple possible** pour faire passer le test. `return "1"` est suffisant pour l'instant.

Lancez : `pytest -v` → **PASSED**

### REFACTOR

Rien à refactorer pour l'instant. Le code est trivial.

---

## Cycle 2 — Généraliser : `fizzbuzz(2)`

### RED

```python
def test_fizzbuzz_2_retourne_2():
    assert fizzbuzz(2) == "2"
```

Lancez → **FAILED** (le code retourne `"1"` pour tout)

### GREEN

```python
def fizzbuzz(n):
    return str(n)
```

Lancez → tous les tests passent.

### REFACTOR

Le code est déjà propre. Les deux tests passent.

---

## Cycle 3 — Premier Fizz : `fizzbuzz(3)`

### RED

```python
def test_fizzbuzz_3_retourne_fizz():
    assert fizzbuzz(3) == "Fizz"
```

Lancez → **FAILED** (retourne `"3"`)

### GREEN

```python
def fizzbuzz(n):
    if n == 3:
        return "Fizz"
    return str(n)
```

??? question "Pourquoi `n == 3` et pas `n % 3 == 0` ?"
    Parce que nous n'avons pas encore de test qui exige `n % 3 == 0` pour d'autres multiples de 3. On écrit le **minimum**. La généralisation viendra au prochain cycle.

---

## Cycle 4 — Généraliser Fizz : `fizzbuzz(6)`

### RED

```python
def test_fizzbuzz_6_retourne_fizz():
    assert fizzbuzz(6) == "Fizz"
```

Lancez → **FAILED** (retourne `"6"` car `n == 3` est faux)

### GREEN

```python
def fizzbuzz(n):
    if n % 3 == 0:
        return "Fizz"
    return str(n)
```

Maintenant on généralise avec le modulo. Tous les tests passent.

### REFACTOR

Le code est toujours simple et lisible. Rien à changer.

---

## Cycle 5 — Premier Buzz : `fizzbuzz(5)`

### RED

```python
def test_fizzbuzz_5_retourne_buzz():
    assert fizzbuzz(5) == "Buzz"
```

Lancez → **FAILED** (retourne `"5"`)

### GREEN

```python
def fizzbuzz(n):
    if n % 3 == 0:
        return "Fizz"
    if n % 5 == 0:
        return "Buzz"
    return str(n)
```

Tous les tests passent.

---

## Cycle 6 — FizzBuzz : `fizzbuzz(15)`

### RED

```python
def test_fizzbuzz_15_retourne_fizzbuzz():
    assert fizzbuzz(15) == "FizzBuzz"
```

Lancez → **FAILED** (retourne `"Fizz"` car `15 % 3 == 0` est testé en premier)

### GREEN

```python
def fizzbuzz(n):
    if n % 3 == 0 and n % 5 == 0:
        return "FizzBuzz"
    if n % 3 == 0:
        return "Fizz"
    if n % 5 == 0:
        return "Buzz"
    return str(n)
```

Tous les tests passent.

### REFACTOR — Éliminer la duplication

Le code fonctionne mais contient de la duplication (`n % 3 == 0` apparaît deux fois, `n % 5 == 0` aussi). Refactorons avec la technique de la concaténation :

```python
def fizzbuzz(n):
    sortie = ""
    if n % 3 == 0:
        sortie += "Fizz"
    if n % 5 == 0:
        sortie += "Buzz"
    return sortie or str(n)
```

!!! tip "Pourquoi cette version est meilleure"
    1. Chaque condition n'est vérifiée qu'**une seule fois**
    2. Le cas `FizzBuzz` est géré **naturellement** par concaténation
    3. L'ajout futur d'un `"Jazz"` pour les multiples de 7 ne nécessite qu'**une seule ligne**

Relancez tous les tests → **tous passent**. Le refactoring est validé.

---

## Cycle 7 — Consolider avec des cas supplémentaires

Ajoutez des tests pour renforcer la confiance :

```python
@pytest.mark.parametrize("n, attendu", [
    (1, "1"),
    (2, "2"),
    (3, "Fizz"),
    (4, "4"),
    (5, "Buzz"),
    (6, "Fizz"),
    (7, "7"),
    (9, "Fizz"),
    (10, "Buzz"),
    (15, "FizzBuzz"),
    (30, "FizzBuzz"),
    (45, "FizzBuzz"),
    (98, "98"),
    (99, "Fizz"),
    (100, "Buzz"),
])
def test_fizzbuzz_parametrise(n, attendu):
    assert fizzbuzz(n) == attendu
```

---

## Récapitulatif des cycles

| Cycle | Test ajouté | Code modifié | Technique TDD |
|-------|------------|-------------|---------------|
| 1 | `fizzbuzz(1) == "1"` | `return "1"` | Code en dur (fake it) |
| 2 | `fizzbuzz(2) == "2"` | `return str(n)` | Généralisation |
| 3 | `fizzbuzz(3) == "Fizz"` | `if n == 3` | Code en dur pour un cas |
| 4 | `fizzbuzz(6) == "Fizz"` | `if n % 3 == 0` | Généralisation |
| 5 | `fizzbuzz(5) == "Buzz"` | `if n % 5 == 0` | Nouveau cas |
| 6 | `fizzbuzz(15) == "FizzBuzz"` | Condition combinée | Gestion de la combinaison |
| 7 | 15 cas paramétrés | Concaténation | Refactoring + consolidation |

---

## Exercice bonus — Étendre FizzBuzz

!!! question "Défi"
    Ajoutez la règle suivante **en TDD strict** :

    - Si `n` est divisible par **7**, ajouter `"Jazz"` à la sortie
    - Exemples : `fizzbuzz(7) == "Jazz"`, `fizzbuzz(21) == "FizzJazz"`, `fizzbuzz(105) == "FizzBuzzJazz"`

    Suivez les cycles Red → Green → Refactor.

??? example "Solution"
    ```python
    # Test RED
    def test_fizzbuzz_7_retourne_jazz():
        assert fizzbuzz(7) == "Jazz"

    # GREEN — ajouter dans fizzbuzz :
    def fizzbuzz(n):
        sortie = ""
        if n % 3 == 0:
            sortie += "Fizz"
        if n % 5 == 0:
            sortie += "Buzz"
        if n % 7 == 0:
            sortie += "Jazz"
        return sortie or str(n)

    # Tests de consolidation
    def test_fizzbuzz_21_retourne_fizzjazz():
        assert fizzbuzz(21) == "FizzJazz"

    def test_fizzbuzz_35_retourne_buzzjazz():
        assert fizzbuzz(35) == "BuzzJazz"

    def test_fizzbuzz_105_retourne_fizzbuzzjazz():
        assert fizzbuzz(105) == "FizzBuzzJazz"
    ```

    Grâce au refactoring du cycle 6, l'ajout ne nécessite qu'**une seule ligne** de code de production.

---

## Vérification finale

```bash
pytest tests/test_fizzbuzz.py -v
```

!!! success "Critères de réussite"
    - [ ] Au moins 7 cycles TDD documentés (RED → GREEN → REFACTOR)
    - [ ] Tous les tests passent
    - [ ] Le code final utilise la version refactorisée (concaténation)
    - [ ] Test paramétré avec au moins 10 cas
    - [ ] Le code n'a **jamais** été écrit avant le test correspondant

---

*Continuez avec le [TP6 — CI avec GitHub Actions](tp6-ci-github-actions.md).*
