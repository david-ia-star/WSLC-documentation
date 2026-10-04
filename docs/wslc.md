# WSLC : lancer des conteneurs Linux depuis Windows

Version de référence : WSL 3.0.1 (disponibilité générale du 29 septembre 2026). Document rédigé le 4 octobre 2026.

Chaque bloc de commandes porte l'étiquette **testé** ou **à tester**. À la date de rédaction, aucune commande n'a été exécutée sous Windows : tout est à tester. Les commandes viennent des documentations officielles de Microsoft et de chaque projet. Les syntaxes sont en PowerShell, avec des volumes nommés plutôt que des dossiers Windows.

## 1. Ce qu'est WSLC

WSLC (WSL Containers) est un runtime de conteneurs Linux livré avec WSL. Il comprend une CLI, `wslc.exe` (alias `container.exe`), et une API pour les applications Windows. Aucun moteur séparé n'est à installer : `wslc.exe` arrive avec WSL.

Microsoft l'a ouvert en préversion publique le 29 juin 2026 (WSL 2.9.3) et l'a déclaré stable le 29 septembre 2026 avec WSL 3.0.1. Toute instruction qui mentionne `wsl --update --pre-release` date de la préversion.

Chaque session WSLC tourne dans sa propre machine virtuelle. Les dossiers Windows sont partagés par virtiofs, les volumes nommés sont des disques VHD au format Linux natif, et les conteneurs utilisent le noyau Linux de WSL 2.

## 2. Installer et vérifier

WSL 2.9.3 ou plus est requis. Les prérequis Windows (numéro de build, virtualisation) ne figurent dans aucune page lue : vérifiez-les sur la page d'installation de WSL.

**Statut : à tester.**

```powershell
wsl --update
wsl --version
wslc version
wslc run --rm hello-world
```

`wslc run --rm hello-world` télécharge l'image si elle manque, puis affiche un message de bienvenue.

## 3. De Docker à `wslc`

Les options de `docker run` les plus courantes (`-d`, `-p`, `-e`, `-v`, `--name`, `--rm`, `-it`) existent dans `wslc run`. Le tableau donne les écarts de syntaxe.

| Docker | `wslc` | Remarque |
|---|---|---|
| `docker run` | `wslc run` | Mêmes options courantes |
| `docker exec` | `wslc exec <nom> <commande>` | |
| `docker logs` | `wslc container logs <nom>` | |
| `docker ps` | `wslc container list` (ou `ps`) | `--all` inclut les conteneurs arrêtés |
| `docker images` | `wslc image list` (ou `ls`) | |
| `docker stop` | `wslc container stop <nom>` | |
| `docker inspect` | `wslc container inspect <nom>`, `wslc image inspect <image>` | |
| `docker build` | `wslc build -t <nom> .` | Fichier par défaut : `Containerfile`. Le nom `Dockerfile` et l'option `-f` ne sont pas vérifiés |
| `docker system prune` | `wslc container prune`, `wslc image prune` | |
| `docker stats` | `wslc stats` | |
| `docker compose` | **absent** | Voir la section 6 |
| `--restart` | **non confirmé** | Voir la section 6 |

Pour la liste complète des commandes : `wslc --help` et `wslc <COMMANDE> --help`. Aucune page de référence officielle ne liste toutes les options.

## 4. Un premier conteneur

Un serveur web publié sur le port 8080 de Windows. Les commandes sont celles du tutoriel Microsoft du 29 septembre 2026.

**Statut : à tester.**

```powershell
wslc run -d --rm -p 8080:80 --name web nginx
curl.exe localhost:8080
wslc container list
wslc exec web cat /etc/os-release
wslc container stop web
```

PowerShell définit `curl` comme alias de `Invoke-WebRequest` : `curl.exe` appelle le vrai programme. Comme `--rm` est actif, le conteneur disparaît dès l'arrêt.

## 5. Dix services en une commande

