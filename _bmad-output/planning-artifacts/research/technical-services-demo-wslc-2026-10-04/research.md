---
title: 'technical research: Services candidats pour les démos WSLC'
type: 'technical'
topic: 'Services candidats pour les démos WSLC'
decision: "Choisir les services à ajouter à CAP-3 de l'exposé WSLC (15-20 min, développeurs Docker)"
source: 'native run'
shape: 'select'
status: complete
preset: 'standard'
validation: 'normal'
created: '2026-10-04'
updated: '2026-10-04'
claims_verified: 4
claims_unverified: 6
---

# Recherche technique : services candidats pour les démos WSLC

**Décision servie :** choisir les services à ajouter à CAP-3 (PostgreSQL, Redis, Solr, Mailpit, SonarQube sont déjà retenus) pour un exposé de 15 à 20 minutes devant des développeurs qui connaissent Docker.

**Limites de méthode :** aucune commande n'a été exécutée, ni sous WSLC ni ailleurs. Les pages ont été lues via un outil qui les résume (des erreurs de résumé ont été constatées une fois). Aucune source ne donne de temps de démarrage ni de consommation de mémoire mesurés : les notes « léger » reposent sur la taille des images et les recommandations des éditeurs, et sont donc un jugement, pas une mesure. Les scores de la matrice sont mon jugement à partir des preuves lues ; tu peux les repondérer.

## Synthèse

