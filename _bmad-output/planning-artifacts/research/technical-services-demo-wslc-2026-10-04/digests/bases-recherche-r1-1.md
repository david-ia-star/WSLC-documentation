# Digest — bases et recherche : MySQL, Meilisearch, OpenSearch (round 1, assistant 1)

Accès : 2026-10-04. ~20 appels, ~12 sources. Les pages sont résumées par un petit modèle : les chaînes « verbatim » sont celles que l'outil a renvoyées, à revérifier avec `curl` avant l'exposé. **Rien n'a été exécuté dans un conteneur.**

Format : constat | source | éditeur | date | confiance | classe

## 1. MySQL (image officielle `mysql`)
- Image Docker Official Image. Tags supportés : `26.7.0` (latest / innovation), `9.7.2` (LTS), `8.4.11` (stable). | https://hub.docker.com/_/mysql | Docker Hub | page au 2026-10-04 | haute | image/version
- Tag recommandé pour la démo : `mysql:lts` (9.7.2) ou `mysql:latest` (26.7.0). `latest`, `lts`, `innovation`, `oraclelinux9`, `oracle` mis à jour le 2026-09-29T20:03Z. Tailles : latest 272,3 Mo, lts 270,9 Mo (amd64). Multi-architecture. | https://hub.docker.com/v2/repositories/library/mysql/tags?page_size=8&ordering=last_updated | API Docker Hub | 2026-09-29 | haute | version
- MySQL 26.7.0 publiée le 2026-07-28, GA, version Innovation, première version à numérotation calendaire (AA.M.P) après la 9.7 LTS ; prochaine Innovation annoncée : 26.10.0 (octobre 2026). | https://dev.mysql.com/doc/relnotes/mysql/26.7/en/news-26-7-0.html | Oracle/MySQL | 2026-07-28 | moyenne : le titre de l'extrait de recherche dit « (Early Access Release) » mais le résumé dit GA ; conflit non résolu | version/maintenance
- Variable obligatoire : `MYSQL_ROOT_PASSWORD`. Optionnelles : `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_ALLOW_EMPTY_PASSWORD`, `MYSQL_RANDOM_ROOT_PASSWORD`, `MYSQL_ONETIME_PASSWORD`, `MYSQL_INITDB_SKIP_TZINFO`. | https://hub.docker.com/_/mysql | Docker Hub | page au 2026-10-04 | haute | env
- Port : « The image exposes port 3306 ». | https://github.com/docker-library/docs/blob/master/mysql/README.md | docker-library | page au 2026-10-04 | moyenne (formulation du résumé) | port
- Volume `/var/lib/mysql`. Exemple de la doc : `docker run --name some-mysql -v /my/own/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag`. | même README | docker-library | page au 2026-10-04 | haute | volume
- Image de base : Oracle Linux 9 pour tous les tags (« All tags support Oracle Linux 9 base images »), ni Debian ni Alpine. Docker Hub mentionnait aussi des images MySQL 8 sur Debian : non vérifié. | README et https://hub.docker.com/_/mysql | docker-library, Docker Hub | page au 2026-10-04 | moyenne | image
- Piège : « it will not accept incoming connections until such initialization completes » : boucle d'attente ou nouvel essai nécessaire. | README | docker-library | haute | piège
- Piège : `MYSQL_ROOT_PASSWORD` est ignoré si le répertoire de données contient déjà une base : important quand on rejoue une démo avec un volume réutilisé. | https://hub.docker.com/_/mysql | Docker Hub | haute | piège
- Licence : la page renvoie vers mysql.com/about/legal (Community Edition, GPL, non écrit explicitement dans les pages lues). L'image contient d'autres logiciels sous diverses licences. Gratuit pour une démo : très probable mais non vérifié. | README | docker-library | basse | licence
- Maintenance : active (mise à jour du 2026-09-29). MySQL 26.7.0 intègre le plugin Thread Pool à la Community Edition ; abandon d'Enterprise Linux 7. | Docker Hub et dev.mysql.com | Docker Hub, Oracle | 2026-09-29 / 2026-07-28 | haute | maintenance
- Non récupéré : interface web (l'image n'en a pas, non vérifié), minimum mémoire, temps de démarrage, RAM, commande de vérification. `mysql -uroot -p -e "select version()"` est une suggestion de l'assistant, pas une source.