Chaque service se lance seul avec un `wslc run`. Compose n'existe pas, et la résolution de noms entre conteneurs n'est pas confirmée : aucune démo ne relie deux conteneurs. Les versions datent du 4 octobre 2026 ; revérifiez-les la veille.

Trois services sont prévus en direct : Mailpit, Grafana et Gitea. Ils démarrent vite, ont une interface web et n'exigent aucun réglage de la machine. Les sept autres sont à lire. Le choix définitif se fait après les tests.

### Mailpit : serveur de mail factice

**Statut : à tester.**

```powershell
wslc run -d --name mailpit -p 8025:8025 -p 1025:1025 axllent/mailpit
```

- Interface : http://localhost:8025. Le serveur SMTP écoute sur le port 1025.
- Test d'envoi : `curl.exe --url smtp://localhost:1025 --mail-from a@x.test --mail-rcpt b@x.test -T mail.txt`, avec un fichier `mail.txt` qui contient un message. Cette commande ne figure pas dans les pages lues.
- Version v1.31.4 (3 octobre 2026). Le format du tag Docker n'est pas vérifié : épinglez la version après contrôle.
- MailHog propose les mêmes ports, mais n'est plus maintenu.

### Grafana : tableaux de bord

**Statut : à tester.**

```powershell
wslc run -d --name grafana -p 3000:3000 -v grafana-data:/var/lib/grafana grafana/grafana:13.2.3
```

- Interface : http://localhost:3000. Connexion `admin` / `admin`, puis Grafana demande de changer le mot de passe.
- La version 13.2.3 (29 septembre 2026) corrige trois failles de sécurité.
- L'image par défaut repose sur Alpine ; la variante `13.2.3-ubuntu` existe. Licence AGPL-3.0.
- Seul, Grafana montre ses tableaux de bord et la source de test intégrée. Pour de vraies métriques, il faut lui connecter Prometheus, ce qui demande un second conteneur.

### Gitea : forge Git

**Statut : à tester.**

```powershell
wslc run -d --name gitea -p 3001:3000 -p 2222:22 -v gitea-data:/data gitea/gitea:28.0.0
```

- Interface : http://localhost:3001. L'assistant d'installation s'affiche à la première visite, avec SQLite par défaut : un seul conteneur suffit.
- Le port 3001 évite le conflit avec Grafana. La correspondance du port SSH 22 vers 2222 est une déduction, pas un texte de la documentation.
- `latest` pointe sur la version 28.0 (30 septembre 2026). Les images avec et sans root (`-rootless`) sont incompatibles : ne mélangez pas leurs données.

### PostgreSQL

**Statut : à tester.**

```powershell
wslc run -d --name pg -e POSTGRES_PASSWORD=secret -p 5432:5432 -v pgdata:/var/lib/postgresql postgres:18.6
wslc exec pg psql -U postgres -c "select version();"
```

- À partir de la version 18, le volume déclaré est `/var/lib/postgresql` et `PGDATA` vaut `/var/lib/postgresql/18/docker`. Pour la 17 et avant, montez `/var/lib/postgresql/data`, pas `/var/lib/postgresql`.
- Le port 5432 et la commande `psql` sont standards mais ne figurent pas sur la page Docker Hub lue.

### Redis

**Statut : à tester.**

```powershell
wslc run -d --name redis -p 6379:6379 -v redisdata:/data redis:8.10.2
wslc exec redis redis-cli ping
```

- Réponse attendue : `PONG`.
- Depuis la version 8.0, Redis est publié sous triple licence (RSALv2, SSPLv1, AGPLv3). L'alternative libre est `valkey/valkey:9`, qui n'a pas d'image officielle Docker sous le nom `valkey`.
- La persistance par instantanés s'active avec `redis-server --save 60 1`. L'option `--appendonly yes` pour le journal AOF n'est pas sur la page lue.

### Solr

**Statut : à tester.**

```powershell
wslc run -d --name solr -p 8983:8983 -v solrdata:/var/solr solr:10.0.0 solr-precreate gettingstarted
```

