# CI Front – Ressources Relationnelles

## 1. Objectif de la CI

La **CI (Continuous Integration / Intégration Continue)** a été mise en place afin d’automatiser les principales vérifications du front à chaque modification du code.

L’objectif est de :

- détecter rapidement les erreurs avant leur intégration sur les branches principales ;
- fiabiliser les Pull Requests ;
- appliquer les mêmes contrôles à l’ensemble de l’équipe ;
- réduire les vérifications manuelles ;
- améliorer progressivement la qualité, la sécurité et la maintenabilité du projet.

La CI permet aujourd’hui de vérifier automatiquement la qualité du code, la compilation du projet, la sécurité des dépendances et du code, la construction de l’image Docker, le comportement minimal de l’application dans un navigateur ainsi que certains indicateurs de performance et d’accessibilité.

---

## 2. Pourquoi cette CI a été mise en place ?

La mise en place de la CI permet notamment de :

- **Détecter les erreurs plus tôt** : une erreur de lint, de typage, de build ou de configuration est repérée dès le push.
- **Sécuriser les Pull Requests** : les contrôles sont exécutés automatiquement avant l’intégration sur `dev` ou `main`.
- **Réduire les vérifications manuelles** : les mêmes commandes sont rejouées automatiquement dans GitHub Actions.
- **Améliorer la qualité du code** : ESLint, TypeScript et SonarCloud permettent d’identifier des problèmes de qualité, de maintenabilité et certaines vulnérabilités.
- **Renforcer la sécurité** : la CI analyse les dépendances, les secrets potentiellement exposés et certaines vulnérabilités du code.
- **Vérifier le comportement réel du front** : Playwright lance l’application dans un navigateur et vérifie qu’elle répond correctement.
- **Valider l’environnement Docker** : la CI s’assure que le `Dockerfile` reste fonctionnel.
- **Suivre les performances** : Lighthouse permet d’obtenir des indicateurs sur la performance, l’accessibilité, les bonnes pratiques et le SEO.

---

## 3. Déclenchement des workflows

Les workflows sont déclenchés sur :

- les branches principales :
  - `dev`
  - `main`
- les branches de travail :
  - `Chore/**`
  - `chore/**`
  - `feat/**`
  - `fix/**`
- les Pull Requests vers :
  - `dev`
  - `main`

---

## 4. Workflows mis en place

### `ci-quality.yml` – Qualité du code

Ce workflow vérifie la qualité générale du front.

Il exécute :

- **ESLint** pour contrôler les règles de qualité et de style du code ;
- **TypeScript** avec `tsc --noEmit` afin de détecter les erreurs de typage ;
- le **build Next.js** afin de vérifier que l’application peut être compilée en production.

---

### `ci-tests.yml` – Tests unitaires Jest

Jest a été intégré à la CI afin de préparer l’exécution automatique des tests unitaires.

Pour le moment, l’absence de tests est temporairement tolérée avec :

```bash
jest --ci --passWithNoTests
```

Cela permet de préparer le pipeline sans le bloquer tant qu’aucun test unitaire réel n’a encore été ajouté.

Les tests E2E Playwright sont explicitement exclus de Jest afin qu’ils ne soient pas exécutés par erreur comme des tests unitaires.

---

### `ci-security.yml` – Sécurité

Ce workflow regroupe plusieurs contrôles de sécurité.

#### npm audit

`npm audit` permet de rechercher les vulnérabilités connues dans les dépendances npm.

L’audit est actuellement utilisé de manière informative afin de ne pas bloquer immédiatement la CI sur des vulnérabilités déjà présentes dans le projet.

#### Gitleaks

Gitleaks analyse l’historique Git afin de rechercher des éléments sensibles qui auraient pu être commités par erreur, par exemple :

- tokens ;
- mots de passe ;
- clés API ;
- secrets.

#### Semgrep

Semgrep effectue une analyse statique du code afin de détecter certaines mauvaises pratiques et vulnérabilités potentielles.

---

### `ci-sonar.yml` – SonarCloud

SonarCloud a été intégré afin d’analyser automatiquement le code du front.

Il permet notamment de détecter :

- des bugs ;
- des vulnérabilités ;
- des code smells ;
- des problèmes de maintenabilité ;
- certaines duplications de code.

Le projet utilise un `SONAR_TOKEN` stocké dans les **GitHub Actions Secrets**.

La configuration du projet est définie dans :

```text
sonar-project.properties
```

---

### `ci-docker.yml` – Build Docker

Ce workflow vérifie que l’image Docker du front peut être construite correctement.

Il permet de :

