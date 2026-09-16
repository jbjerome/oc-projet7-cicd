<p align="center">
   <img src="./front/src/favicon.png" width="192px" />
</p>

# MicroCRM (P7 - Développeur Full-Stack - Java et Angular - Mettez en œuvre l'intégration et le déploiement continu d'une application Full-Stack)

MicroCRM est une application de démonstration basique ayant pour être objectif de servir de socle pour le module "P7 - Développeur Full-Stack".

L'application MicroCRM est une implémentation simplifiée d'un ["CRM" (Customer Relationship Management)](https://fr.wikipedia.org/wiki/Gestion_de_la_relation_client). Les fonctionnalités sont limitées à la création, édition et la visualisations des individus liés à des organisations.

![Page d'accueil](./misc/screenshots/screenshot_1.png)
![Édition de la fiche d'un individu](./misc/screenshots/screenshot_2.png)

## Code source

### Organisation

Ce [monorepo](https://en.wikipedia.org/wiki/Monorepo) contient les 2 composantes du projet "MicroCRM":

- La partie serveur (ou "backend"), en Java SpringBoot 3;
- La partie cliente (ou "frontend"), en Angular 17.

### Démarrer avec les sources

#### Serveur

##### Dépendances

- [OpenJDK >= 17](https://openjdk.org/)

##### Procédure

1. Se positionner dans le répertoire `back` avec une invite de commande:

   ```shell
   cd back
   ```

2. Construire le JAR:

   ```shell
   # Sur Linux
   ./gradlew build

   # Sur Windows
   gradlew.bat build
   ```

3. Démarrer le service:

   ```shell
   java -jar build/libs/microcrm-0.0.1-SNAPSHOT.jar
   ```

Puis ouvrir l'URL http://localhost:8080 dans votre navigateur.

#### Client

##### Dépendances

- [NPM >= 10.2.4](https://www.npmjs.com/)

##### Procédure

1. Se positionner dans le répertoire `front` avec une invite de commande:

   ```shell
   cd front
   ```

2. (La première fois seulement) Installer les dépendances NodeJS:

   ```shell
   npm install
   ```

3. Démarrer le service de développement:

   ```shell
   npx @angular/cli serve
   ```

Puis ouvrir l'URL http://localhost:4200 dans votre navigateur.

### Exécution des tests

#### Client

**Dépendances**

- Google Chrome ou Chromium

Dans votre terminal:

```shell
cd front
CHROME_BIN=</path/to/google/chrome> npm test
```

#### Serveur

Dans votre terminal:

```shell
cd back
./gradlew test
```

### Docker

Chaque service a son propre `Dockerfile` (multi-stage, images Alpine minimales, utilisateur non-root, `HEALTHCHECK`). Un `docker-compose.yml` à la racine orchestre les deux.

#### Démarrer l'application complète

```shell
docker compose up --build
```

- Front (nginx) : http://localhost
- API back : http://localhost:8080/persons

Le front est configuré pour attendre que le back soit `healthy` avant de démarrer.

#### Construire une seule image

```shell
docker build -t microcrm-back:local ./back
docker build -t microcrm-front:local ./front
```

#### Images publiées

Sur chaque push `main`, les images sont publiées sur GHCR :

- `ghcr.io/jbjerome/microcrm-back:latest` (+ `sha-<commit>`)
- `ghcr.io/jbjerome/microcrm-front:latest` (+ `sha-<commit>`)

Sur chaque tag `vX.Y.Z`, les mêmes images sont taguées `vX.Y.Z`, `X.Y`, `X`.

## CI/CD

Trois workflows GitHub Actions dans `.github/workflows/` :

| Workflow | Déclencheur | Rôle |
|---|---|---|
| `ci.yml` | push, pull_request, cron lundi 06:00 UTC | Build back (Gradle + JaCoCo), build front (Karma + coverage + `ng build`), smoke test via `docker compose`, scan Trivy des images |
| `cd.yml` | push sur `main` | Build & push images sur GHCR (tags `latest` + `sha-<commit>`) |
| `release.yml` | push d'un tag `vX.Y.Z` | Build artefacts (JAR + `dist.zip`), push images tagées SemVer sur GHCR, création de la GitHub Release avec les artefacts attachés |

### Secrets attendus

Aucun secret n'est requis pour `ci.yml` dans sa version actuelle. `cd.yml` et `release.yml` utilisent le `GITHUB_TOKEN` fourni automatiquement par GitHub Actions (scope `packages:write` déjà autorisé dans `permissions:`).

L'intégration SonarQube Cloud (à activer ultérieurement) nécessitera un `SONAR_TOKEN` dans les secrets du repo.

### Monitoring (ELK)

Stack ELK autonome (non intégrée au CI/CD car trop lourde) pour visualiser les logs applicatifs du back.

```shell
# 1. Lancer la stack ELK (Elasticsearch, Logstash, Kibana)
docker compose -f docker-compose-elk.yml up -d

# 2. Lancer l'application connectée à ELK
docker compose -f docker-compose.yml -f docker-compose-monitoring.yml up --build -d
```

- Kibana : http://localhost:5601
- Elasticsearch : http://localhost:9200
- Index créé par Logstash : `microcrm-YYYY.MM.dd`

Dans Kibana, créer un **Data View** sur le pattern `microcrm-*` (champ time : `@timestamp`) pour explorer les logs et construire un dashboard.

Prévoir ~4 Go de RAM libres pour la stack.

### Versioning

Politique SemVer (`MAJOR.MINOR.PATCH`). Une release se déclenche par la création d'un tag :

```shell
git tag v0.1.0
git push origin v0.1.0
```
