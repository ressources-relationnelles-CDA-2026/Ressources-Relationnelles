# Documentation Docker – Ressources Relationnelles

## Objectif

La mise en place de Docker permet de lancer facilement l’ensemble du projet avec un environnement identique pour tous les développeurs.

L’architecture contient :

* un conteneur pour le **front Next.js**
* un conteneur pour le **back Symfony**
* un conteneur pour la **base de données MySQL**
* un `docker-compose.yml` pour orchestrer l’ensemble

---

## Organisation du dépôt

```text
Ressources-Relationnelles/
├── Ressources-Relationnelles-Back/
│   ├── Dockerfile
│   └── .dockerignore
│
├── Ressources-Relationnelles-Front/
│   ├── Dockerfile
│   └── .dockerignore
│
├── docker-compose.yml
├── .env
└── .gitmodules
```

Le dépôt principal contient le front et le back sous forme de sous-modules Git.

---

## Front

Le Dockerfile du front utilise plusieurs étapes.

### Installation des dépendances

```dockerfile
FROM node:22-alpine AS dependances
```

Une image Node légère est utilisée pour installer les dépendances avec :

```bash
npm ci
```

### Build

Une deuxième étape exécute :

```bash
npm run build
```

afin de générer le build de production Next.js.

### Exécution

La dernière étape contient uniquement les fichiers nécessaires au fonctionnement de l’application.

L’application est exécutée avec un utilisateur non-root pour améliorer la sécurité du conteneur.

Le front est disponible sur le port :

```text
3000
```

---

## Correction des accès à localStorage

Lors du build Next.js, certaines pages provoquaient l’erreur :

```text
ReferenceError: localStorage is not defined
```

Next.js effectue une partie du rendu côté serveur pendant le build.

Or :

```text
localStorage
window
```

n’existent que dans le navigateur.

Les accès exécutés directement pendant le rendu ont donc été déplacés dans des :

```tsx
useEffect(() => {
  // accès localStorage
}, []);
```

Les accès présents uniquement dans des actions utilisateur comme un clic ou un formulaire ont été conservés.

---

## Back

Le Dockerfile du back utilise également plusieurs étapes.

### Dépendances

Composer installe les dépendances PHP nécessaires au projet Symfony.

### Image d’exécution

L’image finale utilise PHP CLI avec les extensions nécessaires :

```text
intl
pdo
pdo_mysql
zip
```

Symfony CLI est également installé afin de pouvoir lancer :

```bash
symfony server:start
```

Le conteneur utilise un utilisateur non-root.

Le back est disponible sur :

```text
8000
```

---

## Base de données

Docker Compose démarre une base :

```text
MySQL 8.4
```

La base utilisée par le projet est :

```text
projetcollab
```

Les données sont stockées dans un volume Docker afin qu’elles ne soient pas perdues lorsque le conteneur est recréé.

---

## Variables d’environnement

Les valeurs ne sont pas écrites directement dans le `docker-compose.yml`.

Le compose utilise par exemple :

```yaml
MYSQL_DATABASE: ${MYSQL_DATABASE}
DATABASE_URL: ${DATABASE_URL}
```

En local, les valeurs sont stockées dans :

```text
.env
```

Le fichier `.env` ne doit pas être envoyé sur GitHub.

Il doit être ajouté au :

```text
.gitignore
```

Pour GitHub Actions, les mêmes variables pourront être récupérées depuis les Secrets et Variables GitHub.

---

## Sous-modules Git

Le front et le back sont reliés au dépôt principal sous forme de sous-modules.

Après le clone du dépôt principal, les sous-modules peuvent être récupérés avec :

```bash
git submodule update --init --recursive
```

Pour récupérer la branche `dev` du front :

```bash
cd Ressources-Relationnelles-Front
git fetch
git switch dev
git pull origin dev
```

Pour le back :

```bash
cd ../Ressources-Relationnelles-Back
git fetch
git switch dev
git pull origin dev
```

Le dépôt principal enregistre ensuite le commit utilisé pour chaque sous-module.

---

## Lancer le projet

Depuis la racine du dépôt :

```bash
docker compose up --build
```

Cette commande :

1. construit l’image du front
2. construit l’image du back
3. démarre MySQL
4. démarre le back
5. démarre le front

---

## Vérifier les conteneurs

```bash
docker compose ps
```

ou :

```bash
docker ps
```

---

## Consulter les logs

Front :

```bash
docker compose logs front
```

Back :

```bash
docker compose logs back
```

Base de données :

```bash
docker compose logs db
```

---

## Arrêter le projet

Pour simplement arrêter les conteneurs :

```bash
docker compose stop
```

Pour arrêter et supprimer les conteneurs :

```bash
docker compose down
```

Les données MySQL restent conservées grâce au volume.

Pour supprimer également les données MySQL :

```bash
docker compose down -v
```

À utiliser uniquement lorsqu’on souhaite repartir avec une base vide.

---

## Reconstruction complète

Après une modification importante du front ou du back :

```bash
docker compose up --build
```

Pour reconstruire sans utiliser le cache Docker :

```bash
docker compose build --no-cache
docker compose up
```

---

## Résultat

La solution Docker permet désormais de lancer l’environnement complet du projet avec une seule commande :

```bash
docker compose up --build
```

Cela facilite l’installation du projet, limite les différences d’environnement entre les développeurs et prépare également le projet à une future utilisation dans une pipeline CI/CD.