- Interface : http://localhost:8983/. La commande `solr-precreate` crée le cœur `gettingstarted` au démarrage.
- Le guide officiel monte un dossier de l'hôte (`$PWD/solrdata`). Un volume nommé est plus sûr ici, car les droits de l'utilisateur 8983 ne sont pas documentés. À tester.
- La ligne 9.x existe : `solr:9.10.1`.

### SonarQube : analyse de qualité de code

**Statut : à tester. Ne pas lancer en direct sans test préalable.**

```powershell
wslc run -d --name sonarqube -p 9000:9000 sonarqube:26.9.0.129388-community
```

- Interface : http://localhost:9000, connexion `admin` / `admin`.
- SonarQube embarque Elasticsearch, qui exige côté hôte `vm.max_map_count=524288`, `fs.file-max=131072`, `ulimit -n 131072` et `ulimit -u 8192`. Comment régler `vm.max_map_count` dans la machine virtuelle d'une session WSLC n'est documenté nulle part dans les sources lues. La démo peut échouer au démarrage.
- La base H2 embarquée convient à une démonstration, pas à de la production.

### Meilisearch : moteur de recherche

**Statut : à tester.**

```powershell
wslc run -d --name meili -p 7700:7700 -e MEILI_ENV=development -v meili-data:/meili_data getmeili/meilisearch:v1.54.3
curl.exe http://localhost:7700/health
```

- Interface de prévisualisation : http://localhost:7700, active en mode développement.
- La documentation de Meilisearch cite encore le tag `v1.37`, périmé. Les tags `v1` et `v1.52` paraissent en retard : épinglez `v1.54.3` (1er octobre 2026).
- En mode production, une clé maîtresse (`MEILI_MASTER_KEY`, 16 octets minimum) devient obligatoire.

### Prometheus : métriques

**Statut : à tester.**

```powershell
wslc run -d --name prometheus -p 9090:9090 prom/prometheus:v3.15.0
```

- Interface : http://localhost:9090. Dans Status, puis Targets, vérifiez que Prometheus se surveille lui-même, puis lancez la requête `up`.
- Le fichier d'exemple du dépôt Prometheus collecte `localhost:9090`. Que l'image l'utilise par défaut n'est pas confirmé : la page Targets le montrera.

### SeaweedFS : stockage compatible S3

SeaweedFS remplace MinIO, dont le dépôt est archivé depuis le 25 avril 2026.

**Statut : à tester.**

```powershell
wslc run -d --name seaweed -p 8333:8333 -p 8888:8888 -p 23646:23646 -v weed-data:/data -e AWS_ACCESS_KEY_ID=admin -e AWS_SECRET_ACCESS_KEY=secret -e S3_BUCKET=my-bucket chrislusf/seaweedfs:4.48
```

- Interface d'administration : http://localhost:23646. Interface des fichiers : http://localhost:8888. API S3 : port 8333.
- L'image démarre par défaut en mode tout-en-un (`mini -dir=/data`) et crée le seau `my-bucket`.
- Test avec le client AWS : `aws s3 ls --endpoint-url http://localhost:8333`. Cette ligne ne vient d'aucune source : à confirmer.
- Sans identifiants, SeaweedFS passe en mode « Allow All » et n'authentifie plus personne. Licence Apache 2.0.

## 6. Ce qui ne marche pas encore

Relevé au 4 octobre 2026, sur WSL 3.0.1.

