# Catalogue des services de démo

Commandes assemblées à partir des documentations officielles, en syntaxe PowerShell avec volumes nommés. **Statut de toutes les commandes : à tester sous WSLC.** Versions relevées le 2026-10-04 ; revérifier les tags la veille de l'exposé. Détails et sources : les deux rapports de recherche adoptés (voir `companions:`).

## Démos de base

| Service | Commande | Vérification | Notes |
|---|---|---|---|
| PostgreSQL | `wslc run -d --name pg -e POSTGRES_PASSWORD=secret -p 5432:5432 -v pgdata:/var/lib/postgresql postgres:18` | `wslc exec pg psql -U postgres -c "select version();"` | À partir de la version 18, le volume déclaré est `/var/lib/postgresql` (`PGDATA` = `/var/lib/postgresql/18/docker`). Pour 17 et avant : `/var/lib/postgresql/data`. Port 5432 et commandes de vérification : standard, absents de la page lue |
| Redis | `wslc run -d --name redis -p 6379:6379 -v redisdata:/data redis:8` | `wslc exec redis redis-cli ping` (attendu : PONG) | Licence RSALv2 / SSPLv1 / AGPLv3 depuis la 8.0 ; alternative libre : `valkey/valkey:9` (pas d'image officielle `valkey`) |
| Solr | `wslc run -d --name solr -p 8983:8983 -v solrdata:/var/solr solr:10.0.0 solr-precreate gettingstarted` | http://localhost:8983/ | Guide officiel : dossier lié `$PWD/solrdata`. Volume nommé jugé plus sûr (droits de l'utilisateur 8983 non documentés) : à tester. Ligne 9.x : `solr:9.10.1` |
| Mailpit | `wslc run -d --name mailpit -p 8025:8025 -p 1025:1025 axllent/mailpit` | Interface http://localhost:8025 ; SMTP sur 1025 | Version v1.31.4 (2026-10-03) ; format du tag Docker non vérifié. Test d'envoi : `curl --url smtp://localhost:1025 --mail-from a@x.test --mail-rcpt b@x.test -T mail.txt` (non issu des pages lues). MailHog n'est plus maintenu |
| SonarQube | `wslc run -d --name sonarqube -p 9000:9000 sonarqube:community` | http://localhost:9000, admin/admin | Exige côté hôte `vm.max_map_count=524288`, `fs.file-max=131072`, `ulimit -n 131072`, `ulimit -u 8192`. Réglage dans la VM WSLC inconnu : **jamais en direct sans test préalable**. Base H2 embarquée, démo seulement. Tag `26.9.0.129388-community` (2026-10-04) |

## Services ajoutés

| Service | Commande | Vérification | Notes |
|---|---|---|---|
| Grafana | `wslc run -d --name grafana -p 3000:3000 -v grafana-data:/var/lib/grafana grafana/grafana:13.2.3` | http://localhost:3000 ; admin/admin puis changement de mot de passe demandé | Version de sécurité (2026-09-29). Base Alpine par défaut (variante `-ubuntu`). AGPL-3.0 |
| Gitea | `wslc run -d --name gitea -p 3001:3000 -p 2222:22 -v gitea-data:/data gitea/gitea:28.0.0` | http://localhost:3001 (assistant d'installation, SQLite par défaut) | `latest` = 28.0 (2026-09-30). Port SSH 22 → 2222 : inférence, pas un texte de la doc. Les images avec et sans root (`-rootless`) sont incompatibles entre elles |
| Meilisearch | `wslc run -d --name meili -p 7700:7700 -e MEILI_ENV=development -v meili-data:/meili_data getmeili/meilisearch:v1.54.3` | http://localhost:7700 (prévisualisation en mode développement) ; `curl http://localhost:7700/health` | Le tag `v1.37` de la doc est périmé ; les tags `v1` et `v1.52` semblent en retard : épingler. Base Alpine |
| Prometheus | `wslc run -d --name prometheus -p 9090:9090 prom/prometheus:v3.15.0` | http://localhost:9090 → Status > Targets ; requête `up` | Le fichier d'exemple du dépôt se collecte lui-même (`localhost:9090`) ; image par défaut non confirmée |
| SeaweedFS (remplace MinIO) | `wslc run -d --name seaweed -p 8333:8333 -p 8888:8888 -p 23646:23646 -v weed-data:/data -e AWS_ACCESS_KEY_ID=admin -e AWS_SECRET_ACCESS_KEY=secret -e S3_BUCKET=my-bucket chrislusf/seaweedfs:4.48` | Interface d'administration http://localhost:23646 ; client S3 : `aws s3 ls --endpoint-url http://localhost:8333` (ligne non fournie par les sources) | Commande par défaut de l'image : `mini -dir=/data` (vérifiée). Sans identifiants : mode « Allow All », sans authentification. Apache 2.0 |

## Plan de ports

Aucune collision une fois Gitea (3001) et Adminer (8081) remappés : Grafana 3000, Gitea 3001, Meilisearch 7700, Keycloak 8080, Adminer 8081, Mailpit 8025 et 1025, Solr 8983, SonarQube 9000, Prometheus 9090, SeaweedFS 8333 / 8888 / 23646, PostgreSQL 5432, Redis 6379, RabbitMQ 5672 et 15672, MongoDB 27017, MariaDB 3306.

## Annexe : repli et suivants, non démontrés

| Service | Commande | Notes |
|---|---|---|
| Adminer | `wslc run -d --name adminer -p 8081:8080 adminer` | Interface http://localhost:8081 ; « Server » = nom du conteneur sur le même réseau (non confirmé sous WSLC) |
| RabbitMQ | `wslc run -d --name rabbit -p 5672:5672 -p 15672:15672 rabbitmq:4-management` | Interface http://localhost:15672 (guest/guest, à tester) ; `RABBITMQ_DEFAULT_USER` / `RABBITMQ_DEFAULT_PASS` : statut de dépréciation ambigu sur 4.x |
| MongoDB | `wslc run -d --name mongo -p 27017:27017 -v mongodata:/data/db mongo` | Éviter les montages depuis `/mnt/c` ; vérification `wslc exec mongo mongosh --eval "db.runCommand({ping:1})"` |
| MariaDB | `wslc run -d --name maria -e MARIADB_ROOT_PASSWORD=secret -p 3306:3306 mariadb:lts` | Vérification `wslc exec maria mariadb -uroot -psecret -e "select 1"` (nom du client supposé) |
| Keycloak (score 3,60) | `wslc run -d --name keycloak -p 8080:8080 -e KC_BOOTSTRAP_ADMIN_USERNAME=admin -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:26.8.0 start-dev` | Console http://localhost:8080/admin ; 2 Go de mémoire conseillés ; `start-dev` : développement seulement |
| RustFS (score 3,50) | `wslc run -d --name rustfs -p 9100:9000 -p 9001:9001 -v rustfs-data:/data -e RUSTFS_ACCESS_KEY=<clé> -e RUSTFS_SECRET_KEY=<secret> rustfs/rustfs:latest /data` | Version 1.0.1 (2026-10-03), très récente ; utilisateur non-root `10001:10001` ; port de l'API remappé en 9100 pour éviter SonarQube (9000) |
| pgAdmin (score 3,65) | `wslc run -d --name pgadmin -e PGADMIN_DEFAULT_EMAIL=admin@example.com -e PGADMIN_DEFAULT_PASSWORD=secret -p 8082:80 dpage/pgadmin4:9.18` | Inutile seul sans PostgreSQL joignable ; port 8082 choisi pour éviter les conflits |
