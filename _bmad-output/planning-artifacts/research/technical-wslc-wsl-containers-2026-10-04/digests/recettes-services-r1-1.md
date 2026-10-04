# Digest — recettes de services (round 1, assistant 3)

Accès : 2026-10-04. Les pages Docker Hub donnent des dates relatives (« il y a N jours »), pas de dates absolues : la date de publication est « page en ligne au 2026-10-04 ». Les commandes sont celles des docs, avec `docker` remplacé par `wslc`. **Rien n'a été testé sous wslc.** Deux appels (mailpit.axllent.org et api.github.com) ont renvoyé 403 : Mailpit repose sur Docker Hub, le README GitHub et une recherche secondaire. WebFetch résume les pages via un petit modèle : vérifier les tags sur les pages Hub avant publication.

Format : constat | source | éditeur | confiance | classe

## 1. PostgreSQL
- Image `postgres`. Tags : `18.6` (= latest), `17.11`, `16.15`, `15.19`, `14.24`, `19beta4` (bêta) ; variantes Debian trixie/bookworm et Alpine. Recommandé : `postgres:18` ou `postgres:18.6`. | https://hub.docker.com/_/postgres | Docker Official Image | haute | image/version
- `POSTGRES_PASSWORD` obligatoire (ni vide ni absent). | idem | idem | haute | env
- Port 5432 : non cité dans l'extrait, valeur standard. Non vérifié ce run. | moyenne | port
- **Chemin de données modifié en 18+** : PGDATA = `/var/lib/postgresql/18/docker`, VOLUME déclaré = `/var/lib/postgresql`. Pour 17 et avant : monter `/var/lib/postgresql/data` ; monter seulement `/var/lib/postgresql` ne persiste pas. | idem | idem | haute | volume
- Pour 18 : `-v pgdata:/var/lib/postgresql`. Pour 17 ou avant : `-v pgdata:/var/lib/postgresql/data`. | dérivé | haute | volume
- Vérification (outils standards, non montrés sur la page) : `wslc exec pg psql -U postgres -c "select version();"` ou `pg_isready`. | basse | commande
- Commande proposée : `wslc run -d --name pg -e POSTGRES_PASSWORD=secret -p 5432:5432 -v pgdata:/var/lib/postgresql postgres:18`. | assemblée à partir de la page | moyenne | commande

## 2. Redis / Valkey
- Image toujours `redis`. Tags : `8.10.2` (= latest, Debian trixie et Alpine), `8.8.3`, `8.6.7`, `8.4.7`, `7.4.11`, `7.2.16`, `6.2.24`. Recommandé : `redis:8` ou `redis:8.10.2`. | https://hub.docker.com/_/redis | Docker Official Image | haute | image/version
- Licence : Redis 8.0+ triple licence RSALv2 / SSPLv1 / AGPLv3 ; versions <= 7.2.4 en BSD-3-Clause. | idem | idem | haute | autre
- Port 6379 (standard, confirmé par la page Valkey). | moyenne | port
- Volume `/data`. Persistance par snapshots : `redis-server --save 60 1 --loglevel warning`. Pour l'AOF, `redis-server --appendonly yes` (standard, non présent sur la page : non vérifié). | idem | haute (/data) ; moyenne (appendonly) | volume/commande
- Aucune variable obligatoire. Depuis 8.0.2 l'image corrige les permissions de /data (`SKIP_FIX_PERMS=1` pour l'éviter). | idem | haute | env
- Exemple de la page : `docker run -it --network some-network --rm redis redis-cli -h some-redis`. Test simple `wslc exec <nom> redis-cli ping` (attendu : PONG) : standard, non cité tel quel. | moyenne | commande
- Alternative Valkey : image `valkey/valkey`, tags `9.1.2` (= `9.1`, `9`, `latest`), variantes Alpine, 9.0, 8.1, 8.0, 7.2, 9.2.0-rc1. Port 6379, volume `/data`, client `valkey-cli`. La page ne dit pas explicitement « drop-in ». `https://hub.docker.com/_/valkey` renvoie 404 : pas d'image officielle Docker nommée `valkey` ; utiliser `valkey/valkey`. Mode protégé désactivé par défaut : mettre un mot de passe si le port est exposé. | https://hub.docker.com/r/valkey/valkey | projet Valkey | haute | image/version/port/volume