| Limite | Détail | Statut |
|---|---|---|
| Compose absent | Microsoft en fait la première demande des utilisateurs et la priorité des prochaines versions. Visual Studio affiche : « Docker Compose is not supported with the WSL container runtime (wslc) » (issue #41754) | Vérifié par deux sources |
| DNS entre conteneurs | Aucun exemple documenté de résolution par nom sur un réseau créé avec `wslc network create`. Les commandes `network connect` et `disconnect` existent | À tester |
| Politique de redémarrage | `--restart` était absent en préversion. La version 3.0.1 ajoute seulement la commande ponctuelle `wslc container restart` | Non revérifié sur 3.0.1 |
| Options refusées | `--privileged`, `--cap-add`, `--cap-drop`, `--security-opt`, `--read-only`, `--pids-limit` rejetées en 2.9.13. `--memory`, `--cpus` et `--ulimit` acceptées | Source tierce, non retestée sur 3.0.1 |
| GPU sous Alpine | `--gpus all` fait échouer la création d'un conteneur Alpine (issue #41791). Les images Debian et Ubuntu fonctionnent | Ouvert |
| USB | Passage de périphérique non supporté en préversion | Non revérifié |
| Adresse de l'hôte | `host.wslc.internal` ne se résout pas dans les conteneurs Alpine (issue #41769) | Ouvert |
| Autres issues ouvertes | `wslc container cp` échoue sur les liens symboliques (#41793), `wslc run` renvoie parfois ERROR_TIMEOUT (#41740), les variables proxy de l'hôte ne sont pas reprises (#41794) | Ouvert |
| Fichiers Windows | Les fichiers du disque Windows ralentissent nettement les outils Linux : gardez le code sur le système de fichiers Linux. Virtiofs va plus vite que Plan9 sans égaler le natif | Documenté par Microsoft |
| Réglages hôte | SonarQube et OpenSearch exigent un `vm.max_map_count` élevé, dont le réglage sous WSLC est inconnu | Non résolu |
| Runtime sous-jacent | Containerd ou runtime propre à Microsoft : aucune source ne le dit | Inconnu |
| Comparaison avec Docker Desktop ou Podman | Aucune comparaison officielle ni testée n'existe | Absente |

## 7. Plan de ports

| Port | Service |
|---|---|
| 3000 | Grafana |
| 3001 | Gitea (le 3000 est pris par Grafana) |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 7700 | Meilisearch |
| 8025, 1025 | Mailpit (interface, SMTP) |
| 8333, 8888, 23646 | SeaweedFS (S3, fichiers, administration) |
| 8983 | Solr |
| 9000 | SonarQube |
| 9090 | Prometheus |

Aucun conflit tant que Gitea reste sur 3001. Les services de l'annexe tiennent aussi dans ce plan : Keycloak 8080, Adminer 8081, pgAdmin 8082, RabbitMQ 5672 et 15672, RustFS 9100 et 9001, MongoDB 27017, MariaDB 3306.

## 8. Avant l'exposé

À cocher sur la machine qui servira à la démonstration. Chaque test réussi fait passer l'étiquette du bloc de **à tester** à **testé**.

- [ ] `wsl --version` affiche 2.9.3 ou plus, `wslc version` répond.
- [ ] `wslc run --rm hello-world` et la démo nginx passent.
- [ ] Les trois services en direct démarrent, répondent comme indiqué et tiennent dans la mémoire de la machine virtuelle.
- [ ] Deux conteneurs sur un réseau créé avec `wslc network create` se joignent par leur nom.
- [ ] `wslc run --help` montre ce que deviennent `--restart`, `--env-file` et `--memory`.
- [ ] `vm.max_map_count` se règle dans la machine virtuelle WSLC et SonarQube démarre.
- [ ] Prometheus affiche sa cible dans Status, puis Targets.
- [ ] La commande `aws s3 ls` répond contre SeaweedFS.
- [ ] Le tag Docker de Mailpit est épinglé.
- [ ] La veille : tous les tags sont revérifiés, les étiquettes mises à jour.

Déroulé proposé pour 15 à 20 minutes :

| Minutes | Contenu |
|---|---|
| 2 | Ce qu'est WSLC, installation (sections 1 et 2) |
| 2 | De Docker à `wslc` et le premier conteneur (sections 3 et 4) |
| 9 à 12 | Trois démonstrations en direct (section 5) |
| 3 | Ce qui ne marche pas encore (section 6) |

## Annexe : services de repli et suivants

Non montrés en direct. À utiliser si un service du corps du document pose problème.

| Service | Commande | Remarques |
|---|---|---|
| Adminer | `wslc run -d --name adminer -p 8081:8080 adminer:6.1.1` | Interface http://localhost:8081. Le champ « Server » doit contenir le nom du conteneur : résolution de noms non confirmée |
| RabbitMQ | `wslc run -d --name rabbit -p 5672:5672 -p 15672:15672 rabbitmq:4-management` | Interface http://localhost:15672, `guest` / `guest` à tester. Tag flottant à épingler. Le statut des variables `RABBITMQ_DEFAULT_USER` et `RABBITMQ_DEFAULT_PASS` est ambigu en 4.x |
| MongoDB | `wslc run -d --name mongo -p 27017:27017 -v mongodata:/data/db mongo` | N'utilisez pas de montage depuis `/mnt/c`. Test : `wslc exec mongo mongosh --eval "db.runCommand({ping:1})"` |
| MariaDB | `wslc run -d --name maria -e MARIADB_ROOT_PASSWORD=secret -p 3306:3306 mariadb:lts` | Test : `wslc exec maria mariadb -uroot -psecret -e "select 1"` (nom du client supposé) |
| Keycloak | `wslc run -d --name keycloak -p 8080:8080 -e KC_BOOTSTRAP_ADMIN_USERNAME=admin -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:26.8.0 start-dev` | Console http://localhost:8080/admin. Prévoir 2 Go de mémoire. Le mode `start-dev` convient au développement seulement |
| RustFS | `wslc run -d --name rustfs -p 9100:9000 -p 9001:9001 -v rustfs-data:/data -e RUSTFS_ACCESS_KEY=<clé> -e RUSTFS_SECRET_KEY=<secret> rustfs/rustfs:latest /data` | Version 1.0.1 du 3 octobre 2026, très récente. Le port 9000 de l'API est remappé en 9100 pour éviter SonarQube. Utilisateur `10001:10001` dans le conteneur |
| pgAdmin | `wslc run -d --name pgadmin -e PGADMIN_DEFAULT_EMAIL=admin@example.com -e PGADMIN_DEFAULT_PASSWORD=secret -p 8082:80 dpage/pgadmin4:9.18` | Interface http://localhost:8082. Inutile sans un PostgreSQL joignable |

## Sources

Pages consultées le 4 octobre 2026.

- [Microsoft Learn : démarrer avec les conteneurs WSL](https://learn.microsoft.com/windows/wsl/tutorials/wsl-containers) (29 septembre 2026)
- [Microsoft Learn : wsl container](https://learn.microsoft.com/windows/wsl/wsl-container) (29 septembre 2026)
- [Annonce de la disponibilité générale](https://blogs.windows.com/windowsdeveloper/2026/09/29/wsl-containers-now-generally-available/) (29 septembre 2026)
- [Architecture de WSLC](https://devblogs.microsoft.com/commandline/wslc-architecture-deep-dive/) (29 septembre 2026)
- [Annonce de la préversion publique](https://devblogs.microsoft.com/commandline/wsl-container-is-now-available-for-public-preview/) (29 juin 2026)
- [Dépôt microsoft/WSL : versions](https://github.com/microsoft/WSL/releases) et [issues WSLC](https://github.com/microsoft/WSL/issues?q=is%3Aissue+wslc+sort%3Aupdated-desc)
- Images et guides : [PostgreSQL](https://hub.docker.com/_/postgres), [Redis](https://hub.docker.com/_/redis), [Solr](https://solr.apache.org/guide/solr/latest/deployment-guide/solr-in-docker.html), [Mailpit](https://hub.docker.com/r/axllent/mailpit), [SonarQube](https://hub.docker.com/_/sonarqube), [Grafana](https://hub.docker.com/r/grafana/grafana), [Gitea](https://docs.gitea.com/installation/install-with-docker), [Meilisearch](https://www.meilisearch.com/docs/learn/self_hosted/install_meilisearch_locally), [Prometheus](https://prometheus.io/docs/prometheus/latest/installation/), [SeaweedFS](https://github.com/seaweedfs/seaweedfs)
- Recherches détaillées, avec les sources de chaque constat : `_bmad-output/planning-artifacts/research/`