- **Cinq services à ajouter, par ordre de score :** Grafana (4,40), Gitea (4,35), Meilisearch (4,10), Prometheus (4,00), SeaweedFS (3,85). Tous ont une interface web à montrer, démarrent en une commande `wslc run` et n'exigent aucun réglage de la machine hôte [1][7][10][5][14].
- **Écart Grafana / Gitea : 0,05 point, c'est-à-dire un ex æquo.** Ils sont tous deux recommandés, donc l'ordre n'a pas d'importance pour la décision.
- **OpenSearch est écarté** : il exige `vm.max_map_count=262144`, le même problème que SonarQube, et une image de 1,1 Go avec 4 Go de RAM conseillés [20]. **Garage est écarté** : fichier de configuration obligatoire, pas d'interface web, version stable actuelle non confirmée [18][19]. **MySQL est écarté** : aucune interface web, et il fait double emploi avec MariaDB et Adminer déjà couverts [22].
- **Remplaçant de MinIO pour la démo S3 : SeaweedFS** (Apache 2.0, une seule commande, interface d'administration, publié le 28 septembre 2026). Suivant : RustFS, trop récent (1.0.1 du 3 octobre) pour une démo sans test [14][15][16][17].
- **Plan de ports à fixer** (Grafana et Gitea utilisent tous deux le 3000 par défaut) : voir la section « Informations croisées ».

## 1. Cadre de choix

Cadre approuvé avant la recherche.

**Critères éliminatoires :** image maintenue (version de moins de 12 mois) ; un seul conteneur suffit ; aucun réglage hôte obligatoire, ou signalé comme risque ; licence gratuite pour une démo.

**Critères pondérés (notes de 0 à 5) :** interface web à montrer 25 % ; démarrage rapide et léger 20 % ; utile pour des développeurs 20 % ; persistance simple 15 % ; pièges connus faibles 10 % ; histoire à raconter 10 %.

## 2. Filtre des candidats

| Candidat | Verdict du filtre | Raison | Source |
|---|---|---|---|
| OpenSearch | **Écarté** | Exige `vm.max_map_count=262144` côté hôte ; image 1,1 Go, 4 Go de RAM conseillés ; l'interface (Dashboards) demande un second conteneur et un réseau partagé non vérifié sous WSLC. Le projet lui-même est actif (3.9.0 du 2026-09-29) : l'écart est technique, pas un problème de maintenance | [20][21] |
| Garage | **Écarté** | Fichier `garage.toml` obligatoire (contient une substitution de shell `$(openssl rand -hex 32)`) ; pas d'interface web de gestion ; tag v2.3.0 vieux de 5,5 mois, stabilité actuelle non confirmée | [18][19] |
| MySQL | **Écarté** (score 2,85) | Aucune interface web ; image Oracle Linux 9 ; double emploi avec MariaDB et Adminer ; mot de passe racine ignoré si le volume est réutilisé | [22][23] |
| Les autres | Retenus pour notation | Passent tous les critères éliminatoires | |

## 3. Preuves par critère (candidats retenus)

| Service | Image et version | Interface web | Hôte et mémoire | Licence | Source |
|---|---|---|---|---|---|
| Grafana | `grafana/grafana` 13.2.3 (2026-09-29, version de sécurité), base Alpine par défaut, 460 Mo | Port 3000, admin/admin puis changement de mot de passe demandé | Aucun réglage hôte trouvé | AGPL-3.0 | [1][2][3][4] |
| Gitea | `gitea/gitea` 28.0.0 (`latest` = 28.0, 2026-09-30) ; SQLite par défaut | Port 3000, assistant d'installation | Aucun réglage hôte trouvé | MIT (non vérifié) | [7][8][9] |
| Meilisearch | `getmeili/meilisearch` v1.54.3 (2026-10-01), 105 Mo, base Alpine | Prévisualisation de recherche sur 7700 en mode développement | Aucun réglage hôte ; indexation limitable par `MEILI_MAX_INDEXING_MEMORY` | MIT (édition communautaire) | [10][11] |
| Prometheus | `prom/prometheus` v3.15.0 (2026-09-25), ~100 Mo | Port 9090 | Aucun réglage hôte trouvé | Apache 2.0 | [5][6] |
| SeaweedFS | `chrislusf/seaweedfs` 4.48 (2026-09-28), base Alpine, commande par défaut `mini -dir=/data` | Administration 23646, filer 8888, master 9333 | Aucun réglage hôte trouvé | Apache 2.0 | [14][15][16] |
| Keycloak | `quay.io/keycloak/keycloak` 26.8.0 (2026-10-01), mode `start-dev` | Console d'administration sur 8080 | 2 Go recommandés en production ; tas à 70 % de la mémoire du conteneur | Apache 2.0 | [12][13] |
| pgAdmin | `dpage/pgadmin4` 9.18 (2026-09-17, correctifs de sécurité) | Port 80 | Aucun réglage hôte trouvé | licence PostgreSQL (non vérifiée) | [24][25] |
| RustFS | `rustfs/rustfs` 1.0.1 (2026-10-03) | Console sur 9001 | Aucun réglage hôte trouvé | Apache 2.0 | [17] |

Le temps de démarrage et la RAM au repos n'ont été rapportés par aucune source pour aucun candidat.

## 4. Matrice pondérée

Notes de 0 à 5 par critère ; total = somme pondérée. Les colonnes suivent l'ordre des poids : interface 25, léger 20, utile 20, persistance 15, pièges 10, histoire 10.

| Service | Interface | Léger | Utile | Persist. | Pièges | Histoire | **Total** |
|---|---|---|---|---|---|---|---|
| **Grafana** | 5 | 4 | 4 | 5 | 4 | 4 | **4,40** |
| **Gitea** | 5 | 4 | 5 | 4 | 3 | 4 | **4,35** |
| **Meilisearch** | 4 | 5 | 4 | 4 | 3 | 4 | **4,10** |
| **Prometheus** | 4 | 5 | 4 | 4 | 3 | 3 | **4,00** |
| **SeaweedFS** | 4 | 4 | 3 | 5 | 3 | 4 | **3,85** |
| pgAdmin | 5 | 3 | 4 | 4 | 2 | 2 | 3,65 |
| Keycloak | 5 | 2 | 4 | 3 | 3 | 4 | 3,60 |
| RustFS | 4 | 4 | 3 | 4 | 2 | 3 | 3,50 |
| MySQL (écarté) | 0 | 4 | 4 | 5 | 3 | 2 | 2,85 |
| OpenSearch (écarté) | 2 | 1 | 4 | 4 | 1 | 3 | 2,50 |
| Garage (écarté) | 0 | 5 | 3 | 2 | 1 | 3 | 2,30 |

**Ce qui justifie les notes les plus faibles :**
- *Pièges de Grafana (4)* : image Alpine par défaut (variante `-ubuntu` disponible) [2]. *Gitea (3)* : deux registres et deux variantes d'image incompatibles entre elles (avec et sans root), numérotation passée de 1.x à 28 [7][9]. *Meilisearch (3)* : le tag de la doc (`v1.37`) est périmé et les tags `v1` et `v1.52` semblent en retard [10]. *SeaweedFS (3)* : sans identifiants, le mode « Allow All » désactive l'authentification [14].
- *Keycloak, léger (2)* : application Java, 2 Go recommandés en production [12].
- *pgAdmin (2 en histoire)* : redondant avec Adminer déjà prévu, et inutile seul sans PostgreSQL.
- *RustFS (2 en pièges)* : version 1.0.1 publiée il y a un jour ; utilisateur non-root `10001:10001` qui complique les montages de dossiers de l'hôte [17].

## 5. Coût et dépendances

- **Coût : nul** pour tous les candidats retenus (licences libres : Apache 2.0, MIT, AGPL-3.0 pour Grafana). L'AGPL de Grafana n'impose rien pour une démo non modifiée [4].
- **Coût de sortie :** négligeable ; ce sont des démos de conteneurs sans données à conserver.
- **Dépendance cachée :** Grafana et Prometheus ne montrent de vraies métriques que s'ils se parlent, donc avec deux conteneurs et un nom d'hôte ou un réseau dont le fonctionnement sous WSLC n'est pas documenté. Voir les recommandations.

## 6. Verdict

**Les cinq à ajouter à CAP-3 :**
1. **Grafana** (4,40) : `wslc run -d --name grafana -p 3000:3000 -v grafana-data:/var/lib/grafana grafana/grafana:13.2.3` [1][2].
2. **Gitea** (4,35) : `wslc run -d --name gitea -p 3001:3000 -p 2222:22 -v gitea-data:/data gitea/gitea:28.0.0`. Le port 3001 évite le conflit avec Grafana ; la correspondance du port SSH 22 est une inférence de l'assistant, pas un texte de la doc [7][9].
3. **Meilisearch** (4,10) : `wslc run -d --name meili -p 7700:7700 -e MEILI_ENV=development -v meili-data:/meili_data getmeili/meilisearch:v1.54.3`. Ne pas reprendre le tag `v1.37` de la doc [10][11].
4. **Prometheus** (4,00) : `wslc run -d --name prometheus -p 9090:9090 prom/prometheus:v3.15.0`. Le fichier d'exemple du dépôt se collecte lui-même (`localhost:9090`) ; que l'image utilise ce fichier par défaut n'est pas confirmé : vérifier la page Status > Targets [5][6].
5. **SeaweedFS** (3,85), remplaçant S3 de MinIO : `wslc run -d --name seaweed -p 8333:8333 -p 8888:8888 -p 23646:23646 -v weed-data:/data -e AWS_ACCESS_KEY_ID=admin -e AWS_SECRET_ACCESS_KEY=secret -e S3_BUCKET=my-bucket chrislusf/seaweedfs:4.48` [14][15].

**Toutes ces commandes sont assemblées à partir des documentations ; aucune n'a été testée.**

**Suivants et conditions où ils passent devant :**
- **Keycloak** (3,60) gagne si l'exposé veut un volet « identité et authentification » : `wslc run -d --name keycloak -p 8080:8080 -e KC_BOOTSTRAP_ADMIN_USERNAME=admin -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:26.8.0 start-dev`. Prévoir une machine avec au moins 2 Go de mémoire pour la VM WSLC [12].
- **RustFS** (3,50) passe devant SeaweedFS si SeaweedFS pose problème à l'exécution ; à réévaluer dans quelques semaines, la version 1.0.1 étant toute récente [17].
- **pgAdmin** (3,65) ne vaut que si tu veux montrer une interface graphique pour PostgreSQL ; Adminer (déjà prévu) fait le même travail avec moins de réglages.

**Argument le plus fort contre ce choix :** les scores de « léger » ne reposent sur aucune mesure, et cinq services de plus dépassent ce qu'on peut montrer en 15 à 20 minutes. Il faut en montrer deux ou trois en direct, et laisser les autres au document.

**Couverture la moins chère contre l'erreur :** tester les cinq commandes sur une seule machine Windows une fois, avant l'exposé, et garder RabbitMQ, MongoDB, MariaDB et Adminer (déjà documentés dans la première recherche) comme plan de repli.

## Informations croisées entre critères

- **Conflits de ports** si tout tourne en même temps : Grafana 3000, Gitea 3001 (remappé), SonarQube 9000, Prometheus 9090, Mailpit 8025 et 1025, Solr 8983, Meilisearch 7700, Keycloak 8080, SeaweedFS 8333 / 8888 / 23646, PostgreSQL 5432, Redis 6379, Adminer 8081 (remappé). Aucun doublon une fois Gitea et Adminer remappés.
- **Une démo à plusieurs conteneurs reste à valider** : ni la résolution de noms entre conteneurs ni l'adresse de l'hôte vue d'un conteneur ne sont documentées pour WSLC (voir le premier rapport). Cela touche le couple Grafana et Prometheus, et pgAdmin avec PostgreSQL. Tant que ce n'est pas testé, présenter chaque service seul.
- **Trois services partagent le défaut « base Alpine »** (Grafana par défaut, Meilisearch, SeaweedFS). Le premier rapport signale des issues ouvertes sur 3.0.1 qui touchent surtout `--gpus all` en Alpine ; elles ne concernent pas ces démos, mais autant les connaître.

## Preuves contraires

La passe red team n'a pas été lancée (réglage `off`). Aucune source lue ne contredit le choix. Un avertissement : une source (README de SeaweedFS) recommande RustFS comme successeur de MinIO, ce qui plaide pour RustFS plutôt que SeaweedFS [14] ; je garde SeaweedFS parce que RustFS est publié depuis un jour et que sa version 1.0.0 n'est pas confirmée [17].

## Recommandations

1. **Ajouter à CAP-3 : Grafana, Gitea, Meilisearch, Prometheus et SeaweedFS** comme services supplémentaires, avec leur étiquette « à tester ». Confiance moyenne : les caractéristiques des images sont vérifiées pour quatre points, les autres reposent sur une source unique.
2. **Fixer le plan de ports** ci-dessus dans le document pour éviter les collisions pendant la démo.
3. **Tester d'abord** : la résolution entre conteneurs (Grafana vers Prometheus), l'auto-collecte de Prometheus dans l'image, `localhost:7700` pour Meilisearch, la commande SeaweedFS avec `aws s3 ls --endpoint-url http://localhost:8333` (ligne non fournie par les sources).
4. **Écarter OpenSearch, Garage et MySQL** de la liste de démos (voir le filtre).
5. **Pour le document :** marquer chaque service « testé » ou « à tester », avec la version de l'image et la date, et revérifier les tags la veille de l'exposé.

## Questions ouvertes

| Question | Pour y répondre |
|---|---|
| Temps de démarrage et RAM réels de chaque service | Mesurer sur la machine de l'exposé |
| L'image `prom/prometheus` se collecte-t-elle elle-même par défaut ? | Démarrer et ouvrir Status > Targets |
| Licence MIT de Gitea ; licence de pgAdmin | Lire les fichiers LICENSE des dépôts |
| La 1.0.0 de RustFS est-elle une version stable ? | Releases GitHub (l'API a répondu 403) |
| Garage : quelle est la dernière version stable ? | Miroir GitHub ou journal du projet |
| Le tag `v1` de Meilisearch est-il en retard sur v1.54.3 ? | Comparer les digests des tags |
| Commande `aws s3 ls --endpoint-url` pour SeaweedFS | Test réel |
| Le DNS entre conteneurs fonctionne-t-il sous WSLC ? | Test réel (déjà noté dans le premier rapport) |

## Annexe des sources

Date d'accès de toutes les sources : 2026-10-04.

| [n] | Constat soutenu | Éditeur | Date de publication | Confiance |
|---|---|---|---|---|
| [1] | Image Grafana, port 3000, tags | [Docker Hub — grafana/grafana](https://hub.docker.com/r/grafana/grafana) | page au 2026-10-04 | haute |
| [2] | Volume `/var/lib/grafana`, base Alpine, variables | [Grafana Labs — configure-docker](https://grafana.com/docs/grafana/latest/setup-grafana/configure-docker/) | page au 2026-10-04 | haute |
| [3] | Identifiants admin/admin, invite de changement de mot de passe | [Grafana Labs — sign-in](https://grafana.com/docs/grafana/latest/setup-grafana/sign-in-to-grafana/) | page au 2026-10-04 | haute (relu par le lead) |
| [4] | Grafana 13.2.3 (sécurité), licence AGPL-3.0 | [GitHub grafana/grafana — releases](https://api.github.com/repos/grafana/grafana/releases/latest) et [LICENSE](https://raw.githubusercontent.com/grafana/grafana/main/LICENSE) | 2026-09-29 | haute |
| [5] | Image, port, volumes et licence de Prometheus | [Prometheus — installation](https://prometheus.io/docs/prometheus/latest/installation/) et [Docker Hub](https://hub.docker.com/r/prom/prometheus) | page au 2026-10-04 | haute |
| [6] | Fichier d'exemple se collectant lui-même | [GitHub prometheus/prometheus — prometheus.yml](https://raw.githubusercontent.com/prometheus/prometheus/main/documentation/examples/prometheus.yml) | page au 2026-10-04 | moyenne (image non confirmée) |
| [7] | Gitea : image, tags, SQLite par défaut | [Gitea — Docker](https://docs.gitea.com/installation/install-with-docker) et [Docker Hub — tags](https://hub.docker.com/v2/repositories/gitea/gitea/tags?page_size=10&ordering=last_updated) | 2026-09-30 | haute (relu par le lead) |
| [8] | Gitea v28.0.0 | [GitHub go-gitea/gitea — releases](https://api.github.com/repos/go-gitea/gitea/releases/latest) | 2026-09-29 | haute |
| [9] | Gitea sans root, ports et volumes | [Gitea — Docker sans root](https://docs.gitea.com/installation/install-with-docker-rootless) | page au 2026-10-04 | haute |
| [10] | Meilisearch : image, commande, volume, interface, licence | [Meilisearch — doc](https://www.meilisearch.com/docs/learn/self_hosted/install_meilisearch_locally), [référence](https://www.meilisearch.com/docs/resources/self_hosting/configuration/reference), [Dockerfile](https://raw.githubusercontent.com/meilisearch/meilisearch/main/Dockerfile) et [Docker Hub](https://hub.docker.com/r/getmeili/meilisearch) | 2026-10-01 | haute (interface : moyenne) |
| [11] | Meilisearch v1.54.3 | [GitHub meilisearch/meilisearch — releases](https://api.github.com/repos/meilisearch/meilisearch/releases?per_page=4) | 2026-10-01 | haute |
| [12] | Keycloak : image, commande, variables, mémoire | [Keycloak — Docker](https://www.keycloak.org/getting-started/getting-started-docker) et [conteneurs](https://www.keycloak.org/server/containers) | page au 2026-10-04 | haute |
| [13] | Keycloak 26.8.0 | [GitHub keycloak/keycloak — releases](https://github.com/keycloak/keycloak/releases) | 2026-10-01 | moyenne (page résumée) |
| [14] | SeaweedFS : démarrage rapide, ports, identifiants, licence, mention de MinIO | [GitHub seaweedfs/seaweedfs](https://github.com/seaweedfs/seaweedfs) et [wiki weed mini](https://github.com/seaweedfs/seaweedfs/wiki/Quick-Start-with-weed-mini) | page au 2026-10-04 | moyenne-haute |
| [15] | Commande par défaut `mini -dir=/data`, base Alpine, volume `/data` | [GitHub seaweedfs — Dockerfile](https://raw.githubusercontent.com/seaweedfs/seaweedfs/master/docker/Dockerfile.go_build) | page au 2026-10-04 | haute (relu par le lead) |
| [16] | SeaweedFS 4.48 | [GitHub seaweedfs — releases](https://api.github.com/repos/seaweedfs/seaweedfs/releases?per_page=3) et [Docker Hub](https://hub.docker.com/v2/repositories/chrislusf/seaweedfs/tags/4.48) | 2026-09-28 | haute |
| [17] | RustFS 1.0.1, ports, utilisateur non-root, licence | [GitHub rustfs/rustfs](https://github.com/rustfs/rustfs), [doc](https://docs.rustfs.com/installation/docker/) et [Docker Hub](https://hub.docker.com/v2/repositories/rustfs/rustfs/tags) | 2026-10-03 | moyenne |
| [18] | Garage : fichier de configuration obligatoire, options `--single-node` | [Garage — démarrage rapide](https://garagehq.deuxfleurs.fr/documentation/quick-start/) et [référence](https://garagehq.deuxfleurs.fr/documentation/reference-manual/configuration/) | page au 2026-10-04 | haute |
| [19] | Garage v2.3.0 du 2026-04-16 | [Docker Hub — dxflrs/garage](https://hub.docker.com/v2/repositories/dxflrs/garage/tags/v2.3.0) | 2026-04-16 | haute |
| [20] | OpenSearch : `vm.max_map_count`, mot de passe, ports, taille | [OpenSearch — doc Docker](https://raw.githubusercontent.com/opensearch-project/documentation-website/main/_install-and-configure/install-opensearch/docker.md) et [Docker Hub](https://hub.docker.com/r/opensearchproject/opensearch) | 2026-09-29 | moyenne |
| [21] | OpenSearch 3.9.0 | [GitHub opensearch-project/OpenSearch — releases](https://api.github.com/repos/opensearch-project/OpenSearch/releases?per_page=4) | 2026-09-29 | haute |
| [22] | MySQL : image, variables, base Oracle Linux 9 | [Docker Hub — mysql](https://hub.docker.com/_/mysql) et [README docker-library](https://github.com/docker-library/docs/blob/master/mysql/README.md) | page au 2026-10-04 | haute |
| [23] | MySQL 26.7.0 | [MySQL — notes de version](https://dev.mysql.com/doc/relnotes/mysql/26.7/en/news-26-7-0.html) | 2026-07-28 | moyenne (conflit « Early Access » / GA non résolu) |
| [24] | pgAdmin : variables, ports, volume, 9.18 | [pgAdmin — conteneur](https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html) et [notes 9.18](https://www.pgadmin.org/docs/pgadmin4/latest/release_notes_9_18.html) | 2026-09-17 | haute |
| [25] | Image pgAdmin | [Docker Hub — dpage/pgadmin4](https://hub.docker.com/r/dpage/pgadmin4) | page au 2026-10-04 | haute |

## Carte de péremption

Calculée avec les fenêtres du pack technique (versions et compatibilité : 1 mois).

| Affirmation | Classe | À revérifier le |
|---|---|---|
| Gitea `latest` = 28.0 | version | 2026-10-30 |
| Grafana admin/admin et invite de changement | compatibilité | 2026-11-04 |
| SeaweedFS : commande par défaut `mini` | version | 2026-11-04 |
| OpenSearch : `vm.max_map_count` | compatibilité | 2026-11-04 |
| Meilisearch v1.54.3 | version | 2026-11-01 |
| Prometheus : auto-collecte | compatibilité | 2026-11-04 |
| Garage v2.3.0 | version | **périmé** (2026-05-16 dépassé) |
| Keycloak 26.8.0 | version | 2026-11-01 |
| SeaweedFS 4.48 | version | 2026-10-28 |
| MySQL lts 9.7.2 | version | 2026-10-29 |

**Première échéance :** Garage est déjà périmé ; ensuite SeaweedFS 4.48 le 2026-10-28. Les rapports sur la sélection de services vieillissent vite : revérifier les tags la veille de l'exposé. Un rapport de sélection de plus de deux trimestres doit être rafraîchi avant toute décision.
