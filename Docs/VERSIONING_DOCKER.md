# Versioning des images Docker

## Objectif

Le projet utilise une convention de versioning simple pour identifier clairement les images Docker du Front et du Back.

Chaque image peut être associée à plusieurs tags afin de distinguer :

- la version fonctionnelle de l'application ;
- la dernière version disponible ;
- le commit Git précis ayant servi à générer l'image.

---

## Version sémantique

Les versions applicatives utilisent le format :

```text
vMAJOR.MINOR.PATCH
```

Exemple :

```text
v1.0.0
```

### MAJOR

Le numéro **MAJOR** correspond à une évolution importante pouvant introduire des changements incompatibles avec la version précédente.

Exemple :

```text
v1.0.0 → v2.0.0
```

### MINOR

Le numéro **MINOR** correspond à l'ajout d'une nouvelle fonctionnalité sans casser le fonctionnement existant.

Exemple :

```text
v1.0.0 → v1.1.0
```

### PATCH

Le numéro **PATCH** correspond principalement à une correction de bug ou à une petite modification qui n'ajoute pas de nouvelle fonctionnalité majeure.

Exemple :

```text
v1.1.0 → v1.1.1
```

---

## Tags Docker utilisés

Pour une même image, plusieurs tags peuvent être générés.

### Tag de version

Exemple :

```text
ressources-relationnelles-front:v1.0.0
ressources-relationnelles-back:v1.0.0
```

Ce tag correspond à une version précise de l'application.

Il permet de retrouver ou de redéployer une version connue.

### Tag `latest`

Exemple :

```text
ressources-relationnelles-front:latest
ressources-relationnelles-back:latest
```

Le tag `latest` représente la dernière image publiée considérée comme version courante.

Il est pratique pour récupérer rapidement la dernière version disponible, mais il ne remplace pas les tags de version précis.

### Tag SHA

Exemple :

```text
ressources-relationnelles-front:sha-a1b2c3d
ressources-relationnelles-back:sha-a1b2c3d
```

Le SHA correspond à l'identifiant du commit Git ayant servi à construire l'image.

Ce tag permet de faire le lien entre une image Docker et une version exacte du code source.

Il est particulièrement utile pour :

- retrouver le commit utilisé lors d'un déploiement ;
- faciliter le diagnostic en cas de problème ;
- revenir précisément à une image connue.

---

## Exemple complet

Une même image Front peut donc avoir les trois tags suivants :

```text
ressources-relationnelles-front:v1.0.0
ressources-relationnelles-front:latest
ressources-relationnelles-front:sha-a1b2c3d
```

Et de la même manière pour le Back :

```text
ressources-relationnelles-back:v1.0.0
ressources-relationnelles-back:latest
ressources-relationnelles-back:sha-a1b2c3d
```

Ces tags peuvent pointer vers la même image Docker.

---

## Génération dans la CI

Les tags sont générés automatiquement dans les workflows Docker grâce à :

```yaml
docker/metadata-action@v5
```

La stratégie actuelle génère :

```yaml
tags: |
  type=raw,value=latest
  type=raw,value=v1.0.0
  type=sha,prefix=sha-
```

Le SHA est récupéré automatiquement à partir du commit Git courant.

Pour le moment, les workflows utilisent :

```yaml
push: false
```

Les images sont donc construites et vérifiées par la CI, mais ne sont pas encore publiées dans un registre Docker.

La publication réelle des images sera réalisée dans la partie déploiement / registry.

---

## Déclenchement de la CI Docker

Afin d'éviter de lancer inutilement un build Docker sur chaque branche de travail, la CI Docker est limitée aux branches principales :

```text
dev
main
```

Cela permet de conserver la vérification des images sur les branches importantes tout en réduisant les exécutions inutiles.

---

## Règle de versioning retenue

En résumé :

```text
v1.0.0
│ │ │
│ │ └── PATCH : correction ou petite modification
│ └──── MINOR : nouvelle fonctionnalité compatible
└────── MAJOR : changement majeur ou incompatible
```

Exemples d'évolution :

```text
v1.0.0 → v1.0.1   Correction
v1.0.1 → v1.1.0   Nouvelle fonctionnalité
v1.1.0 → v2.0.0   Évolution majeure
```