- valider le `Dockerfile` ;
- détecter les erreurs de build Docker ;
- vérifier que les modifications du projet ne cassent pas l’image ;
- profiter du cache GitHub Actions pour accélérer les builds suivants.

L’image est uniquement construite pour validation et n’est pas publiée sur un registre.

---

### `ci-e2e.yml` – Tests E2E Playwright

Playwright a été ajouté afin de tester le comportement du front dans un vrai navigateur.

Le workflow :

1. installe les dépendances ;
2. installe Chromium ;
3. build et démarre l’application ;
4. exécute les tests E2E ;
5. sauvegarde le rapport Playwright en artefact.

Un premier test minimal vérifie que la page d’accueil se charge correctement.

Les tests E2E permettent de compléter Jest :

- **Jest** → tests unitaires ;
- **Playwright** → tests de parcours utilisateur dans un navigateur.

---

### `ci-lighthouse.yml` – Lighthouse

Lighthouse a été intégré afin d’obtenir automatiquement des indicateurs concernant :

- la performance ;
- l’accessibilité ;
- les bonnes pratiques ;
- le SEO.

Les seuils sont actuellement configurés en avertissement afin d’identifier les points à améliorer sans bloquer immédiatement la CI.

---

### `ci-zap.yml` – OWASP ZAP

OWASP ZAP a été ajouté afin d’effectuer un scan dynamique de sécurité du front.

Le workflow démarre l’application puis lance un **Baseline Scan**.

Le scan peut notamment identifier :

- des headers de sécurité absents ou incorrects ;
- des problèmes de configuration HTTP ;
- certaines faiblesses côté navigateur ;
- des informations potentiellement exposées.

Le scan est actuellement non bloquant afin de permettre une première phase d’observation et de correction.

---

## 5. Organisation des contrôles

Les contrôles ont volontairement été séparés dans plusieurs workflows.

Cette organisation permet de savoir rapidement quelle partie de la CI pose problème :

- qualité ;
- tests ;
- sécurité ;
- SonarCloud ;
- Docker ;
- E2E ;
- Lighthouse ;
- ZAP.

Cela facilite également la maintenance de la CI et permet de faire évoluer chaque contrôle indépendamment.

---

## 6. Résultats obtenus et tickets de suivi créés

La mise en place de la CI n’a pas uniquement permis de vérifier que le projet compile correctement.

Les différents outils ont également permis de faire apparaître plusieurs problèmes techniques et axes d’amélioration dans le front.

Afin de séparer la mise en place de la CI des corrections applicatives, plusieurs tickets de suivi ont été créés :

| Ticket | Objet |
|---|---|
| `CCP-168` | Corriger le Blocker SonarQube |
| `CCP-169` | Corriger le Bug SonarQube (Nullish operator) |
| `CCP-170` | Remplacer `setAttribute` par `dataset` (Code Smell) |
| `CCP-171` | Ajouter l’attribut `type` explicite aux boutons |
| `CCP-172` | Correction du nommage des composants |
| `CCP-173` | Nettoyage des variables et imports inutilisés |
| `CCP-174` | Optimisation des performances d’affichage |
| `CCP-175` | Mise à jour de la version Node.js dans la CI |

Ces tickets montrent l’intérêt concret de la CI : elle ne sert pas uniquement à vérifier qu’un projet compile, mais aussi à faire remonter des problèmes de qualité, de sécurité, de maintenabilité et de performance qui peuvent ensuite être traités progressivement par l’équipe.

---

## 7. Points encore évolutifs

Certains éléments ont volontairement été laissés évolutifs :

- les tests unitaires Jest sont intégrés, mais l’absence de tests est encore tolérée avec `--passWithNoTests` ;
- `npm audit` peut être rendu bloquant une fois les vulnérabilités existantes corrigées ;
- OWASP ZAP peut également devenir bloquant une fois les alertes initiales traitées ;
- la couverture Jest pourra être envoyée à SonarCloud lorsque de vrais tests unitaires seront ajoutés ;
- des notifications automatiques en cas d’échec de pipeline pourront être ajoutées ultérieurement.

---

## 8. Conclusion

La CI Front apporte désormais une base commune de contrôle à chaque évolution du projet.

Elle permet de :

- détecter plus rapidement les régressions ;
- améliorer la qualité des Pull Requests ;
- sécuriser davantage le code ;
- vérifier la compatibilité Docker ;
- tester le comportement du front ;
- suivre les performances ;
- rendre les problèmes visibles avant leur intégration sur les branches principales.

La mise en place de cette CI a également permis de transformer les anomalies détectées automatiquement en tickets de suivi dédiés, ce qui facilite la priorisation et l’amélioration continue du projet.
