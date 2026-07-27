# TP2 — Tests unitaires en Java avec JUnit 5

!!! info "Informations pratiques"
    **Durée** : 1h30 · **Langage** : Java 17+ · **Outils** : JUnit 5.10+, Maven
    
    **Objectifs** :

    - Structurer un projet Java avec JUnit 5 et Maven
    - Écrire des tests avec `@Test`, `@BeforeEach`, `@DisplayName`
    - Utiliser `@ParameterizedTest` avec `@CsvSource`
    - Vérifier les exceptions avec `assertThrows`

---

## Étape 0 — Mise en place du projet Maven

### 0.1 Structure du projet

```
librairie-java/
├── pom.xml
├── src/
│   ├── main/java/fr/efrei/librairie/
│   │   ├── Livre.java
│   │   └── LibraireService.java
│   └── test/java/fr/efrei/librairie/
│       └── LibraireServiceTest.java
```

### 0.2 Fichier `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>fr.efrei</groupId>
    <artifactId>librairie</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <junit.version>5.10.2</junit.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

### 0.3 Code de départ — `Livre.java`

```java
package fr.efrei.librairie;

public record Livre(String isbn, String titre, double prix, int stock) {
}
```

### 0.4 Code de départ — `LibraireService.java`

```java
package fr.efrei.librairie;

import java.util.Map;
import java.util.NoSuchElementException;

public class LibraireService {
    private final Map<String, Livre> catalogue;

    public LibraireService(Map<String, Livre> catalogue) {
        this.catalogue = catalogue;
    }

    public double getPrix(String isbn) {
        Livre livre = catalogue.get(isbn);
        if (livre == null) {
            throw new NoSuchElementException("ISBN inconnu : " + isbn);
        }
        return livre.prix();
    }

    public boolean estDisponible(String isbn, int quantite) {
        Livre livre = catalogue.get(isbn);
        return livre != null && livre.stock() >= quantite;
    }

    public boolean estDisponible(String isbn) {
        return estDisponible(isbn, 1);
    }

    public double calculerTotal(String isbn, int quantite) {
        Livre livre = catalogue.get(isbn);
        if (livre == null) {
            throw new NoSuchElementException("ISBN inconnu : " + isbn);
        }
        return livre.prix() * quantite;
    }
}
```

---

## Étape 1 — Premier test JUnit 5

### 1.1 Créer la classe de test

Créez `LibraireServiceTest.java` dans `src/test/java/fr/efrei/librairie/` :

```java
package fr.efrei.librairie;

import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

import java.util.HashMap;
import java.util.Map;

class LibraireServiceTest {

    private LibraireService service;

    @BeforeEach
    void setUp() {
        Map<String, Livre> catalogue = new HashMap<>();
        catalogue.put("978-2-07-036024-1",
            new Livre("978-2-07-036024-1", "Le Petit Prince", 7.50, 100));
        catalogue.put("978-2-07-040850-9",
            new Livre("978-2-07-040850-9", "L'Étranger", 6.90, 50));
        service = new LibraireService(catalogue);
    }
}
```

!!! note "`@BeforeEach`"
    La méthode `setUp()` est exécutée **avant chaque test**. Elle joue le rôle du **Arrange** partagé. Chaque test démarre avec un service fraîchement initialisé.

### 1.2 Ajouter un test pour `getPrix`

Ajoutez cette méthode dans la classe :

```java
@Test
@DisplayName("getPrix retourne le prix d'un livre existant")
void getPrix_livreExistant_retournePrix() {
    // Act
    double prix = service.getPrix("978-2-07-036024-1");

    // Assert
    assertEquals(7.50, prix, 0.001);
}
```

### 1.3 Lancer les tests

```bash
mvn test
```

Résultat attendu :

```
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
```

---

## Étape 2 — Tester les cas d'erreur

### 2.1 Exception pour ISBN inexistant

Écrivez un test qui vérifie que `getPrix` lève `NoSuchElementException` pour un ISBN absent :

??? tip "Indice"
    Utilisez `assertThrows(ExceptionClass.class, () -> { ... })`.

??? example "Solution"
    ```java
    @Test
    @DisplayName("getPrix lève NoSuchElementException pour ISBN inexistant")
    void getPrix_isbnInexistant_leveException() {
        assertThrows(NoSuchElementException.class,
            () -> service.getPrix("000-0-00-000000-0"));
    }
    ```

### 2.2 Vérifier le message d'exception

Vous pouvez aussi capturer l'exception pour vérifier son message :

```java
@Test
@DisplayName("getPrix inclut l'ISBN dans le message d'erreur")
void getPrix_isbnInexistant_messageContientIsbn() {
    NoSuchElementException exception = assertThrows(
        NoSuchElementException.class,
        () -> service.getPrix("000-0-00-000000-0")
    );
    assertTrue(exception.getMessage().contains("000-0-00-000000-0"));
}
```

---

## Étape 3 — Tester `calculerTotal`

Écrivez les tests suivants :

| # | Scénario | Attendu |
|---|----------|---------|
| 1 | 1 exemplaire à 7.50 € | 7.50 |
| 2 | 3 exemplaires à 7.50 € | 22.50 |
| 3 | ISBN inexistant | `NoSuchElementException` |

