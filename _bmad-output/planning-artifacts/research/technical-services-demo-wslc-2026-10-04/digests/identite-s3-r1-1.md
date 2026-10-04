# Digest — identité et stockage S3 : Keycloak, Garage, SeaweedFS, RustFS (round 1, assistant 3)

Accès : 2026-10-04. ~19 appels, WebFetch seul. Les pages sont résumées par un petit modèle ; des erreurs de résumé ont été vues (voir la date de SeaweedFS) : revérifier les commandes critiques en les exécutant. **Rien n'a été exécuté.**

Format : constat | source | éditeur | date | confiance | classe

## 1. Keycloak
- Image `quay.io/keycloak/keycloak:26.8.0`. | https://www.keycloak.org/getting-started/getting-started-docker | keycloak.org | page au 2026-10-04 | haute | image/version
- Commande officielle : `docker run -p 127.0.0.1:8080:8080 -e KC_BOOTSTRAP_ADMIN_USERNAME=admin -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:26.8.0 start-dev`. | même URL | keycloak.org | haute | env/image
- Variables obligatoires : `KC_BOOTSTRAP_ADMIN_USERNAME`, `KC_BOOTSTRAP_ADMIN_PASSWORD` (« required when running in containers »). | https://www.keycloak.org/server/containers | keycloak.org | haute | env
- Version 26.8.0 publiée le 2026-10-01. Précédentes : 26.7.5 (30 sept.), 26.7.4 (16 sept.), 26.7.3 (31 août), surtout des correctifs CVE. | https://github.com/keycloak/keycloak/releases | GitHub / Keycloak | page au 2026-10-04 | moyenne (résumé) | version/maintenance
- Notes 26.8.0 : OID4VCI en préversion, API SCIM activée par défaut, rotation des secrets de client activée par défaut. | https://www.keycloak.org/docs/latest/upgrading/index.html | keycloak.org | moyenne | version
- Ports : 8080 (HTTP, dev), 8443 (HTTPS), 9000 (gestion : santé/métriques via `KC_HEALTH_ENABLED=true` / `KC_METRICS_ENABLED=true`). | https://www.keycloak.org/server/containers | keycloak.org | haute | port
- Vérification et connexion : `http://localhost:8080/admin`, identifiants admin/admin (selon la commande ci-dessus). Console de compte : `http://localhost:8080/realms/myrealm/account`. Console d'administration web intégrée, même port 8080. | getting-started-docker | haute | ui
- Volume : import de domaines (realms) en montant `/opt/keycloak/data/import` avec `--import-realm`. Le mode dev n'exige aucun volume (H2 embarquée, éphémère sans montage ; chemin de persistance non récupéré). | /server/containers | moyenne | volume
- Mémoire : « maximum heap size as 70% of the total container memory », tas initial 50 % ; « Recommended minimum for production is 2 GB total container memory ». | /server/containers | haute | réglage hôte
- `start-dev` = « Development/testing only with insecure defaults ». | /server/containers | haute | piège
- Pièges du guide de mise à niveau : 26.0.0 a changé la configuration de l'administrateur initial (les anciennes variables `KEYCLOAK_ADMIN` remplacées par `KC_BOOTSTRAP_ADMIN_*` ; le détail du renommage n'est pas confirmé) ; 26.0.0 a supprimé Hostname v1 et l'option proxy ; 26.0.0 persiste par défaut toutes les sessions utilisateur ; 25.0.0 a introduit le port de gestion 9000. | https://www.keycloak.org/docs/latest/upgrading/index.html | keycloak.org | moyenne | piège
- Licence Apache 2.0 ; code de conduite CNCF. Aucun changement de licence trouvé. | https://github.com/keycloak/keycloak | GitHub | moyenne | licence
- Maintenance : très active (37,1 k étoiles, 4 versions en ~5 semaines). | GitHub | haute | maintenance
- Non récupéré : temps de démarrage, RAM au repos, image de base (UBI attendue, non vérifiée), date exacte du tag quay.

