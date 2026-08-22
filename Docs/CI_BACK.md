# CI Back – Ressources Relationnelles

## 1. Objectif de la CI

La **CI (Continuous Integration / Intégration Continue)** du back a été mise en place afin d’automatiser les principaux contrôles du projet Symfony à chaque modification du code.

L’objectif est de :

- détecter rapidement les erreurs avant leur intégration sur les branches principales ;
- fiabiliser les Pull Requests ;
- exécuter les mêmes contrôles pour l’ensemble de l’équipe ;
- réduire les vérifications manuelles ;
- améliorer progressivement la qualité, la sécurité et la maintenabilité du back ;
- vérifier que l’API reste exécutable et que les tests continuent de fonctionner.

---

## 2. Déclenchement des workflows

Les workflows sont exécutés lors des pushs sur :

- `main`
- `dev`
- les branches respectant le format `**/**`

Ils sont également exécutés sur les Pull Requests vers :

- `main`
- `dev`

---

## 3. Workflows mis en place

### `ci-quality.yml` – Qualité PHP & Symfony

Ce workflow vérifie la qualité générale du projet back.

Il effectue notamment :

- la validation du fichier `composer.json` ;
- l’installation des dépendances Composer ;
- la vérification de la syntaxe PHP ;
- le lint du container Symfony ;
- le lint des fichiers YAML.

Des rapports sont générés et sauvegardés sous forme d’artefacts GitHub Actions afin de pouvoir consulter les résultats après l’exécution du pipeline.

Les rapports comprennent notamment :

- `composer-validate.txt`
- `lint-php.txt`
- `lint-container.txt`
- `lint-yaml.txt`

---

### `ci-tests.yml` – Tests PHPUnit

Le back possède déjà des tests unitaires et fonctionnels avec PHPUnit.

Le workflow met donc en place un environnement de test complet :

1. démarrage d’un service MySQL ;
2. installation de PHP et des extensions nécessaires ;
3. installation des dépendances Composer ;
4. création du schéma Doctrine pour la base de test ;
5. chargement des fixtures ;
6. exécution des tests PHPUnit ;
7. génération d’un rapport PHPUnit.

La base utilisée par la CI est temporaire et dédiée aux tests.

Un rapport PHPUnit est sauvegardé sous forme d’artefact afin de pouvoir consulter le détail des tests exécutés.

---

### `ci-security.yml` – Sécurité

Ce workflow regroupe plusieurs contrôles de sécurité.

#### Composer Audit

`composer audit` analyse les dépendances PHP afin de détecter les vulnérabilités connues.

Deux rapports sont générés :

- un rapport texte ;
- un rapport JSON.

Pour le moment, cette étape reste **non bloquante** afin de permettre la mise en place de la CI tout en conservant la visibilité sur les vulnérabilités existantes.

#### Gitleaks

Gitleaks analyse l’historique Git afin de détecter des informations sensibles qui auraient pu être commitées par erreur, notamment :

- tokens ;
- mots de passe ;
- clés API ;
- secrets.

La licence Gitleaks est stockée dans les GitHub Actions Secrets sous le nom :

```text
GITLEAKS_LICENSE
```

#### Semgrep

Semgrep réalise une analyse statique du code afin d’identifier certaines vulnérabilités et mauvaises pratiques de sécurité.

---

### `ci-sonar.yml` – SonarCloud

SonarCloud est utilisé pour analyser automatiquement le code source du back.

Il permet notamment de détecter :

- bugs ;
- vulnérabilités ;
- code smells ;
- problèmes de maintenabilité ;
- duplications de code.

Le projet est configuré via :

```text
sonar-project.properties
```

Le token SonarCloud utilisé par GitHub Actions est stocké dans les secrets du repository :

```text
SONAR_TOKEN
```

L’analyse automatique SonarCloud est désactivée afin d’éviter une double analyse avec GitHub Actions.

---

### `ci-docker.yml` – Build Docker

Ce workflow vérifie que l’image Docker du back peut toujours être construite correctement.

Il permet de :

- valider le `Dockerfile` ;
- détecter les erreurs liées à la construction de l’image ;
- vérifier qu’une modification du projet ne casse pas l’environnement Docker ;
- réutiliser le cache GitHub Actions afin d’accélérer les builds suivants.

