---
title: 'technical research: WSLC (WSL Containers)'
type: 'technical'
topic: 'WSLC (WSL Containers)'
decision: 'Rédiger un exposé et un document Markdown avec des commandes WSLC vérifiées (PostgreSQL, Redis, Solr, mail factice, etc.)'
source: 'native run'
status: complete
preset: 'standard'
validation: 'normal'
created: '2026-10-04'
updated: '2026-10-04'
claims_verified: 3
claims_unverified: 7
---

# Recherche technique : WSLC (WSL Containers)

**Décision servie :** écrire un exposé et un document Markdown de commandes WSLC vérifiées (PostgreSQL, Redis, Solr, mail factice et autres services).

**Limite de méthode :** tout provient de documentation lue en ligne. Aucune commande n'a été exécutée, car cet environnement est Linux et pas Windows. Les pages GitHub et plusieurs blogs ont été lus via un outil qui les résume : les commandes issues de ces pages sont « telles qu'extraites ».

## Synthèse

- **WSLC n'est plus en préversion.** Disponibilité générale le 2026-09-29 avec WSL 3.0.1 [3][4]. Pour l'installer : `wsl --update`, WSL 2.9.3 ou plus [1]. Le `--pre-release` de l'annonce de juin est périmé. Confiance haute (vérifié par plusieurs sources).
- **Compose n'existe pas** : Microsoft le désigne comme priorité des prochaines itérations [3]. Pour l'exposé, il faut tout montrer avec des `wslc run` séparés et un réseau explicite, jamais avec un fichier `compose.yaml`.
- **Les commandes de base sont sûres**, car publiées dans la doc officielle au 2026-09-29 [1][2] : `run`, `exec`, `container list|stop|logs|inspect|prune`, `image list|inspect|prune`, `build`, `volume create`, `--help`.
- **Trois zones à tester sur la machine avant l'exposé** : le DNS par nom de conteneur entre services, les options `--restart` / `--memory` / `--cpus`, et les chemins de données persistantes. Aucune source primaire ne les documente de façon complète.
- **Pièges de services à connaître** : PostgreSQL 18 a changé son volume (`/var/lib/postgresql`) [10] ; MinIO est archivé et inutilisable en démo [21] ; MailHog est abandonné au profit de Mailpit [15][22].

## 1. Paysage et maturité

| Point | Constat | Source | Confiance |
|---|---|---|---|
| Nature | Deux parties : la CLI intégrée `wslc.exe` (alias `container.exe`) et une API de conteneurs pour les applications Windows | [2][3] | haute |
| Statut | GA le 2026-09-29, WSL 3.0.1, « Latest » sur GitHub. Préversion publique le 2026-06-29 (WSL 2.9.3) | [3][4][5][24] | haute |
| Prérequis | WSL 2.9.3 ou plus ; `wsl --version` pour vérifier. Aucun prérequis de build Windows trouvé | [1] | haute (WSL) ; build non trouvé |
| Architecture | Une VM dédiée par session, processus `wslcsession.exe` par utilisateur, partage de fichiers Windows par virtiofs, réseau « consomme » en processus utilisateur | [6] | moyenne (page résumée) |
| Noyau | Les conteneurs tournent sur le noyau Linux de WSL 2 | [1] | haute |
| Runtime sous-jacent | **Non établi** (containerd ou custom) : ne rien affirmer dans l'exposé | [6] | — |
| Cadence | Versions toutes les 1 à 2 semaines pendant l'été : 2.9.8 (24 août) à 3.0.1 (29 septembre) | [4] | moyenne-haute |
| Entreprise | Contrôles Intune, liste d'autorisation de registres, surveillance Defender for Endpoint | [3][5] | haute |
| Comparaison Docker Desktop / Podman | **Aucune comparaison officielle ni testée trouvée.** Seul un billet d'opinion d'avant lancement | [25] | basse |

