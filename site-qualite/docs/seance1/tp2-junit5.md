# TP2 — Tests unitaires en Java avec JUnit 5

!!! info "Informations pratiques"
    **Durée** : 1h30 · **Langage** : Java 17+ · **Outils** : JUnit 5.10+, Maven
    
    **Objectifs** :

    - Structurer un projet Java avec JUnit 5 et Maven
    - Écrire des tests avec `@Test`, `@BeforeEach`, `@DisplayName`
    - Utiliser `@ParameterizedTest` avec `@CsvSource`
    - Vérifier les exceptions avec `assertThrows`

!!! danger "Comment utiliser les indices — À lire avant de commencer"
    Chaque exercice propose **2 niveaux d'aide imbriqués** :

    - **Indice 1 — Direction** : ouvrez-le après avoir cherché seul au moins 5 minutes.
    - **Indice 2 — Approche** : ouvrez-le si l'indice 1 ne suffit pas et après 5 minutes supplémentaires.

    **Les corrigés complets sont fournis dans un document séparé, distribué après le rendu du TP.** Cette organisation vous garantit un apprentissage authentique : impossible de "regarder la réponse" avant d'avoir vraiment cherché.

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

??? tip "Indice 1 — Direction"
    JUnit 5 fournit une assertion dédiée aux exceptions. Cherchez `assertThrows` dans la documentation `org.junit.jupiter.api.Assertions`.

    ??? tip "Indice 2 — Approche"
        Signature : `assertThrows(ClasseException.class, () -> methodQuiDoitEchouer())`. Le second argument est une lambda (`Executable`).

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

??? tip "Indice 1 — Direction"
    Trois tests, trois scénarios. Deux tests utilisent `assertEquals(attendu, réel, delta)` pour les flottants ; le troisième réutilise `assertThrows`.

    ??? tip "Indice 2 — Approche"
        Le `delta` (troisième argument de `assertEquals` pour un `double`) permet la tolérance flottante — `0.001` suffit ici. Réutilisez `service` déjà initialisé par `@BeforeEach`.

---

## Étape 4 — Tester `estDisponible`

Écrivez des tests pour couvrir les cas suivants :

| # | Stock | Quantité demandée | Attendu |
|---|-------|--------------------|---------|
| 1 | 100 | 1 | `true` |
| 2 | 50 | 50 | `true` (stock exact) |
| 3 | 50 | 51 | `false` (insuffisant) |
| 4 | ISBN absent | 1 | `false` |

??? tip "Indice 1 — Direction"
    Quatre tests indépendants. Pour un booléen, `assertTrue` / `assertFalse` sont plus lisibles que `assertEquals(true, ...)`.

    ??? tip "Indice 2 — Approche"
        Chaque test appelle `service.estDisponible(isbn, quantite)`. Attention au cas "stock exact" (stock == quantité) qui doit retourner `true`.

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

??? tip "Indice 1 — Direction"
    Remplacez `@Test` par `@ParameterizedTest` et ajoutez une source de données `@CsvSource`. La méthode reçoit alors des paramètres.

    ??? tip "Indice 2 — Approche"
        Chaque ligne de `@CsvSource` correspond à une ligne du tableau des cas. Les paramètres de la méthode reçoivent les colonnes dans l'ordre. Attention aux imports `org.junit.jupiter.params.ParameterizedTest` et `org.junit.jupiter.params.provider.CsvSource`.

### 5.2 Validation d'âge paramétrée

Ajoutez dans `LibraireService.java` :

```java
public static boolean validerAge(int age) {
    return age >= 13 && age <= 120;
}
```

Écrivez un test paramétré avec au moins 7 cas (bornes incluses, exclues, extrêmes) :

??? tip "Indice 1 — Direction"
    Même pattern que le test précédent, mais avec 2 paramètres (`int age`, `boolean attendu`). Pensez à tester les **bornes** (12/13, 120/121) et pas seulement l'intérieur.

    ??? tip "Indice 2 — Approche"
        Les 7 cas doivent couvrir : borne min − 1, borne min, milieu, borne max, borne max + 1, zéro, négatif.

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