## 3. Apache Solr
- Image `solr`. Tags : `10.0.0` (= latest), `10.0.0-slim`, `9.10.1`, `9.10.1-slim`, `9.9.0`, `9.9.0-slim`. Recommandé : `solr:10.0.0` ou `solr:9.10.1`. | https://hub.docker.com/_/solr | Docker Official Image | haute | image/version
- Port 8983 ; interface http://localhost:8983/. | idem | haute | port
- Guide officiel (Solr 10.0) : `docker run -d -v "$PWD/solrdata:/var/solr" -p 8983:8983 --name my_solr solr solr-precreate gettingstarted` ; dans un conteneur en marche : `docker exec -it my_solr solr create -c gettingstarted`. | https://solr.apache.org/guide/solr/latest/deployment-guide/solr-in-docker.html | Apache Solr | haute | commande
- Volume `/var/solr` (données et logs). Vérification : ouvrir l'interface (Core Admin / Query `*:*`) ; `curl "http://localhost:8983/solr/gettingstarted/select?q=*:*"` standard, non cité. | moyenne | volume/commande
- Aucune variable obligatoire. Permissions de bind mount (uid 8983) non mentionnées : sous WSL un volume nommé est plus sûr. À tester. | basse | autre

## 4. Mailpit / MailHog
- Image `axllent/mailpit` (tag `latest`, mise à jour il y a ~1 jour, 15,3 Mo, multi-arch). Numéro de version non récupéré (releases GitHub et API en 403) : épingler après vérification sur https://github.com/axllent/mailpit/releases. | https://hub.docker.com/r/axllent/mailpit | projet Mailpit | haute (image) ; basse (version) | image/version
- Ports : 1025 (SMTP), 8025 (interface web http://localhost:8025). | Docker Hub et https://github.com/axllent/mailpit | idem | haute | port
- Commande Hub : `docker run -d --name=mailpit --restart unless-stopped -p 8025:8025 -p 1025:1025 axllent/mailpit`. | idem | haute | commande
- Volume `/data` recommandé ; `MP_DATABASE=/data/mailpit.db` pour la base SQLite (nom de variable issu du README et d'une recherche secondaire). | moyenne | volume/env
- Pas de variable obligatoire.
- Test d'envoi (README) : `echo "Test message" | mail -S smtp=smtp://localhost:1025 test@example.com` (client `mail` pas toujours installé). Alternative portable, non vue sur les pages : `curl --url smtp://localhost:1025 --mail-from a@x.test --mail-rcpt b@x.test -T mail.txt`. Vérification via l'interface ou `GET http://localhost:8025/api/v1/messages` (chemin de l'API issu de la connaissance de fond, non vérifié ce run). | moyenne/basse | commande
- MailHog : dernière release v1.0.0, 307 commits, 220 issues ouvertes, pas d'avis d'abandon explicite ni de date visible. Recherche secondaire : pas de release significative depuis ~2020, « effectivement non maintenu depuis 2022 », Mailpit cité comme successeur (mêmes ports 1025/8025). Pas de date primaire (api.github.com en 403). | résultats de recherche (chriswiegman.com, tessl.io, kinsta) | moyenne | autre

## 5. Extras
- **Adminer** : image `adminer`, tags `6.1.1`, `6`, `latest`, variantes `standalone` et `fastcgi`, v5 disponible. Port 8080 (standalone) ou 9000 (fastcgi). `ADMINER_DEFAULT_SERVER` optionnel. Exemple : `docker run -p 8080:8080 -e ADMINER_DEFAULT_SERVER=mysql adminer`. Dans Adminer, « Server » = nom du conteneur dans le même réseau. | https://hub.docker.com/_/adminer | Docker Official Image | haute
- **RabbitMQ** : `docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4-management`. Version la plus récente citée : 4.3.6. Ports 5672 (AMQP), 15672 (interface, guest/guest). Volume `/var/lib/rabbitmq`. `RABBITMQ_DEFAULT_USER` / `RABBITMQ_DEFAULT_PASS` : formulation ambiguë sur la dépréciation, à vérifier sur 4.x. La page Hub montrait encore un exemple `rabbitmq:3` périmé : utiliser `4-management`. L'utilisateur `guest` limité à localhost par défaut : connaissance de fond, non vérifié. | https://www.rabbitmq.com/docs/download et https://hub.docker.com/_/rabbitmq | Broadcom/RabbitMQ et Docker Official Image | haute (commande/ports) ; basse (variables) 
- **MongoDB** : image `mongo` (tags exacts non récupérés). Port 27017. `MONGO_INITDB_ROOT_USERNAME` et `MONGO_INITDB_ROOT_PASSWORD` optionnels (pas d'auth par défaut). Volume `/data/db`. La doc recommande des volumes nommés plutôt que des bind mounts sous Windows/macOS (fichiers mappés en mémoire) : éviter `/mnt/c`. Vérification : `exec <nom> mongosh --eval "db.runCommand({ping:1})"` (eval de l'assistant). | https://hub.docker.com/_/mongo | Docker Official Image | haute (env/port/volume)
- **MariaDB** : image `mariadb`, tags `latest` et `lts` (numéros non récupérés). Obligatoire : `MARIADB_ROOT_PASSWORD`, ou `MARIADB_ALLOW_EMPTY_ROOT_PASSWORD=1`, ou `MARIADB_RANDOM_ROOT_PASSWORD=1`. Port 3306. Volume `/var/lib/mysql`. Exemple : `docker run --detach --name mariadb-container -e MARIADB_ROOT_PASSWORD=secret -p 3306:3306 mariadb:latest`. Vérification : `exec ... mariadb -uroot -psecret -e "select 1"` (nom du client supposé, non vérifié). | https://hub.docker.com/_/mariadb | Docker Official Image | haute (env/port/volume)
- **MinIO : attention.** Sources secondaires (glukhov.org, dev.to, madewithlove, issue ragflow) : MinIO a cessé de publier ses images sur Docker Hub en oct. 2025 ; quay.io/minio/minio n'a qu'une release communautaire de sept. 2025 ; dépôt GitHub archivé en févr. 2026 (« no longer maintained ») ; console communautaire retirée en mai 2025. À ne pas utiliser en démo. Alternatives à vérifier (non vérifiées) : Garage, SeaweedFS, LocalStack. | https://glukhov.org/data-infrastructure/object-storage/minio-dead/ ; https://dev.to/rosgluk/minio-ce-in-2026-retired-upstream-source-only-and-what-to-use-1k02 | blogs indépendants | moyenne (page MinIO elle-même non consultée)
- Extras recommandés pour la démo : Adminer, RabbitMQ (`4-management`), MongoDB, MariaDB, Valkey (alternative à Redis). Nginx, Elasticsearch, OpenSearch, Meilisearch et MySQL n'ont **pas** été recherchés.

## Pistes à creuser
- Mailpit : version exacte et page des options (https://mailpit.axllent.org/docs/configuration/runtime-options/, https://github.com/axllent/mailpit/releases/latest) ; confirmer `POST /api/v1/send` et `/api/v1/messages`.
- Postgres 18 sous WSLC : tester `-v pgdata:/var/lib/postgresql` avec des volumes nommés ; un montage sur l'ancien `/var/lib/postgresql/data` en 18 donne un volume anonyme sans persistance.
- Solr 10 : permissions de bind mount (uid 8983) sous WSL ; choisir entre tag `10.0.0` et `9.10.1`.
- RabbitMQ : variables par défaut sur 4.x.
- Remplaçant de MinIO pour du S3 : Garage, SeaweedFS, LocalStack.

## Cherché et non trouvé
- Dates de publication absolues des pages Docker Hub.
- Version courante de Mailpit et contenu de sa doc officielle (403).
- Date primaire du dernier commit/release de MailHog.
- Listes de tags officielles pour mongo et mariadb ; port Postgres, exemples `pg_isready`/psql, `appendonly` et `redis-cli ping` verbatim, curl Solr : non montrés sur les pages (résumés possiblement incomplets).
- Image officielle Docker `valkey` (404).
- Nginx, Elasticsearch, OpenSearch, Meilisearch, MySQL : non recherchés (budget).
