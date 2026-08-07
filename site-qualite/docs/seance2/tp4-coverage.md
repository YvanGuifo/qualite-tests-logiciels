# TP4 — Couverture de code avec coverage.py et JaCoCo

!!! info "Informations pratiques"
    **Durée** : 1h · **Langages** : Python + Java · **Outils** : pytest-cov, coverage.py, JaCoCo
    
    **Objectifs** :

    - Mesurer la couverture d'instructions (C0) et de branches (C1)
    - Interpréter un rapport de couverture (terminal et HTML)
    - Identifier le code non couvert et écrire les tests manquants
    - Configurer un seuil minimal de couverture

!!! danger "Comment utiliser les indices — À lire avant de commencer"
    Ce TP est majoritairement procédural (commandes à exécuter). Les rares blocs contenant des indices imbriqués suivent la règle : **essayez d'abord seul**, ouvrez les indices dans l'ordre.

    **Les corrigés complets sont fournis dans un document séparé, distribué après le rendu du TP.**

---

## Partie A — Python : coverage.py

### Étape 1 — Installation

```bash
pip install pytest-cov
```

### Étape 2 — Première mesure

Lancez les tests du TP3 avec la mesure de couverture :

```bash
pytest tests/ --cov=src --cov-report=term-missing
```

Exemple de sortie :

```
Name              Stmts   Miss  Cover   Missing
------------------------------------------------
src/librairie.py     18      2    89%   28, 35
src/panier.py        42      5    88%   67-69, 82, 95
------------------------------------------------
TOTAL                60      7    88%
```

!!! note "Lecture du rapport"
    - **Stmts** : nombre total d'instructions
    - **Miss** : instructions non exécutées par les tests
    - **Cover** : pourcentage de couverture
    - **Missing** : numéros de lignes non couvertes

### Étape 3 — Couverture de branches

Ajoutez l'option `--cov-branch` pour mesurer la couverture de branches (C1), plus exigeante que la couverture d'instructions (C0) :

```bash
pytest tests/ --cov=src --cov-branch --cov-report=term-missing
```

La couverture de branches vérifie que chaque `if/else`, chaque boucle et chaque condition a été évaluée à `True` **et** à `False`.

??? question "Pourquoi la couverture de branches est-elle plus faible ?"
    Un `if` sans `else` compte comme deux branches : une pour la condition vraie, une pour la condition fausse (passage au-delà du `if`). Si vos tests ne déclenchent jamais la condition fausse, la branche est manquée.

### Étape 4 — Rapport HTML

Générez un rapport visuel :

```bash
pytest tests/ --cov=src --cov-branch --cov-report=html
```

Ouvrez `htmlcov/index.html` dans votre navigateur. Le rapport montre :

- Les lignes couvertes en **vert**
- Les lignes non couvertes en **rouge**
- Les branches partiellement couvertes en **jaune**

### Étape 5 — Atteindre 90 % de couverture

Identifiez les lignes non couvertes dans le rapport HTML et écrivez les tests manquants.

??? tip "Stratégie (méthode à essayer d'abord)"
    1. Consultez les lignes rouges et jaunes dans le rapport
    2. Pour chaque ligne non couverte, identifiez le scénario qui la déclencherait
    3. Écrivez un test pour ce scénario
    4. Relancez la mesure pour vérifier

    ??? tip "Indice — quel scénario cibler ?"
        Pour couvrir une ligne dans un `if X: raise Error`, il faut un test qui **déclenche X** — donc un test qui passe un paramètre invalide. Le nom du test doit refléter ce paramètre invalide (par exemple `test_<methode>_<parametre_invalide>_leve_<Exception>`).

### Étape 6 — Configurer un seuil minimal

Configurez pytest pour **échouer** si la couverture est inférieure à 85 % :

```bash
pytest tests/ --cov=src --cov-branch --cov-fail-under=85
```

Si la couverture est inférieure, pytest retourne un code d'erreur :

```
FAIL Required test coverage of 85% not reached. Total coverage: 82.50%
```

??? tip "Dans `pyproject.toml`"
    Pour rendre ce seuil permanent :

    ```toml
    [tool.pytest.ini_options]
    addopts = "--cov=src --cov-branch --cov-fail-under=85"
    ```

---

## Partie B — Java : JaCoCo

### Étape 1 — Configuration Maven

Si ce n'est pas déjà fait (voir cours), ajoutez le plugin JaCoCo dans `pom.xml` :

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

### Étape 2 — Générer le rapport

```bash
mvn clean test
```

Le rapport HTML est généré dans `target/site/jacoco/index.html`.

### Étape 3 — Interpréter le rapport JaCoCo

JaCoCo utilise un code couleur :

| Couleur | Signification |
|---------|---------------|
| Vert | Instruction/branche entièrement couverte |
| Jaune | Branche partiellement couverte (une seule direction testée) |
| Rouge | Instruction/branche non couverte |

Le rapport affiche les métriques par package, classe et méthode :

- **Instructions** (C0) : pourcentage d'instructions bytecode couvertes
- **Branches** (C1) : pourcentage de branches couvertes
- **Cxty** (Complexité cyclomatique) : nombre de chemins indépendants

### Étape 4 — Ajouter un seuil dans Maven

Pour faire échouer le build si la couverture est insuffisante :

```xml
<execution>
    <id>check</id>
    <goals><goal>check</goal></goals>
    <configuration>
        <rules>
            <rule>
                <element>BUNDLE</element>
                <limits>
                    <limit>
                        <counter>BRANCH</counter>
                        <value>COVEREDRATIO</value>
                        <minimum>0.85</minimum>
                    </limit>
                </limits>
            </rule>
        </rules>
    </configuration>
</execution>
```

---

## Partie C — Exercice de synthèse

### Défi : atteindre 95 % de couverture de branches

1. Lancez `pytest --cov=src --cov-branch --cov-report=html`
2. Ouvrez le rapport HTML
3. Pour chaque ligne/branche non couverte, écrivez le test manquant
4. Itérez jusqu'à atteindre 95 %

!!! warning "Attention aux faux positifs"
    Certaines lignes non couvertes sont **normales** (ex : `if __name__ == "__main__"`). Concentrez-vous sur le code métier.

---

## Vérification finale

```bash
pytest tests/ --cov=src --cov-branch --cov-fail-under=85 -v
```

!!! success "Critères de réussite"
    - [ ] Rapport terminal lisible avec `term-missing`
    - [ ] Rapport HTML généré et consultable
    - [ ] Couverture de branches ≥ 85 % sur `librairie.py`
    - [ ] Couverture de branches ≥ 85 % sur `panier.py`
    - [ ] Seuil `--cov-fail-under=85` configuré
    - [ ] Chaque test ajouté cible une branche spécifique identifiée dans le rapport

---

*Rendez-vous en [Séance 3](../seance3/index.md) pour le TDD, l'intégration continue et les pratiques avancées.*