L'API développeurs passe par le paquet NuGet `Microsoft.WSL.Containers` (`dotnet add package Microsoft.WSL.Containers`) ; la projection C++/WinRT est en préversion [2].

## 2. Référence CLI

**Commandes confirmées dans la doc Learn (2026-09-29)** [1][2], à citer telles quelles :

```powershell
wsl --update
wslc version
wslc run --rm hello-world
wslc run --rm -it ubuntu:latest bash -c "echo Hello world from WSL container!"
wslc run -d --rm -p 8080:80 --name web nginx
curl localhost:8080
wslc container list              # ou: wslc container ps
wslc container list --all
wslc image list                  # ou: wslc image ls
wslc exec web cat /etc/os-release
wslc stats
wslc container logs <nom>
wslc container inspect <nom>
wslc container stop web
wslc build -t helloworld-django .   # fichier: Containerfile
wslc container prune
wslc image prune
```

- `ls`, `ps` et `list` apparaissent tous dans la doc officielle [1][2].
- Un billet de préversion montre des formes courtes (`wslc pull`, `wslc logs`, `wslc stop`, `wslc rm`) [23] : confiance basse, non confirmées au GA ; préférer les formes complètes de la doc officielle.
- **Non vérifié sur 3.0.1** : `wslc --version` (blog de juin) contre `wslc version` (Learn). Utiliser `wslc version`.
- Arbre complet des sous-commandes, relevé sur une issue tierce de l'ère préversion [8] (confiance moyenne, périmé) : `container` (run, create, start, stop, kill, rm, exec, attach, logs, list, inspect, stats, prune, export), `image` (build, pull, push, load, save, import, tag, list, inspect, rm, prune), `network` et `volume` (create, list, inspect, rm, prune), `session`, `registry login|logout`, `settings`, `system`.
- Nouveautés de la 3.0.x [3][4] : `wslc events`, `wslc container restart`, `wslc container cp`, `wslc system info`, `wslc network connect|disconnect`, `--mount` sur `run` et `create`, healthchecks, `--stop-timeout`.
- **Non trouvé** : une page de référence CLI officielle listant toutes les options, et une sortie brute de `wslc --help`. Capturer la vraie aide sur la machine de l'exposé.

## 3. Intégration et réseau

