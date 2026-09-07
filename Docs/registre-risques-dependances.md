# Registre des risques liés aux dépendances du front

## 1. Objet du document

Ce document décrit le traitement des vulnérabilités détectées dans les dépendances du projet **Ressources Relationnelles Front**.

Il distingue :

- les dépendances utilisées par l'application en production ;
- les dépendances utilisées uniquement pendant le développement ou dans la CI.

Dernière analyse : **3 septembre 2026**.

## 2. Situation constatée

La commande suivante contrôle uniquement les dépendances intégrées à l'application déployée :

```bash
npm audit --omit=dev
```

Résultat obtenu après la mise à jour de Next.js vers la version `16.3.4` :

```text
found 0 vulnerabilities
```

L'audit complet signale encore des vulnérabilités transitives dans les outils de développement :

```bash
npm audit
```

Résultat au 3 septembre 2026 :

```text
13 vulnerabilities (2 low, 4 moderate, 7 high)
```

Ces alertes proviennent principalement de dépendances transitives de `@lhci/cli@0.15.1`, notamment Lighthouse, Puppeteer, `extract-zip`, `tmp`, `uuid` et `qs`.

## 3. Analyse du risque résiduel

| Élément | Analyse |
|---|---|
| Composant concerné | Lighthouse CI (`@lhci/cli`) et ses dépendances transitives |
| Usage | Audit automatisé des performances, de l'accessibilité, des bonnes pratiques et du SEO |
| Présence en production | Non : dépendance de développement uniquement |
| Exposition | Exécution ponctuelle dans un environnement local ou un runner GitHub Actions éphémère |
| Impact potentiel | Atteinte au runner de CI ou blocage de l'audit si une entrée malveillante est traitée |
| Risque pour les utilisateurs | Faible, car les paquets concernés ne sont pas intégrés au livrable de production |
| Niveau retenu | Risque résiduel temporairement accepté |

## 4. Décision

La commande suivante ne doit pas être utilisée :

```bash
npm audit fix --force
```

Au moment de l'analyse, npm propose en effet de remplacer `@lhci/cli@0.15.1` par `@lhci/cli@0.1.0`. Cette opération constituerait une régression majeure susceptible de casser les audits Lighthouse.

La version `0.15.1` est la dernière version stable publiée de Lighthouse CI au moment de la vérification :

<https://github.com/GoogleChrome/lighthouse-ci/releases/tag/v0.15.1>

Le risque est donc accepté temporairement jusqu'à la publication d'une version corrective compatible.

## 5. Mesures de réduction du risque

Les mesures suivantes sont appliquées :

1. L'audit des dépendances de production est bloquant dans GitHub Actions :

   ```bash
   npm audit --omit=dev --audit-level=high
   ```

2. L'audit complet reste exécuté à titre informatif afin de conserver la visibilité sur les vulnérabilités des outils de développement.

3. `package-lock.json` est versionné afin de garantir des installations reproductibles avec `npm ci`.

4. Lighthouse CI est exécuté dans un runner GitHub Actions temporaire et isolé.

5. Les audits ciblent uniquement l'application du projet et ne doivent pas être lancés sur une URL non fiable.

6. Les mises à jour sont validées par les contrôles suivants avant intégration :

   ```bash
   npm run lint
   npm run type-check
   npm run test:ci
   npm run build
   npm audit --omit=dev
   ```

## 6. Suivi

| Action | Responsable | Échéance ou fréquence | Statut |
|---|---|---|---|
| Vérifier les nouvelles versions de `@lhci/cli` | Équipe projet | Une fois par mois | À suivre |
| Relancer l'audit complet après chaque mise à jour | Équipe projet | À chaque modification de dépendance | En continu |
| Mettre à jour LHCI lorsqu'une version corrective compatible est disponible | Équipe projet | Dès publication | En attente |
| Réévaluer le risque résiduel | Équipe projet | Prochaine revue : 3 octobre 2026 | Planifié |

## 7. Critères de clôture

Le risque pourra être clôturé lorsque :

- une version stable et compatible de Lighthouse CI corrigera les dépendances concernées ;
- `npm audit` ne signalera plus de vulnérabilité élevée liée à LHCI ;
- les audits Lighthouse, le lint, la vérification TypeScript, les tests et le build fonctionneront après la mise à jour.