## 2. Garage
- Image `dxflrs/garage:v2.3.0` sur Docker Hub (espace de noms dxflrs, pas l'espace officiel). Tag poussé le 2026-04-16T20:22Z, ~5,5 mois : dépasse la barre de fraîcheur d'un mois, signalé. 27,1 Mo, multi-architecture. | https://hub.docker.com/v2/repositories/dxflrs/garage/tags/v2.3.0 | API Docker Hub | page au 2026-10-04 | haute | image/version
- Des tags plus récents existent, sous forme de hash de commit (dernier : 2026-10-04), pas de versions publiées. Que v2.3.0 soit encore la dernière stable : non vérifié (page des releases de deuxfleurs bloquée par Anubis). | API tags Docker Hub | moyenne | maintenance
- **Un fichier de configuration est obligatoire.** « By default, Garage looks for its configuration file in /etc/garage.toml. » Les variables d'environnement seules ne le remplacent pas. | https://garagehq.deuxfleurs.fr/documentation/quick-start/ | Deuxfleurs | page au 2026-10-04 | haute | env/piège
- `garage.toml` minimal du démarrage rapide (verbatim) : `metadata_dir = "/tmp/meta"`, `data_dir = "/tmp/data"`, `db_engine = "sqlite"`, `replication_factor = 1`, `rpc_bind_addr = "[::]:3901"`, `rpc_public_addr = "127.0.0.1:3901"`, `rpc_secret = "$(openssl rand -hex 32)"`, `[s3_api]` avec `s3_region = "garage"`, `api_bind_addr = "[::]:3900"`, `root_domain = ".s3.garage.localhost"`, `[s3_web]` `bind_addr = "[::]:3902"`, `[admin]` `api_bind_addr = "[::]:3903"`, `admin_token`, `metrics_token`. NB : `$(...)` est une substitution du shell : le fichier doit être généré par un shell, pas monté tel quel. | quick-start | haute | env
- Pas d'étape manuelle de « layout » en mode conteneur unique : « The `--single-node` flag instructs Garage to automatically configure a single-node cluster without data replication. The `--default-bucket` flag instructs Garage to create a default access key and a default bucket using the environment variables we defined above. » | quick-start | haute | piège (réglé par de nouvelles options)
- Commande Docker officielle (verbatim) : `docker run -d --name garage-container -p 3900:3900 -p 3901:3901 -p 3902:3902 -p 3903:3903 -v $(pwd)/garage.toml:/etc/garage.toml -e GARAGE_DEFAULT_ACCESS_KEY -e GARAGE_DEFAULT_SECRET_KEY -e GARAGE_DEFAULT_BUCKET dxflrs/garage:v2.3.0 /garage server --single-node --default-bucket`. | quick-start | haute | env/image
- Format des identifiants : `GARAGE_DEFAULT_ACCESS_KEY="GK$(openssl rand -hex 16)"`, `GARAGE_DEFAULT_SECRET_KEY="$(openssl rand -hex 32)"`, `GARAGE_DEFAULT_BUCKET="default-bucket"`. Préfixe « GK » ; des clés courtes arbitraires comme « admin » : non vérifié (probablement le format GK + hexadécimal exigé). | quick-start | moyenne | env
- Ports : 3900 API S3, 3901 RPC, 3902 S3 web (sites statiques), 3903 API d'admin (+ métriques). Aucune interface web de gestion trouvée (3902 sert des sites statiques). | quick-start | moyenne | port/ui
- Vérification : awscli >= 1.29.0 ou >= 2.13.0 respecte `export AWS_ENDPOINT_URL='http://localhost:3900'`, sinon `--endpoint-url` à chaque appel. La ligne exacte `aws s3 ls` n'était pas dans l'extrait. | quick-start | moyenne | vérification
- Volumes : chemins issus de `metadata_dir` / `data_dir` du fichier (l'exemple utilise /tmp/meta et /tmp/data : pour persister, monter un volume et changer les chemins). Autres variables : `GARAGE_RPC_SECRET`, `GARAGE_ADMIN_TOKEN`, `GARAGE_METRICS_TOKEN` (+ variantes `_FILE`). | https://garagehq.deuxfleurs.fr/documentation/reference-manual/configuration/ | Deuxfleurs | page au 2026-10-04 | haute | env/volume
- Mémoire : `block_ram_buffer_max` vaut 256 MiB par défaut ; moteur LMDB par défaut depuis 0.9.0 (le démarrage rapide utilise sqlite). Pas de RAM ni de temps de démarrage. | référence de configuration | moyenne | réglage hôte
- Licence : non récupérée (non vérifiée). Image de base : non récupérée (très petite, ~27 Mo, probablement statique ; non vérifié).

## 3. SeaweedFS
- Image `chrislusf/seaweedfs` (Docker Hub). Tag `4.48` poussé le 2026-09-28T18:58Z, 92,4 Mo, architectures amd64/arm64/386/arm. | https://hub.docker.com/v2/repositories/chrislusf/seaweedfs/tags/4.48 | API Docker Hub | page au 2026-10-04 | haute | image/version
- Release GitHub 4.48 publiée le 2026-09-28T15:53Z, pas une préversion. (Le résumé a affiché à tort « 2024 » sur la page des releases ; se fier à la date de l'API. Précédentes : 4.47, 4.46, avec des correctifs de sécurité S3.) | https://api.github.com/repos/seaweedfs/seaweedfs/releases?per_page=3 | GitHub | page au 2026-10-04 | haute | version
- Démarrage rapide officiel (README), verbatim : `docker run -p 8333:8333 -v weed-data:/data -e AWS_ACCESS_KEY_ID=admin -e AWS_SECRET_ACCESS_KEY=secret -e S3_BUCKET=my-bucket chrislusf/seaweedfs`. Pas d'argument `server -s3` : l'image semble démarrer par défaut en mode tout-en-un « weed mini » (le README le dit réglé automatiquement pour un seul nœud). Que la commande par défaut soit weed mini est INFÉRÉ, pas écrit : moyenne. | https://github.com/seaweedfs/seaweedfs (+ README brut) | SeaweedFS | page au 2026-10-04 | moyenne | image/env
- Wiki weed mini : processus tout-en-un (master, volume, filer, S3, WebDAV, interface d'admin, worker). Exemple Docker : `docker run -d --name weed-mini -p 8333:8333 -p 8888:8888 -p 9333:9333 -p 23646:23646 -v weed-data:/data -e AWS_ACCESS_KEY_ID=admin -e AWS_SECRET_ACCESS_KEY=secret -e S3_BUCKET=my-bucket chrislusf/seaweedfs`. | https://github.com/seaweedfs/seaweedfs/wiki/Quick-Start-with-weed-mini | wiki SeaweedFS | page au 2026-10-04 | moyenne | env/port
- Ports : S3 8333, interface d'admin 23646, interface Filer 8888, interface Master 9333, Volume 9340, WebDAV 7333. Interface web : oui (admin 23646, filer 8888, master 9333). | wiki weed mini | moyenne | port/ui
- Volume : `/data`. | wiki | moyenne | volume
- Identifiants : `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` alimentent l'identité ; `S3_BUCKET` crée le seau. Sans identifiants : mode « Allow All » (sans authentification), piège si la démo montre l'authentification. | wiki | moyenne | env/piège
- `weed server -s3` lance master + volume + filer + passerelle S3 ensemble (ancienne page du wiki Amazon-S3-API). | https://github.com/seaweedfs/seaweedfs/wiki/Amazon-S3-API | wiki | moyenne | env
- Licence : Apache 2.0 (« Licensed under the Apache License, Version 2.0 »). Une édition Enterprise payante existe séparément ; l'édition OSS reste Apache 2.0, aucun changement de licence vu. | README | haute | licence
- Le README dit (verbatim) : « as Apr 25, 2026 MinIO ceased development. It's strongly discouraged to use that unmaintained software with multiple security bugs. RustFS is a MinIO reimplementation in Rust, Apache 2.0 licensed and still developed ». | README | haute | maintenance (corrobore l'état de MinIO)
- Maintenance : très active (4.46, 4.47, 4.48 en septembre 2026).
- Non récupéré : image de base, RAM, temps de démarrage, la ligne `aws s3 ls --endpoint-url http://localhost:8333`, et si l'ENTRYPOINT exécute vraiment weed mini.

## 4. RustFS (option S3 supplémentaire)
- Image `rustfs/rustfs` (Docker Hub). Tags `1.0.1` et `latest` poussés le 2026-10-03T05:01Z (~1 jour), 111 Mo ; variantes `-glibc` 160 Mo. Tags de préversion (`1.0.1-preview.17`) : la 1.0.0 est-elle GA : non vérifié. | https://hub.docker.com/v2/repositories/rustfs/rustfs/tags | Docker Hub | page au 2026-10-04 | haute | image/version
- Démarrage rapide du README, verbatim : `docker run -d -p 9000:9000 -p 9001:9001 -v $(pwd)/data:/data -v $(pwd)/logs:/logs rustfs/rustfs:latest`. | https://github.com/rustfs/rustfs | GitHub | page au 2026-10-04 | moyenne | image
- Version de la doc : `docker run -d --name rustfs -p 9000:9000 -p 9001:9001 -v rustfs-data:/data -e RUSTFS_ACCESS_KEY="<your-access-key>" -e RUSTFS_SECRET_KEY="<your-secret-key>" rustfs/rustfs:latest /data`. | https://docs.rustfs.com/installation/docker/ | doc RustFS | page au 2026-10-04 | moyenne | env
- Ports 9000 (API S3), 9001 (console web `http://localhost:9001`). Identifiants par défaut rustfsadmin / rustfsadmin ; la doc dit « Do not use the well-known `rustfsadmin` value ». | README + doc | haute | port/ui
- Piège : le conteneur s'exécute avec l'utilisateur non-root `rustfs` (uid/gid `10001:10001`) : les dossiers de l'hôte montés doivent lui appartenir (un volume nommé l'évite ; pertinent avec `wslc -v` sur des chemins de l'hôte). | doc + README | haute | piège
- Licence Apache 2.0. Le README se dit « production-ready » : formulation commerciale (affirmation de l'éditeur, confiance basse). Le README de SeaweedFS parle d'un format disque « byte-compatible » avec MinIO (affirmation de tiers, moyenne). | README | moyenne | licence
- Non récupéré : RAM, temps de démarrage, image de base, date exacte de la 1.0.0, pièges des issues, ligne de vérification aws-cli (API des releases GitHub en 403).

## Pistes à creuser
1. Garage : confirmer la dernière version stable et sa date sur https://git.deuxfleurs.fr/Deuxfleurs/garage (bloqué par Anubis ; essayer un miroir GitHub ou le journal du projet) ; `--single-node --default-bucket` dans v2.3.0 (cité par le démarrage rapide, donc probablement oui ; un test d'exécution est conseillé).
2. SeaweedFS : confirmer l'ENTRYPOINT / commande par défaut (weed mini ?) dans `docker/Dockerfile` et `entrypoint.sh`, et la RAM/le démarrage de weed mini. Repli : `server -s3`, documenté sur l'ancienne page du wiki.
3. RustFS : releases et issues sur github.com (API en 403) pour la date de la 1.0.0 GA et les écarts de compatibilité S3.
4. Keycloak : page de tag quay.io pour la date de 26.8.0, l'image de base, la mémoire/le démarrage de `start-dev`.
5. Exécuter `aws s3 ls --endpoint-url http://localhost:<port>` pour chaque candidat S3 : aucune source ne donne la ligne exacte.

## Cherché et non trouvé
- Temps de démarrage et RAM au repos pour les quatre.
- Images de base pour les quatre.
- Texte de licence de Garage (AGPL connu seulement de la formation : non vérifié).
- Commandes exactes `aws s3 ls --endpoint-url` dans les docs officielles (seulement `AWS_ENDPOINT_URL` pour Garage).
- Réglages hôte : aucune source n'en mentionne (Keycloak : 2 Go recommandés en production seulement).
- Pièges des issues : non recherchés (WebSearch non utilisé). Détail du renommage `KEYCLOAK_ADMIN` → `KC_BOOTSTRAP_ADMIN_*` au-delà de « 26.0.0 a changé » : non confirmé.
- LocalStack S3 : non étudié.

Sources (~15 pages) : keycloak.org (getting-started-docker, /server/containers, /docs/latest/upgrading), github.com/keycloak/keycloak (+ releases), garagehq.deuxfleurs.fr (quick-start, référence de configuration), API hub.docker.com (dxflrs/garage, chrislusf/seaweedfs, rustfs/rustfs), github.com/seaweedfs/seaweedfs (README, README brut, API des releases, wiki Amazon-S3-API, wiki weed mini), github.com/rustfs/rustfs, docs.rustfs.com/installation/docker.
