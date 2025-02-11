# Wiki NT2

## Informations générales

La base de données Notion rassemble les informations sur l'entièreté des sites développés par NT2 :

- [Dates de lancement](https://nt2.notion.site/08061a29cc044a55b02844ebcbb0b6d4?v=15afb8c2a55f44128adbcd21409ef52f)
- [Responsables](https://nt2.notion.site/08061a29cc044a55b02844ebcbb0b6d4?v=2ef0b8d5723d47c0adad9722b6bbb13a)
- [Statuts et activités](https://nt2.notion.site/08061a29cc044a55b02844ebcbb0b6d4?v=f5828038f3df47ec8eede347d64758bc)
- [Services](https://nt2.notion.site/08061a29cc044a55b02844ebcbb0b6d4?v=bb13b3d4e54e4b6d8c938eb79cf1918a)
- [Technologies](https://nt2.notion.site/08061a29cc044a55b02844ebcbb0b6d4?v=0c0f29c5e184418da7226da086e8e02a)
- [Hébergements](https://nt2.notion.site/08061a29cc044a55b02844ebcbb0b6d4?v=f84ffbb4b02447e7b8bb267cd16aee20)
- [Archives](https://nt2.notion.site/08061a29cc044a55b02844ebcbb0b6d4?v=ccf8a0bb2c73430fb263fbe398714294)

## Administration

### Gestionnaire de mots de passe

Toutes les informations de connexion sont stockés dans le compte 1Password de NT2.

### Comptes courriel

Les comptes courriels sont gérés par [Funio](https://funio.com/). Tous les services tiers utilisés par NT2 sont enregistrés sous l'adresse comptes@labo-nt2.org.

### Google Analytics

Le compte Google utilisé pour les intégrations Analytics est nt2.mediatique@gmail.com.

### Domaines web

NT2 utilise [Hover](https://www.hover.com/) pour la gestion de ses domaines.

### Contrats et ententes

Une archive des contrats et ententes signés par NT2 et ses partenaires se trouve dans le disque dur de sauvegarde de l'équipe médiatique.

## Système

### Architecture

Les applications sont hébergé à l'aide de deux machines virtuelles, production.nt2.uqam.ca et databases.nt2.uqam.ca.

La VM PRODUCTION sert au déploiement des conteneurs Docker et aux stockage des fichiers permanents, comme les images ou les sites statiques. Les conteneurs sont situés dans le dossier `var/www/nt2-runner` et les fichiers sont en bind mount dans `var/www/data`.

La VM DATABASES contient uniquement les bases de données MySQL, Postgres et MongoDB utilisés par les conteneurs de la VM PRODUCTION.

Un self-hosted runner de GitHub situé dans `var/www/nt2-runner` permet le déploiement des conteneurs via git.

Le routage des applications est administré par le conteneur Traefik.

### GitHub

L'entièreté du code de NT2 [est hébergé sur GitHub](https://github.com/nt2). Chaque répertoire contient une liste de secrets et de variables défini dans Settings -> Secrets and Variables -> Actions. Ces secrets et variables sont ensuites utilisés par le script de déploiement `.github/workflows/deploy.yml` de chaque application.

Pour déployer les extensions WordPress (submodules), il faut générer un token d'accès valide et l'enregistrer comme secret dans le répertoire. Ce token expire après 90 jours et est généré par un membre de l'organisation GitHub NT2.

### Docker

Chaque application est conteneurisé par Docker à l'aide du fichier `docker-compose.yml`. Il contient les configurations pour le nom de domaine, le certificat SSL et les redirections de Traefik.

Pour lister les conteneur actifs : `docker ps`

Pour visionner les logs d'un conteneur : `docker logs CONTAINER_ID`

Pour ouvrir un terminal à l'intérieur d'un conteneur : `docker exec -it CONTAINER_ID sh`

Pour supprimer les images, les conteneurs et les volumes inactifs : `docker system prune --volumes`

### Traefik

Le [conteneur traefik](https://github.com/nt2/traefik) sert de reverse-proxy à tous les autres conteneurs dans le network `web`. La configuration se trouve dans le fichier `traefik.yml`. Le fichier `docker-compose.yml` s'occupe de créer le volume letsencrypt `/var/www/data/traefik/letsencrypt:/letsencrypt` pour stocker en permanence les certificats SSL.

## Applications

### WordPress

Les applications WordPress de NT2 utilisent principalement l'extension [Advanced Custom Fields](https://www.advancedcustomfields.com/) pour créer le schéma de données et l'interface d'édition.

Le type d'architecture utilisé étant headless (CMS découplé), l'extension [WPGraphQL](https://www.wpgraphql.com/) est utilisé pour gérer l'api GraphQL entre WordPress et la portion front-end Next.js. WPGraphQL comprend aussi un IDE intégré pour créer et tester les requêtes GraphQL.

L'extension [WP Migrate](https://deliciousbrains.com/wp-migrate-db-pro/) est utilisé pour les migrations WordPress.

L'extension [WP All Import](https://www.wpallimport.com/) est utilisé pour l'importation et l'exportation des données en format CSV ou XLS.

### Next.js

Les applications récentes créés par NT2 utilisent le meta-framework Next.js (app router) pour la partie front-end du site.

En mode développement sur une machine locale, Next.js requiert d'avoir installé [node](https://formulae.brew.sh/formula/node), [pnpm](https://formulae.brew.sh/formula/pnpm) et bun (`brew install oven-sh/bun/bun`). Sur un ordinateur mac, ces outils peuvent être installés à l'aide de [homebrew](https://brew.sh/).

`graphql-request` et `graphql-codegen` sont utilisés pour créer et typer le client graphql. la commande `npm run dev` démarre l'application localement et compile les types des requêtes GraphQL à chaque sauvegarde.

`radix-ui` est utilisé pour créer des composantes intéractives et accessibles, comme les tabulations, les menus déroulants ou les fenêtres modales.

`tailwindcss` est utilisé en tant que framework css pour la mise en page du site.

`prettier` et `prettier-plugin-tailwindcss` sont utilisés pour formatter automatiquement le code à chaque sauvegarde.

### Sites statiques

Tous les sites statiques de NT2 sont hébergés dans le conteneur Nginx [static-sites](https://github.com/nt2/static-sites) dans la VM production.nt2.uqam.ca.

Pour activer un site fossile sans domaine web, déposer le fichier du projet dans le dossier `archive`. Le site sera ensuite disponible à l'adresse archive.nt2.uqam.ca/nom-du-dossier. 

Pour activer un site fossile avec un domaine uqam.ca, déposer le fichier du projet dans le dossier `custom-domains` et ajouter la règle suivante dans le fichier `http.conf` :

```
server {
    listen 80;
    server_name sous-domaine.uqam.ca;
    root /usr/share/nginx/html/custom-domains/nom-du-dossier;
    index index.html;
}
```
À noter que chaque nouveau domaine doit être ajouté au label Traefik dans le fichier `docker-compose.yml` :

```
# example.com
- "traefik.http.routers.example.rule=Host(`example.com`)"
- "traefik.http.routers.example.middlewares=example"
- "traefik.http.routers.example.entrypoints=websecure"
- "traefik.http.routers.example.tls.certresolver=myresolver"
- "traefik.http.middlewares.example.redirectregex.regex=^(http|https)://example.com/(.*)"
- "traefik.http.middlewares.example.redirectregex.replacement=https://archive.nt2.uqam.ca/example/"
- "traefik.http.middlewares.example.redirectregex.permanent=true"
```