## 2. Meilisearch (`getmeili/meilisearch`)
- Image maintenue par l'éditeur (pas une Docker Official Image). Tags (API) : `latest` et `v1.54.3` poussés le 2026-10-01T10:17Z (105,4 Mo) ; `v1.53.3` et `v1.52.4` le même jour ; `nightly` le 2026-10-04. amd64 et arm64. La page Hub n'a pas de présentation, seulement `docker pull getmeili/meilisearch`. | https://hub.docker.com/v2/repositories/getmeili/meilisearch/tags?page_size=8&ordering=last_updated et https://hub.docker.com/r/getmeili/meilisearch | Docker Hub | 2026-10-01 | haute | image/version
- Dernière release v1.54.3, publiée le 2026-10-01T09:54Z. Note : « This release contains an important stability fix for Meilisearch. Update from v1.54.2 is recommended. » (PR #6658, rétroportée en v1.53.3 et v1.52.4 le même jour). v1.54.2 publiée le 2026-09-29. | https://api.github.com/repos/meilisearch/meilisearch/releases?per_page=4 | Meilisearch (GitHub) | 2026-10-01 | haute | version/maintenance
- Piège possible (moyenne-basse) : dans la liste des tags, `v1` et `v1.52` mis à jour à 10:36 avec la même taille (99,7 Mo) que `v1.52.4` : ils pointent peut-être vers la ligne 1.52.4 et pas 1.54.3. Épingler `:v1.54.3` ou `:latest` et vérifier avec `GET /version`. | API Docker Hub | 2026-10-01 | basse | piège
- Piège : l'exemple de la doc utilise `getmeili/meilisearch:v1.37`, nettement périmé face à v1.54.3 : ne pas copier le tag de la doc. | https://www.meilisearch.com/docs/learn/self_hosted/install_meilisearch_locally | Meilisearch docs | page au 2026-10-04 | haute | piège
- Commande de la doc (verbatim) : `docker run -it --rm -p 7700:7700 -e MEILI_ENV='development' -v $(pwd)/meili_data:/meili_data getmeili/meilisearch:v1.37`. | même URL | Meilisearch docs | haute | env/port/volume
- Port 7700 : le Dockerfile contient `EXPOSE 7700/tcp` et `ENV MEILI_HTTP_ADDR 0.0.0.0:7700`. | https://raw.githubusercontent.com/meilisearch/meilisearch/main/Dockerfile | Meilisearch (GitHub) | page au 2026-10-04 | haute | port
- Volume `/meili_data` : le Dockerfile met `WORKDIR /meili_data` sans instruction VOLUME ; il faut donc passer explicitement un volume ou un montage. Base par défaut `data.ms/`, relative au répertoire de travail. | Dockerfile et https://www.meilisearch.com/docs/resources/self_hosting/configuration/reference | Meilisearch | haute | volume
- Variables obligatoires : aucune en développement. `MEILI_ENV` vaut `development` par défaut. `MEILI_MASTER_KEY` : « a UTF-8 string of at least 16 bytes », « Mandatory in production; optional in development », protège toutes les routes sauf `GET /health`. Le mode production exige la clé. | page de référence de configuration | Meilisearch docs | haute | env
- Interface web : l'« interface de prévisualisation de recherche » est servie sur le port 7700 en mode développement et désactivée en production. Le chemin exact `/` : non vérifié. | même référence | Meilisearch docs | moyenne | ui
- Vérification : `GET /health` sans authentification : `curl http://localhost:7700/health`. Réponse exacte `{"status":"available"}` non vérifiée. | même référence | moyenne | ui
- Image de base : étape de build `rust:1.89-alpine3.22`, exécution `alpine:3.22`, `ENTRYPOINT ["tini", "--"]`, `CMD /bin/meilisearch`. Le Dockerfile `main` peut différer de l'image publiée. | Dockerfile | Meilisearch (GitHub) | moyenne | image
- Réglages hôte : aucun trouvé (pas de `vm.max_map_count`). Mémoire : `MEILI_MAX_INDEXING_MEMORY` vaut par défaut « 2/3 of the available RAM » ; le limiter par exemple avec `-e MEILI_MAX_INDEXING_MEMORY=512Mb` dans une petite VM (suggestion de l'assistant). | référence | moyenne | réglage hôte
- Licence : Community Edition MIT ; les fonctions Enterprise sous BUSL 1.1 ou licence commerciale (« not allowed in production without a commercial agreement »), permises pour les tests et l'évaluation. L'image standard est gratuite pour une démo. | https://github.com/meilisearch/meilisearch | Meilisearch (GitHub) | moyenne | licence
- Télémétrie : `MEILI_NO_ANALYTICS` la désactive. | référence | haute | env
- Maintenance : très active (plusieurs versions par semaine du 2026-09-29 au 2026-10-01). Aucun avis d'archivage ou de dépréciation. | https://github.com/meilisearch/meilisearch/releases | GitHub | 2026-10-01 | haute | maintenance
- Non récupéré : temps de démarrage, RAM, source pour une « histoire de démo ».

## 3. OpenSearch (`opensearchproject/opensearch`) et Dashboards
- Image `opensearchproject/opensearch`. Tags (API) : `latest`, `3` et `3.9.0` poussés le 2026-09-29T21:11Z (1,16 Go). `3.8.0` 2026-09-15, `2.19.6` 2026-09-15, `3.7.0` 2026-07-15. 81 tags au total. | https://hub.docker.com/v2/repositories/opensearchproject/opensearch/tags?page_size=8&ordering=last_updated | API Docker Hub | 2026-09-29 | haute | image/version
- Dernière release 3.9.0 publiée le 2026-09-29T21:54Z. Autres : 3.8.0 (2026-08-05), 2.19.6 (2026-07-06), 3.7.0 (2026-06-09). La ligne 2.x est toujours maintenue. | https://api.github.com/repos/opensearch-project/OpenSearch/releases?per_page=4 | OpenSearch (GitHub) | 2026-09-29 | haute | version/maintenance
- Ports (page Hub) : 9200 (REST) et 9600 (analyseur de performance). | https://hub.docker.com/r/opensearchproject/opensearch | Docker Hub | page au 2026-10-04 | haute | port
- Commande de la doc (verbatim) : `docker run -d -p 9200:9200 -p 9600:9600 -e "discovery.type=single-node" -e "OPENSEARCH_INITIAL_ADMIN_PASSWORD=<custom-admin-password>" opensearchproject/opensearch:latest`. | https://raw.githubusercontent.com/opensearch-project/documentation-website/main/_install-and-configure/install-opensearch/docker.md | OpenSearch (source GitHub de la doc) | page au 2026-10-04 | haute | env
- Variables obligatoires : `OPENSEARCH_INITIAL_ADMIN_PASSWORD` (exigée depuis la 2.12) et `discovery.type=single-node` pour une démo à un conteneur. | page Hub et doc | Docker Hub, OpenSearch | haute | env
- Piège du mot de passe : « minimum length, include multiple character classes, and pass entropy-based strength validation » ; un mot de passe faible fait échouer le démarrage. Règle exacte non vérifiée ; choisir par exemple `Demo-Passw0rd!x9`. | doc | OpenSearch | moyenne | piège
- Sécurité : configuration de démo avec TLS auto-signé et utilisateurs prédéfinis : utiliser `https://` et `-k`. L'ancien admin/admin ne valait que jusqu'à la 2.12. `DISABLE_SECURITY_PLUGIN=true` supprime la sécurité en développement et passe en http. | doc et page Hub | OpenSearch, Docker Hub | haute | piège
- Vérification (verbatim) : `curl https://localhost:9200 -ku admin:"<custom-admin-password>"`. | doc | OpenSearch | haute | ui
- Volume de données : `/usr/share/opensearch/data`. | doc | OpenSearch | haute | volume
- **Réglages hôte : `vm.max_map_count=262144` (« Linux requirement »)** et minimum 4 Go de RAM pour les utilisateurs de Docker Desktop. Ulimits (memlock, nofile) : non renvoyés par le résumé, non vérifiés. Pas de commande sysctl ni de détails de tas. | doc et page Hub | OpenSearch, Docker Hub | moyenne | réglage hôte
- Image de base : Amazon Linux 2023 (2.10.0 et plus), Amazon Linux 2 avant. ~1,1 Go. | page Hub | Docker Hub | haute | image
- Dashboards (interface) : image `opensearchproject/opensearch-dashboards`, tags `latest`, `3` et `3.9.0` poussés le 2026-09-29 (508,9 Mo). Port 5601. Chaîne de connexion : `OPENSEARCH_HOSTS: '["https://opensearch-node1:9200",...]'`. Une démo à deux conteneurs demande un réseau partagé : contrainte pour `wslc` sans compose ; mode de liaison non vérifié. Alternatives : `-e OPENSEARCH_HOSTS` vers une adresse joignable, ou seulement curl et l'API REST. | API Docker Hub et doc | Docker Hub, OpenSearch | 2026-09-29 | moyenne | ui
- Licence : Apache 2.0 (dérivé d'Elasticsearch 7.10.2 et Kibana 7.10.2). Gratuit pour une démo. | page Hub | Docker Hub | haute | licence
- Maintenance : très active (reconstructions d'images mensuelles ou bimensuelles). Aucun avis de dépréciation. | API Hub et releases GitHub | 2026-09-29 | haute | maintenance
- Piège : seule issue trouvée, #978 « Improve 'OpenSearch exited with code 137' OOM error message on OSX », fermée le 2021-09-23 : arrêt par manque de mémoire (code 137) dans des VM Docker à mémoire limitée. Correction : inconnue. | https://github.com/opensearch-project/OpenSearch/issues?q=docker+vm.max_map_count+bootstrap+checks+failed | GitHub | 2021-09-23 | basse | piège
- Non récupéré : temps de démarrage, RAM, source d'« histoire de démo ».

## Pistes à creuser
1. OpenSearch et `wslc` : la VM a-t-elle déjà `vm.max_map_count>=262144` ? À vérifier avec `wslc exec` ou `wsl -e sysctl vm.max_map_count`. Cause la plus probable d'échec de démo.
2. OpenSearch ulimits et tas : `--ulimit memlock=-1:-1`, `nofile` et `OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m` dans le docker.md de la doc.
3. Liaison OpenSearch Dashboards : `wslc` accepte-t-il un réseau utilisateur ou `--link`, ou faut-il une adresse de type `host.docker.internal` ?
4. Conflit « Early Access » / GA de MySQL 26.7.0 : ouvrir la page dev.mysql.com. Prendre `mysql:lts` (9.7.2) évite la question.
5. Meilisearch : vérifier si le tag `v1` est en retard (comparer les digests avec `v1.54.3`) ; lire la PR #6658 avant de choisir un tag.
6. Meilisearch, histoire de démo : le démarrage rapide officiel propose un jeu de données de films et la prévisualisation de recherche sur `/`. Non récupéré.
7. MySQL, mémoire : `--innodb-buffer-pool-size` et `--performance-schema=OFF` pour limiter la RAM. Non vérifié.

## Cherché et non trouvé
- Temps de démarrage et RAM pour les trois : aucune source.
- Consignes verbatim d'ulimit et de tas pour OpenSearch.
- La règle numérique du mot de passe OpenSearch.
- Une mention GPL explicite pour l'image MySQL.
- Preuve officielle d'« histoire de démo » pour les trois candidats.
- Format de réponse officiel de `/health` de Meilisearch.
- Existence de variantes Debian de l'image MySQL (le résumé Hub dit oui ; le README docker-library dit Oracle Linux 9 seulement).
- Preuve d'un problème ancien corrigé via les trackers d'issues (rien d'exploitable).