- **Ports** : `-p hôte:conteneur`, accessible en `localhost` depuis PowerShell et depuis un navigateur Windows [1]. UDP et IPv6 ajoutés en 3.0.x [4].
- **Dossier Windows monté** (blog d'architecture) [6], syntaxe issue d'un résumé de page :
  ```powershell
  wslc container run -v C:\Windows\System32\drivers\etc:/volume -it debian:latest ls /volume
  ```
- **Volume nommé** (disque VHD, système de fichiers Linux natif) [6] :
  ```powershell
  wslc volume create --driver vhd -o SizeBytes=200000000 my-volume
  wslc container run -v my-volume:/volume -it debian:latest findmnt /volume
  ```
- **Performances** : la doc Learn recommande de garder le code sur le système de fichiers Linux ; les fichiers Windows ralentissent « significativement » les outils Linux [1].
- **Variables d'environnement** : `-e CLE=VALEUR` apparaît dans un exemple de préversion [5] ; `--env-file` est mentionné par un outil tiers, non vérifié.
- **Réseaux** : `wslc network create|list|inspect|rm|prune` et `connect|disconnect` existent [3][8]. **Aucun exemple documenté de DNS par nom de conteneur** : c'est le risque principal pour une démo multi-services. Ne pas supposer que `postgres` se résout depuis un autre conteneur sans l'avoir testé.
- **Politique de redémarrage** : `--restart` était absent en préversion [8][9] ; 3.0.1 n'ajoute que la commande ponctuelle `wslc container restart` [3]. Non revérifié sur 3.0.1.
- **Limites de ressources** : `--memory`, `--cpus` et `--ulimit` sont acceptés ; `--privileged`, `--cap-add`, `--read-only` et `--security-opt` sont refusés (version 2.9.13, source tierce partielle, confiance basse).

## 4. Recettes de services

Commandes de la doc de chaque service, avec `docker` remplacé par `wslc`. **Aucune n'a été testée sous WSLC.**

| Service | Image et tag | Commande de départ | Vérification | Source |
|---|---|---|---|---|
| PostgreSQL | `postgres:18` (18.6 au 2026-10-04) | `wslc run -d --name pg -e POSTGRES_PASSWORD=secret -p 5432:5432 -v pgdata:/var/lib/postgresql postgres:18` | `wslc exec pg psql -U postgres -c "select version();"` | [10] |
| Redis | `redis:8` (8.10.2) | `wslc run -d --name redis -p 6379:6379 -v redisdata:/data redis:8` | `wslc exec redis redis-cli ping` (attendu : PONG) | [11] |
| Valkey (alternative à Redis) | `valkey/valkey:9` (pas d'image officielle `valkey`) | `wslc run -d --name valkey -p 6379:6379 valkey/valkey:9` | `wslc exec valkey valkey-cli ping` | [12] |
| Solr | `solr:10.0.0` ou `solr:9.10.1` | `wslc run -d --name solr -p 8983:8983 -v solrdata:/var/solr solr solr-precreate gettingstarted` | http://localhost:8983/ | [13][14] |
| Mailpit (mail factice) | `axllent/mailpit` (v1.31.4 au 2026-10-03) | `wslc run -d --name mailpit -p 8025:8025 -p 1025:1025 axllent/mailpit` | interface http://localhost:8025 ; SMTP sur le port 1025 | [15][16] |
| Adminer | `adminer` | `wslc run -d --name adminer -p 8081:8080 adminer` | http://localhost:8081 | [17] |
| RabbitMQ | `rabbitmq:4-management` | `wslc run -d --name rabbit -p 5672:5672 -p 15672:15672 rabbitmq:4-management` | http://localhost:15672 (guest/guest, à tester) | [18] |
| MongoDB | `mongo` | `wslc run -d --name mongo -p 27017:27017 -v mongodata:/data/db mongo` | `wslc exec mongo mongosh --eval "db.runCommand({ping:1})"` | [19] |
| MariaDB | `mariadb:lts` | `wslc run -d --name maria -e MARIADB_ROOT_PASSWORD=secret -p 3306:3306 mariadb:lts` | `wslc exec maria mariadb -uroot -psecret -e "select 1"` | [20] |

**Remarques par service** (les commandes du tableau sont assemblées à partir des pages de documentation, volumes nommés inclus, mais aucune n'a été exécutée) :

- **PostgreSQL 18** : le volume déclaré est `/var/lib/postgresql` et `PGDATA` vaut `/var/lib/postgresql/18/docker`. Pour la version 17 et avant, monter `/var/lib/postgresql/data`, et pas `/var/lib/postgresql` [10]. Le port 5432 et les commandes de vérification sont standards mais ne figurent pas sur la page lue.
- **Redis** : la licence est passée à RSALv2 / SSPLv1 / AGPLv3 à partir de la 8.0 [11] ; Valkey est l'alternative libre, sous un nom d'image différent [12]. `--appendonly yes` pour la persistance AOF n'est pas sur la page lue.
- **Solr** : la commande `solr-precreate gettingstarted` vient du guide officiel. Le guide utilise un dossier lié `$PWD/solrdata` ; un volume nommé est plus sûr sous WSL (droits de l'utilisateur 8983 non documentés, à tester) [13].
- **Mailpit** : le test d'envoi du README demande un client `mail` ; l'alternative `curl --url smtp://localhost:1025 --mail-from a@x.test --mail-rcpt b@x.test -T mail.txt` n'est pas dans les pages lues. MailHog n'est plus maintenu selon des sources secondaires (confiance moyenne) [22].
- **RabbitMQ** : les variables `RABBITMQ_DEFAULT_USER` / `RABBITMQ_DEFAULT_PASS` : statut de dépréciation ambigu sur 4.x, à vérifier avant de les mettre dans le doc.
- **MongoDB** : éviter les montages depuis `/mnt/c` (la doc recommande des volumes nommés sous Windows) [19].
- **MinIO : à ne pas utiliser.** Le dépôt GitHub est archivé depuis le 2026-04-25 (« no longer maintained »), les éditions communautaires ne sont distribuées qu'en code source [21]. Alternatives non vérifiées : Garage, SeaweedFS, LocalStack.
- **Non recherchés** : Nginx (hors tutoriel Learn, qui l'utilise), Elasticsearch, OpenSearch, Meilisearch, MySQL.

## 5. Mise en œuvre et limites au 2026-10-04

- **Compose absent** [3], et Visual Studio affiche « Docker Compose is not supported with the WSL container runtime (wslc) » (issue #41754, ouverte) [7].
- **Issues WSLC ouvertes sur 3.0.1** (178 résultats, première page lue seulement ; liste résumée) [7] : `--gpus all` plante sur Alpine/musl (#41791) ; `host.wslc.internal` non résolu en musl (#41769) ; `wslc container cp` échoue sur les liens symboliques (#41793) ; `wslc run` renvoie ERROR_TIMEOUT (#41740) ; héritage des variables proxy et miroirs de registre absents (#41794).
- **Conséquence pour l'exposé** : préférer des images Debian ou Ubuntu aux variantes Alpine pour les démos, tant que ces issues sont ouvertes.
- **Pas de passage de périphérique USB** en préversion (périmé, non revérifié) [5].
- **Santé du projet** : cadence très active, ordre de grandeur d'une version par semaine ou deux, avec un objectif de parité avec Docker [4]. Un outil tiers (`docker2wslc`) traduit déjà les commandes Docker vers WSLC [9], ce qui indique une adoption communautaire naissante (indice faible, pas une mesure).

## Informations croisées entre dimensions

- La CLI est complète pour des démos unitaires, mais **l'absence de compose et d'exemple de DNS par nom** rend fragile toute démo « stack complète » (PostgreSQL + Adminer, ou Solr + application). Le chemin sûr : publier chaque service sur un port de `localhost` et se connecter depuis Windows, plutôt que de laisser les conteneurs se parler.
- Les pièges de services (volume de PostgreSQL 18, MinIO, MailHog) viennent des projets amont, pas de WSLC : ils valent un encadré « pièges » dans le document, indépendamment de l'outil.

## Preuves contraires

La passe red team n'a pas été lancée (réglage `off`). Un seul point contredit un digest : la date d'archivage de MinIO, corrigée de février à avril 2026 après relecture de la page GitHub [21].

## Recommandations

1. **Installer et tester d'abord** : sur la machine de l'exposé, `wsl --update`, puis `wslc version` et `wslc run --rm hello-world` [1]. Confiance haute.
2. **Afficher la version de WSL en début d'exposé** : les informations de juin sont périmées en quelques semaines. Confiance haute.
3. **Écrire le document avec des blocs de commandes marqués « testé » ou « à tester »** : tout est « à tester » tant qu'aucune exécution réelle n'a eu lieu. Confiance haute sur ce principe ; les commandes de services reposent sur des sources officielles mais non exécutées.
4. **Capturer `wslc --help` et `wslc run --help`** sur 3.0.1 pour figer les options réelles (`--restart`, `--memory`, `--cpus`, `--env-file`, `--mount`). Confiance haute que c'est nécessaire.
5. **Tester le DNS par nom** sur un `wslc network create`, avec deux conteneurs, avant d'annoncer une démo où un service en appelle un autre.
6. **Éviter MinIO** ; si une démo S3 est voulue, choisir et tester une alternative. Éviter MailHog au profit de Mailpit. Confiance haute (MinIO), moyenne (MailHog).
7. **Liaison avec le projet** : ces résultats alimentent directement le plan de l'exposé (`bmad-spec`) : prérequis, liste de démos, limites à annoncer.

## Questions ouvertes

| Question | Pour y répondre |
|---|---|
| Runtime sous-jacent (containerd, runc ou custom) ? | Lire `src/windows/wslc` dans le dépôt microsoft/WSL |
| DNS par nom de conteneur sur un réseau utilisateur ? | Test réel sur 3.0.1 |
| Options exactes de `wslc run` (`--restart`, `--env-file`, valeurs de `--memory`) ? | `wslc run --help` sur 3.0.1 |
| Prérequis Windows (build, virtualisation) ? | https://learn.microsoft.com/windows/wsl/install |
| Les issues #41791, #41769, #41793 sont-elles corrigées dans une version plus récente ? | Relire la page des versions la veille de l'exposé |
| Version exacte de WSL 3.0.1 vs ligne 2.9.x : quel est le canal stable ? | Notes de version complètes |
| Nginx, Elasticsearch, OpenSearch, Meilisearch, MySQL | Nouvelle recherche ciblée (Deepen) |

## Annexe des sources

Date d'accès de toutes les sources : 2026-10-04.

| [n] | Constat soutenu | Éditeur | Date de publication | Confiance |
|---|---|---|---|---|
| [1] | Installation, commandes de base, build, ports, inspect, prune | [Microsoft Learn — tutoriel](https://learn.microsoft.com/windows/wsl/tutorials/wsl-containers) | 2026-09-29 | haute (relu par le lead) |
| [2] | Présentation, API, CLI, `container.exe` | [Microsoft Learn — wsl container](https://learn.microsoft.com/windows/wsl/wsl-container) | 2026-09-29 | haute |
| [3] | GA, compose absent, nouvelles commandes | [Windows Developer Blog](https://blogs.windows.com/windowsdeveloper/2026/09/29/wsl-containers-now-generally-available/) | 2026-09-29 | haute (relu par le lead) |
| [4] | WSL 3.0.1 « Latest », cadence des versions | [GitHub microsoft/WSL — releases](https://github.com/microsoft/WSL/releases) | 2026-09-29 | moyenne-haute (page résumée) |
| [5] | Préversion de juin, exemples, limites de l'époque | [Windows Command Line blog](https://devblogs.microsoft.com/commandline/wsl-container-is-now-available-for-public-preview/) | 2026-06-29 | moyenne (périmé) |
| [6] | Architecture, virtiofs, volumes VHD, montage Windows | [Windows Command Line blog](https://devblogs.microsoft.com/commandline/wslc-architecture-deep-dive/) | 2026-09-29 | moyenne-haute (page résumée) |
| [7] | Issues ouvertes (#41754, #41791 et autres) | [GitHub microsoft/WSL — issues](https://github.com/microsoft/WSL/issues?q=is%3Aissue+wslc+sort%3Aupdated-desc) | 2026-10-01 à 2026-10-04 | moyenne |
| [8] | Arbre de commandes, absence de `--restart` en préversion | [GitHub kubernetes-sigs/kind #4208](https://github.com/kubernetes-sigs/kind/issues/4208) | période préversion | moyenne (tiers, périmé) |
| [9] | Options Docker traduites ou non (restart, buildx, compose) | [PyPI docker2wslc 0.2.0](https://pypi.org/project/docker2wslc/) | 2026-07-29 | moyenne (tiers, préversion) |
| [10] | Image postgres, tags, chemin de données de la 18 | [Docker Hub — postgres](https://hub.docker.com/_/postgres) | page au 2026-10-04 | haute (relu par le lead) |
| [11] | Image redis, tags, licence | [Docker Hub — redis](https://hub.docker.com/_/redis) | page au 2026-10 | haute |
| [12] | Image valkey/valkey | [Docker Hub — valkey/valkey](https://hub.docker.com/r/valkey/valkey) | page au 2026-10 | haute |
| [13] | `solr-precreate`, volume `/var/solr` | [Apache Solr — guide Docker](https://solr.apache.org/guide/solr/latest/deployment-guide/solr-in-docker.html) | page « latest » (10.0) | haute |
| [14] | Image solr, tags | [Docker Hub — solr](https://hub.docker.com/_/solr) | page au 2026-10 | haute |
| [15] | Image mailpit, ports 1025/8025 | [Docker Hub — axllent/mailpit](https://hub.docker.com/r/axllent/mailpit) et [GitHub README](https://github.com/axllent/mailpit) | page au 2026-10 | haute |
| [16] | Mailpit v1.31.4 | [GitHub — Mailpit releases](https://github.com/axllent/mailpit/releases/latest) | 2026-10-03 | moyenne (source unique) |
| [17] | Adminer | [Docker Hub — adminer](https://hub.docker.com/_/adminer) | page au 2026-10 | haute |
| [18] | RabbitMQ `4-management`, ports | [RabbitMQ — download](https://www.rabbitmq.com/docs/download) et [Docker Hub](https://hub.docker.com/_/rabbitmq) | page au 2026-10 | haute (commande) ; basse (variables) |
| [19] | MongoDB, volumes nommés | [Docker Hub — mongo](https://hub.docker.com/_/mongo) | page au 2026-10 | haute |
| [20] | MariaDB, variables racine | [Docker Hub — mariadb](https://hub.docker.com/_/mariadb) | page au 2026-10 | haute |
| [21] | MinIO archivé le 2026-04-25 | [GitHub minio/minio](https://github.com/minio/minio) (et blogs tiers) | 2026-04-25 | haute (relu par le lead) |
| [22] | MailHog non maintenu | [GitHub MailHog](https://github.com/mailhog/MailHog) et recherches secondaires | non daté | moyenne |
| [23] | Formes courtes de commandes en préversion | [boxofcables.dev](https://boxofcables.dev/wslc-a-native-linux-container-runtime-for-windows) | 2026-06-02 | basse (périmé) |
| [24] | GA confirmée par une source secondaire | [It's FOSS](https://itsfoss.com/news/wslc-general-availability/) | 2026-09-30 | moyenne |
| [25] | Billet d'opinion d'avant lancement | [falcao.org](http://falcao.org/posts/wsl-containers-what-changes-for-wsl2-users/) | 2026-06-20 | basse (périmé) |

## Carte de péremption

Calculée avec les fenêtres de fraîcheur du pack technique (versions et compatibilité : 1 mois ; écosystème : 6 mois ; paysage : 12 mois).

| Affirmation | Classe | À revérifier le |
|---|---|---|
| WSLC GA avec WSL 3.0.1 | version | 2026-10-29 |
| WSL ≥ 2.9.3, `wsl --update` | version | 2026-10-29 |
| Compose non supporté | paysage | 2027-09-29 |
| Chemin de données de Postgres 18 | version | 2026-11-04 |
| MinIO archivé | écosystème | 2026-10-25 |
| Mailpit v1.31.4 | version | 2026-11-03 |
| Architecture VM par session | paysage | 2027-09-29 |
| Pas de `--restart` (préversion) | compatibilité | **périmé** : à revérifier (2026-08-29 dépassé) |
| `--gpus all` en échec sur Alpine | compatibilité | 2026-11-03 |
| Redis 8.10.2 et licence | version | 2026-11-04 |
| Solr 10.0.0 | version | 2026-11-04 |

**Première échéance :** l'absence de `--restart` est déjà périmée ; ensuite la GA et les prérequis le 2026-10-29. Un Refresh la veille de l'exposé est recommandé.