L’image est uniquement construite pour validation et n’est pas publiée automatiquement sur un registre.

---

### `ci-api.yml` – Vérification de l’API

Ce workflow réalise un **smoke test** de l’application Symfony.

Il :

1. installe les dépendances ;
2. démarre temporairement l’application ;
3. attend que le serveur soit disponible ;
4. vérifie qu’une réponse HTTP est retournée.

Ce contrôle permet notamment de détecter :

- une application Symfony qui ne démarre plus ;
- une erreur fatale ;
- une mauvaise configuration ;
- un problème avec le point d’entrée de l’application.

---

### `ci-zap.yml` – OWASP ZAP

OWASP ZAP est utilisé pour effectuer un scan dynamique de sécurité de l’API.

Le scan cible :

```text
/api
```

afin d’analyser directement la partie exposée par API Platform.

Le scan est actuellement configuré comme **non bloquant**, afin de permettre une première phase d’analyse avant de rendre certaines alertes bloquantes dans la CI.

---

## 4. Rapports générés

La CI du back génère plusieurs artefacts téléchargeables depuis GitHub Actions :

### Qualité

```text
back-quality-reports
```

Contient notamment :

- validation Composer ;
- lint PHP ;
- lint Symfony ;
- lint YAML.

### Sécurité

```text
composer-security-report
```

Contient :

- `composer-audit.txt`
- `composer-audit.json`

### Tests

```text
phpunit-report
```

Contient notamment le rapport XML généré par PHPUnit.

---

## 5. Organisation des contrôles

Les contrôles ont volontairement été séparés dans plusieurs workflows afin de pouvoir identifier rapidement la source d’un problème.

La CI distingue notamment :

- qualité PHP / Symfony ;
- tests ;
- sécurité ;
- SonarCloud ;
- Docker ;
- disponibilité de l’API ;
- scan dynamique OWASP ZAP.

Cette organisation permet également de faire évoluer chaque contrôle indépendamment.

---

## 6. Résultats obtenus et tickets de suivi créés

La mise en place de la CI a permis de détecter plusieurs vulnérabilités présentes dans les dépendances du projet.

Afin de ne pas mélanger la mise en place de la CI avec les corrections de dépendances, trois tickets de suivi ont été créés :

| Ticket | Objet |
|---|---|
| `CCP-176` | Mise à jour de sécurité critique – Twig |
| `CCP-177` | Mise à jour de sécurité – Composants Symfony |
| `CCP-178` | Mise à jour de sécurité – API Platform |

Ces tickets ont été créés à partir des résultats de `composer audit`.

Ils permettent de traiter séparément :

- les vulnérabilités critiques et importantes présentes dans `twig/twig` ;
- les vulnérabilités présentes dans plusieurs composants Symfony ;
- les vulnérabilités détectées dans API Platform.

---

## 7. Points encore évolutifs

Certains contrôles restent volontairement évolutifs :

- `composer audit` est actuellement non bloquant tant que les vulnérabilités existantes ne sont pas corrigées ;
- OWASP ZAP est également non bloquant lors de cette première phase ;
- les seuils de qualité SonarCloud pourront être renforcés progressivement ;
- les rapports pourront être enrichis si de nouveaux outils de qualité ou de sécurité sont ajoutés ;
- les tests pourront être complétés avec davantage de tests fonctionnels et d’intégration.

---

## 8. Conclusion

La CI Back permet désormais de contrôler automatiquement une grande partie du projet Symfony à chaque évolution.

Elle apporte notamment :

- une validation automatique de la qualité du code ;
- une exécution réelle des tests PHPUnit avec MySQL et fixtures ;
- une analyse des dépendances ;
- une recherche de secrets ;
- une analyse statique de sécurité ;
- une analyse SonarCloud ;
- une validation du build Docker ;
- une vérification du démarrage de l’API ;
- un scan dynamique OWASP ZAP.

La mise en place de cette CI a également permis de transformer les vulnérabilités détectées automatiquement en tickets de suivi dédiés, afin de poursuivre l’amélioration du projet sans mélanger les corrections avec la mise en place du pipeline.