??? example "Solution"
    ```java
    @Test
    @DisplayName("calculerTotal — 1 exemplaire")
    void calculerTotal_unExemplaire_retournePrix() {
        double total = service.calculerTotal("978-2-07-036024-1", 1);
        assertEquals(7.50, total, 0.001);
    }

    @Test
    @DisplayName("calculerTotal — 3 exemplaires")
    void calculerTotal_troisExemplaires_retourneTriple() {
        double total = service.calculerTotal("978-2-07-036024-1", 3);
        assertEquals(22.50, total, 0.001);
    }

    @Test
    @DisplayName("calculerTotal — ISBN inexistant lève exception")
    void calculerTotal_isbnInexistant_leveException() {
        assertThrows(NoSuchElementException.class,
            () -> service.calculerTotal("ABSENT", 1));
    }
    ```

---

## Étape 4 — Tester `estDisponible`

Écrivez des tests pour couvrir les cas suivants :

| # | Stock | Quantité demandée | Attendu |
|---|-------|--------------------|---------|
| 1 | 100 | 1 | `true` |
| 2 | 50 | 50 | `true` (stock exact) |
| 3 | 50 | 51 | `false` (insuffisant) |
| 4 | ISBN absent | 1 | `false` |

??? example "Solution"
    ```java
    @Test
    @DisplayName("estDisponible — stock suffisant")
    void estDisponible_stockSuffisant_retourneTrue() {
        assertTrue(service.estDisponible("978-2-07-036024-1", 1));
    }

    @Test
    @DisplayName("estDisponible — stock exact")
    void estDisponible_stockExact_retourneTrue() {
        assertTrue(service.estDisponible("978-2-07-040850-9", 50));
    }

    @Test
    @DisplayName("estDisponible — stock insuffisant")
    void estDisponible_stockInsuffisant_retourneFalse() {
        assertFalse(service.estDisponible("978-2-07-040850-9", 51));
    }

    @Test
    @DisplayName("estDisponible — ISBN inexistant")
    void estDisponible_isbnInexistant_retourneFalse() {
        assertFalse(service.estDisponible("ISBN-ABSENT", 1));
    }
    ```

---

## Étape 5 — Tests paramétrés avec `@ParameterizedTest`

### 5.1 TVA paramétrée avec `@CsvSource`

Ajoutez cette méthode statique dans `LibraireService.java` :

```java
public static double appliquerTva(double prixHt, double taux) {
    return prixHt * (1 + taux);
}
```

Puis écrivez un test paramétré :

??? tip "Indice"
    ```java
    @ParameterizedTest
    @CsvSource({
        "100.0, 0.20, 120.0",
        // ... ajoutez d'autres cas
    })
    void appliquerTva_parametres(double prixHt, double taux, double attendu) {
        assertEquals(attendu, LibraireService.appliquerTva(prixHt, taux), 0.001);
    }
    ```

??? example "Solution"
    ```java
    import org.junit.jupiter.params.ParameterizedTest;
    import org.junit.jupiter.params.provider.CsvSource;

    @ParameterizedTest
    @CsvSource({
        "100.0, 0.20, 120.0",
        "100.0, 0.055, 105.5",
        "0.0, 0.20, 0.0",
        "50.0, 0.0, 50.0",
        "9.99, 0.20, 11.988"
    })
    @DisplayName("appliquerTva avec différents taux")
    void appliquerTva_parametres(double prixHt, double taux, double attendu) {
        assertEquals(attendu, LibraireService.appliquerTva(prixHt, taux), 0.001);
    }
    ```

### 5.2 Validation d'âge paramétrée

Ajoutez dans `LibraireService.java` :

```java
public static boolean validerAge(int age) {
    return age >= 13 && age <= 120;
}
```

Écrivez un test paramétré avec au moins 7 cas (bornes incluses, exclues, extrêmes) :

??? example "Solution"
    ```java
    @ParameterizedTest
    @CsvSource({
        "12, false",
        "13, true",
        "25, true",
        "120, true",
        "121, false",
        "0, false",
        "-1, false"
    })
    @DisplayName("validerAge — valeurs limites")
    void validerAge_parametres(int age, boolean attendu) {
        assertEquals(attendu, LibraireService.validerAge(age));
    }
    ```

---

## Étape 6 — Comparaison Python / Java

| Aspect | pytest (Python) | JUnit 5 (Java) |
|--------|----------------|-----------------|
| Annotation de test | Nom commence par `test_` | `@Test` |
| Setup | `@pytest.fixture` | `@BeforeEach` |
| Assertion | `assert expr` | `assertEquals(attendu, réel)` |
| Exception attendue | `pytest.raises(Error)` | `assertThrows(Error.class, ...)` |
| Paramétrisation | `@pytest.mark.parametrize` | `@ParameterizedTest` + `@CsvSource` |
| Flottants | `pytest.approx(val)` | `assertEquals(att, réel, delta)` |
| Exécution | `pytest -v` | `mvn test` |

---

## Vérification finale

```bash
mvn test
```

!!! success "Critères de réussite"
    - [ ] Tous les tests passent (`BUILD SUCCESS`)
    - [ ] Au moins 10 méthodes de test
    - [ ] `@BeforeEach` utilisé pour le setup partagé
    - [ ] Au moins un `assertThrows` pour les cas d'erreur
    - [ ] Au moins un `@ParameterizedTest` avec `@CsvSource`
    - [ ] `@DisplayName` sur chaque test pour la lisibilité
    - [ ] Noms de méthodes suivant la convention `methode_scenario_resultat`

---

*Rendez-vous en [Séance 2](../seance2/index.md) pour la conception de tests et la couverture de code.*
