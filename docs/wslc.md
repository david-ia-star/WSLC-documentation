# WSLC : lancer des conteneurs Linux depuis Windows

Version de référence : WSL 3.0.1 (stable depuis le 29 septembre 2026). Rédigé le 4 octobre 2026.

> **Aucune commande n'a été exécutée sous Windows : toutes sont à tester.** Elles viennent des documentations officielles de Microsoft et de chaque projet, en PowerShell, avec des volumes nommés. Quand une commande passe un test, son étiquette devient **testé**. Les notes de l'orateur sont à la fin du fichier.

## Au programme

| Minutes | Contenu |
|---|---|
| 2 | Ce qu'est WSLC et installation (sections 1 et 2) |
| 2 | De Docker à `wslc` et premier conteneur (sections 3 et 4) |
| 9 | Trois démonstrations de 3 minutes : Mailpit, Grafana, Gitea (section 5) |
| 3 | Limites actuelles (section 7) |
| 2 | Questions |

## 1. Ce qu'est WSLC

WSLC (WSL Containers) est un runtime de conteneurs Linux livré avec WSL. Il comprend une interface en ligne de commande (CLI), `wslc.exe` (alias `container.exe`), et une API pour les applications Windows. Aucun moteur séparé n'est à installer.

Microsoft l'a ouvert en préversion publique le 29 juin 2026 (WSL 2.9.3), puis l'a déclaré stable le 29 septembre 2026 avec WSL 3.0.1. Toute instruction qui mentionne `wsl --update --pre-release` est périmée : elle date de la préversion.

Chaque session WSLC (le contexte qui exécute vos conteneurs) tourne dans sa propre machine virtuelle, et les conteneurs utilisent le noyau Linux de WSL 2. Les dossiers Windows sont partagés par virtiofs, un système de fichiers partagé entre Windows et la machine virtuelle. Les volumes nommés sont des disques virtuels VHD au format Linux natif.

L'accès aux fichiers du disque Windows ralentit nettement les outils Linux. Virtiofs est plus rapide que Plan 9 sans atteindre le système de fichiers natif. C'est pourquoi les démonstrations utilisent des volumes nommés (`-v nom:/chemin`) plutôt que des dossiers Windows, et pourquoi Microsoft conseille de garder le code sur le système de fichiers Linux.

## 2. Installer et vérifier

Microsoft annonce WSL 2.9.3 comme version minimale. Ce document a été écrit sur la version stable 3.0.1, que `wsl --update` installe. Les prérequis Windows (numéro de build, virtualisation) ne figurent dans aucune page consultée : vérifiez-les sur la page d'installation de WSL.

```powershell
wsl --update
wsl --version
wslc version
wslc run --rm hello-world
```

- `wsl --version` doit afficher 2.9.3 ou une version ultérieure, et `wslc version` doit répondre.
- Si PowerShell ne trouve pas `wslc`, ouvrez une nouvelle fenêtre : un terminal ouvert avant la mise à jour ne connaît peut-être pas encore la commande.
- `wslc run --rm hello-world` télécharge l'image si elle manque, puis affiche un message de bienvenue.

## 3. De Docker à `wslc`

`wslc run` annonce les options courantes de `docker run` (`-d`, `-p`, `-e`, `-v`, `--name`, `--rm`, `-it`). Confirmez la liste avec `wslc run --help`. Les lignes marquées « préversion » viennent d'une issue tierce de la préversion : confirmez-les avec `wslc <COMMANDE> --help`.

| Docker | `wslc` | Remarque |
|---|---|---|
| `docker run` | `wslc run` | Options courantes annoncées |
| `docker exec` | `wslc exec <nom> <commande>` | |
| `docker logs` | `wslc container logs <nom>` | |
| `docker ps` | `wslc container list` (ou `ps`) | `--all` inclut les conteneurs arrêtés |
| `docker images` | `wslc image list` (ou `ls`) | |
| `docker pull` | `wslc image pull <image>` | Préversion |
| `docker start` | `wslc container start <nom>` | Préversion |
| `docker stop` | `wslc container stop <nom>` | |
| `docker rm` | `wslc container rm <nom>` | Préversion |
| `docker volume ls`, `docker volume rm` | `wslc volume list`, `wslc volume rm <nom>` | Préversion |
| `docker network create` | `wslc network create <nom>` | Préversion. `list`, `rm`, `connect` et `disconnect` existent aussi |
| `docker cp` | `wslc container cp` | Issue #41793 : échoue sur les liens symboliques |
| `docker inspect` | `wslc container inspect <nom>`, `wslc image inspect <image>` | |
| `docker build` | `wslc build -t <nom> .` | Fichier par défaut : `Containerfile`. La prise en charge du nom `Dockerfile` et de l'option `-f` n'est pas vérifiée |
| `docker system prune` | `wslc container prune`, `wslc image prune` | |
| `docker stats` | `wslc stats` | |
| `docker compose` | **absent** | Voir la section 7 |
| `--restart` | **non confirmé** | Voir la section 7 |

Trois écarts comptent d'emblée pour un utilisateur de Docker : compose n'existe pas, `--restart` n'est pas confirmé et la résolution de noms entre conteneurs n'est pas confirmée. Le détail est en section 7.

Pour monter le dossier d'un projet comme avec Docker (à tester, avec le ralentissement décrit en section 1) :

```powershell
wslc run --rm -it -v "${PWD}:/app" ubuntu:latest ls /app
```

La liste complète des commandes : `wslc --help`. Aucune page de référence officielle ne détaille toutes les options.

## 4. Un premier conteneur

Cet exemple lance un serveur web publié sur le port 8080 de Windows. Les commandes viennent du tutoriel Microsoft du 29 septembre 2026.

```powershell
wslc run -d --rm -p 8080:80 --name web nginx
Start-Sleep 2
curl.exe localhost:8080
wslc container list
wslc exec web cat /etc/os-release
wslc container stop web
wslc container list --all
```

PowerShell définit `curl` comme alias de `Invoke-WebRequest` : `curl.exe` appelle le vrai programme. Comme `--rm` est actif, le dernier `wslc container list --all` ne doit plus montrer `web`.

## 5. Trois démonstrations en une commande

Chaque service se lance seul avec un `wslc run`, car compose n'existe pas et la résolution de noms entre conteneurs n'est pas confirmée. Les trois services montrés sont Mailpit, Grafana et Gitea : ils démarrent vite, ont une interface web et n'exigent aucun réglage de la machine. Les autres services sont dans le tableau de la section 6.

### Plan de ports

| Port | Service |
|---|---|
| 1025, 8025 | Mailpit (SMTP, interface) |
| 3000 | Grafana |
| 3001 | Gitea (le 3000 est pris par Grafana) |
| 3306 | MariaDB |
| 5432 | PostgreSQL |
| 5672, 15672 | RabbitMQ |
| 6379 | Redis |
| 7700 | Meilisearch |
| 8080 | Keycloak (nginx de la section 4 utilise aussi le 8080 : arrêtez-le avant) |
| 8081, 8082 | Adminer, pgAdmin |
| 8333, 8888, 23646 | SeaweedFS (S3, fichiers, administration) |
| 8983 | Solr |
| 9000 | SonarQube |
| 9001, 9100 | RustFS (console, API) |
| 9090 | Prometheus |
| 27017 | MongoDB |

### Avant de lancer

Vérifiez les ports côté Windows. Si une ligne s'affiche, le port est pris : changez le premier numéro du `-p` correspondant. Windows peut aussi réserver des plages de ports.

```powershell
Get-NetTCPConnection -State Listen | Where-Object LocalPort -in 1025,3000,3001,5432,6379,7700,8025,8080,8333,8888,8983,9000,9090,23646
netsh interface ipv4 show excludedportrange protocol=tcp
```

Une fonction d'attente évite de vérifier un service avant qu'il réponde :

```powershell
function Wait-Http($url) {
  for ($i = 0; $i -lt 60; $i++) {
    try { Invoke-WebRequest $url -UseBasicParsing -TimeoutSec 2 | Out-Null; return } catch { Start-Sleep 2 }
  }
  Write-Warning "Pas de réponse de $url"
}
```

Pour repartir de zéro entre deux essais (un conteneur arrêté garde son nom et un volume garde ses données), exécutez ces lignes. Les erreurs « introuvable » pour un conteneur ou un volume qui n'existe pas sont sans gravité :

```powershell
'web','mailpit','grafana','gitea','pg','redis','solr','sonarqube','meili','prometheus','seaweed' | ForEach-Object { wslc container stop $_; wslc container rm $_ }
'pgdata','redisdata','solrdata','grafana-data','gitea-data','meili-data','weed-data' | ForEach-Object { wslc volume rm $_ }
```

Les démonstrations utilisent des mots de passe d'exemple (`admin`, `secret`) et publient les ports sur toutes les interfaces de Windows. Sur un réseau partagé, testez `-p 127.0.0.1:3000:3000` à la place de `-p 3000:3000`, et répondez à l'invite du pare-feu Windows avant de commencer.

### Mailpit : serveur de mail factice

```powershell
wslc run -d --name mailpit -p 8025:8025 -p 1025:1025 axllent/mailpit
```

- **Interface :** http://localhost:8025. Le serveur SMTP écoute sur le port 1025.
- **Test :** `Wait-Http http://localhost:8025`, puis l'envoi d'un message :

```powershell
@"
From: a@x.test
To: b@x.test
Subject: Essai WSLC

Bonjour depuis WSLC.
"@ | Set-Content -Encoding ascii mail.txt
curl.exe --url smtp://localhost:1025 --mail-from a@x.test --mail-rcpt b@x.test -T mail.txt
```

Le message doit apparaître dans l'interface.

- **Piège :** la version est la 1.31.4 (3 octobre 2026), mais le format de son tag Docker n'est pas vérifié : épinglez-la après contrôle. MailHog expose les mêmes ports et n'est plus maintenu.

### Grafana : tableaux de bord

```powershell
wslc run -d --name grafana -p 3000:3000 -v grafana-data:/var/lib/grafana grafana/grafana:13.2.3
```

- **Interface :** http://localhost:3000. Connexion `admin` / `admin`, puis Grafana impose de changer le mot de passe : annoncez-le au public.
- **Test :** `Wait-Http http://localhost:3000`.
- **Piège :** si vous relancez avec le volume existant, le mot de passe a déjà changé : supprimez `grafana-data`. Les droits d'écriture du volume nommé ne sont pas vérifiés ; si Grafana s'arrête avec « permission denied », lisez `wslc container logs grafana`. La version 13.2.3 (29 septembre 2026) corrige des failles de sécurité ([notes de version](https://github.com/grafana/grafana/releases)). Lancé seul, Grafana n'affiche que ses tableaux de bord et sa source de test intégrée (à confirmer) : de vraies métriques demandent Prometheus dans un second conteneur. L'image par défaut repose sur Alpine.

### Gitea : forge Git

```powershell
wslc run -d --name gitea -p 3001:3000 -p 2222:22 -v gitea-data:/data gitea/gitea:28.0.0
```

- **Interface :** http://localhost:3001. L'assistant d'installation s'affiche à la première visite, avec SQLite par défaut : un seul conteneur suffit.
- **Test :** `Wait-Http http://localhost:3001`. Dans l'assistant, l'URL de base doit valoir `http://localhost:3001/` et le port SSH `2222` : l'assistant peut proposer les ports internes (3000 et 22), corrigez-les.
- **Piège :** la redirection du port SSH 22 vers 2222 est une déduction, pas un texte de la documentation, et le port 2222 peut entrer en conflit avec OpenSSH de Windows : retirez `-p 2222:22` si vous ne montrez pas le SSH. `latest` pointe sur la version 28.0 (30 septembre 2026). Les images avec et sans root (`-rootless`) sont incompatibles : ne mélangez pas leurs données.

## 6. Les autres services

Même gabarit pour tous. Les services « Principal » sont décrits dans la recherche détaillée. Les services « Repli » ne sont pas montrés en direct : utilisez-les si un service principal pose problème.

| Service | Commande | Interface et test | Piège | Rôle |
|---|---|---|---|---|
| PostgreSQL | `wslc run -d --name pg -e POSTGRES_PASSWORD=secret -p 5432:5432 -v pgdata:/var/lib/postgresql postgres:18.6` | `do { Start-Sleep 2; wslc exec pg pg_isready -U postgres } until ($LASTEXITCODE -eq 0)`, puis `wslc exec pg psql -U postgres -c 'select version();'` | Depuis la version 18, le volume déclaré est `/var/lib/postgresql` (`PGDATA` vaut `/var/lib/postgresql/18/docker`). Pour la version 17 et les précédentes, c'est `/var/lib/postgresql/data`. Ne réutilisez pas `pgdata` d'une version majeure à l'autre. Le port 5432 et `psql` sont standards mais absents de la page consultée | Principal |
| Redis | `wslc run -d --name redis -p 6379:6379 -v redisdata:/data redis:8.10.2 redis-server --save 60 1` | `wslc exec redis redis-cli ping` (réponse : `PONG`) | Triple licence depuis la 8.0 (RSALv2, SSPLv1, AGPLv3, selon la [page Docker Hub](https://hub.docker.com/_/redis)). L'alternative libre est Valkey (`valkey/valkey:9`), image publiée par le projet et absente des images officielles de Docker. Pour le fichier journal AOF, `--appendonly yes` n'est pas vérifié | Principal |
| Solr | `wslc run -d --name solr -p 8983:8983 -v solrdata:/var/solr solr:10.0.0 solr-precreate gettingstarted` | `Wait-Http http://localhost:8983/` : le premier démarrage (JVM) est lent | Le guide officiel monte un dossier de l'hôte ; un volume nommé est plus prudent, car les droits de l'utilisateur 8983 ne sont pas documentés. Supprimez `solrdata` avant de relancer. Ligne 9.x : `solr:9.10.1` | Principal |
| SonarQube | `wslc run -d --name sonarqube -p 9000:9000 sonarqube:26.9.0.129388-community` | Attendre que `http://localhost:9000/api/system/status` indique `UP` (plusieurs minutes), puis connexion `admin` / `admin` | Exige dans la machine virtuelle `vm.max_map_count=524288`, `fs.file-max=131072`, `ulimit -n 131072` et `ulimit -u 8192`. Comment les régler sous WSLC est inconnu : **ne le lancez pas en direct sans test**. Repli à tester, absent des pages consultées : `-e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true`. La base H2 convient à une démonstration seulement | Principal |
| Meilisearch | `wslc run -d --name meili -p 7700:7700 -e MEILI_ENV=development -v meili-data:/meili_data getmeili/meilisearch:v1.54.3` | `Wait-Http http://localhost:7700/health`, puis la page de prévisualisation http://localhost:7700 (mode développement) | Le tag `v1.37` de la documentation est périmé ; les tags `v1` et `v1.52` paraissent en retard au 1er octobre 2026 : épinglez `v1.54.3`. En production, une clé maîtresse de 16 octets minimum devient obligatoire. Supprimez `meili-data` si vous changez de version | Principal |
| Prometheus | `wslc run -d --name prometheus -p 9090:9090 prom/prometheus:v3.15.0` | http://localhost:9090 : ouvrez **Status** > **Targets** pour vérifier que Prometheus se surveille lui-même, puis lancez la requête `up` | Le fichier d'exemple du dépôt collecte `localhost:9090`. On ne sait pas si l'image l'utilise par défaut : la page Targets le montrera | Principal |
| SeaweedFS | `wslc run -d --name seaweed -p 8333:8333 -p 8888:8888 -p 23646:23646 -v weed-data:/data -e AWS_ACCESS_KEY_ID=admin -e AWS_SECRET_ACCESS_KEY=secret -e S3_BUCKET=my-bucket chrislusf/seaweedfs:4.48` | Administration : http://localhost:23646 ; fichiers : http://localhost:8888 ; API S3 : port 8333. Client AWS à installer : `$env:AWS_ACCESS_KEY_ID='admin'; $env:AWS_SECRET_ACCESS_KEY='secret'; $env:AWS_DEFAULT_REGION='us-east-1'; aws s3 ls --endpoint-url http://localhost:8333` (commande qui ne vient d'aucune source) | Remplace MinIO, dont le dépôt est archivé depuis le 25 avril 2026. L'image démarre par défaut en mode tout-en-un (`mini -dir=/data`) et crée le bucket `my-bucket`. Sans identifiants, SeaweedFS passe en mode « Allow All » : il n'exige plus aucune authentification | Principal |
| Adminer | `wslc run -d --name adminer -p 8081:8080 adminer:6.1.1` | http://localhost:8081 | Le champ « Server » doit contenir le nom du conteneur : résolution de noms non confirmée. Sinon, l'adresse de l'hôte, qui échoue sous Alpine (#41769) | Repli |
| RabbitMQ | `wslc run -d --name rabbit -p 5672:5672 -p 15672:15672 rabbitmq:4-management` | http://localhost:15672, `guest` / `guest` | Tag flottant à épingler. La prise en charge de `RABBITMQ_DEFAULT_USER` et `RABBITMQ_DEFAULT_PASS` est ambiguë en 4.x | Repli |
| MongoDB | `wslc run -d --name mongo -p 27017:27017 -v mongodata:/data/db mongo` | `wslc exec mongo mongosh --eval 'db.runCommand({ping:1})'` | Tag flottant à épingler. N'utilisez pas de montage depuis `/mnt/c` | Repli |
| MariaDB | `wslc run -d --name maria -e MARIADB_ROOT_PASSWORD=secret -p 3306:3306 mariadb:lts` | `wslc exec -e MYSQL_PWD=secret maria mariadb -uroot -e "select 1"` après l'initialisation | Tag flottant à épingler. Le nom du client `mariadb` est supposé | Repli |
| Keycloak | `wslc run -d --name keycloak -p 8080:8080 -e KC_BOOTSTRAP_ADMIN_USERNAME=admin -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:26.8.0 start-dev` | Console : http://localhost:8080/admin | Prévoyez 2 Go de mémoire. Le mode `start-dev` convient au développement seulement | Repli |
| RustFS | `wslc run -d --name rustfs -p 9100:9000 -p 9001:9001 -v rustfs-data:/data -e RUSTFS_ACCESS_KEY=demoaccess -e RUSTFS_SECRET_KEY=demosecret123 rustfs/rustfs:latest /data` | Console : http://localhost:9001 | Version 1.0.1 du 3 octobre 2026, très récente. Tag flottant à épingler. Le conteneur s'exécute sous l'utilisateur `10001:10001` : droits du volume à vérifier. L'API est redirigée vers le 9100 pour éviter SonarQube. Longueur minimale des identifiants non vérifiée | Repli |
| pgAdmin | `wslc run -d --name pgadmin -e PGADMIN_DEFAULT_EMAIL=admin@example.com -e PGADMIN_DEFAULT_PASSWORD=secret -p 8082:80 dpage/pgadmin4:9.18` | http://localhost:8082 | Inutile sans un PostgreSQL joignable (résolution de noms non confirmée) | Repli |

## 7. Limites actuelles

Relevées au moment de la rédaction, sur WSL 3.0.1. La colonne « À dire » marque les lignes à présenter à voix haute.

| Limite | Détail | Statut | À dire |
|---|---|---|---|
| Compose absent | Microsoft la classe comme première demande des utilisateurs et comme priorité des prochaines versions. Visual Studio affiche : « Docker Compose is not supported with the WSL container runtime (wslc) » ([issue #41754](https://github.com/microsoft/WSL/issues/41754)). En attendant, enchaînez les `wslc run` dans un script `.ps1` : la documentation consultée ne propose pas d'équivalent | Vérifié par deux sources | Oui |
| DNS entre conteneurs | Aucun exemple documenté de résolution par nom sur un réseau créé avec `wslc network create`. Les commandes `connect` et `disconnect` existent | À tester | Oui |
| Politique de redémarrage | `--restart` était absent en préversion. La version 3.0.1 ajoute seulement la commande ponctuelle `wslc container restart` | Non revérifié sur 3.0.1 | Oui |
| Réglages noyau de la machine virtuelle | SonarQube et OpenSearch exigent un `vm.max_map_count` élevé, dont le réglage sous WSLC est inconnu | Non résolu | Oui |
| Options refusées | `--privileged`, `--cap-add`, `--cap-drop`, `--security-opt`, `--read-only` et `--pids-limit` sont rejetées par la version 2.9.13. `--memory`, `--cpus` et `--ulimit` sont acceptées | Source tierce, non revérifiée sur 3.0.1 | |
| GPU sous Alpine | `--gpus all` fait échouer la création d'un conteneur Alpine ([issue #41791](https://github.com/microsoft/WSL/issues/41791)). Les images Debian et Ubuntu fonctionnent | Ouvert | |
| USB | Le passage de périphérique n'était pas supporté en préversion | Non revérifié | |
| Adresse de l'hôte | `host.wslc.internal` ne se résout pas dans les conteneurs Alpine ([issue #41769](https://github.com/microsoft/WSL/issues/41769)) | Ouvert | |
| Proxy d'entreprise | Les variables proxy de l'hôte ne sont pas reprises ([issue #41794](https://github.com/microsoft/WSL/issues/41794)) : derrière un proxy, téléchargez les images avant | Ouvert | |
| Autres issues ouvertes | `wslc container cp` échoue sur les liens symboliques ([#41793](https://github.com/microsoft/WSL/issues/41793)). `wslc run` renvoie parfois ERROR_TIMEOUT ([#41740](https://github.com/microsoft/WSL/issues/41740)) | Ouvert | |

Questions sans réponse dans les sources consultées :

- Le runtime sous-jacent (containerd ou runtime propre à Microsoft).
- Une comparaison officielle ou testée avec Docker Desktop ou Podman.
- Les prérequis Windows (numéro de build, virtualisation).
- La mémoire par défaut de la machine virtuelle et la façon de l'augmenter.

## Sources

Pages consultées le 4 octobre 2026.

- [Microsoft Learn : démarrer avec les conteneurs WSL](https://learn.microsoft.com/windows/wsl/tutorials/wsl-containers) (29 septembre 2026)
- [Microsoft Learn : wsl container](https://learn.microsoft.com/windows/wsl/wsl-container) (29 septembre 2026)
- [Annonce de la disponibilité générale](https://blogs.windows.com/windowsdeveloper/2026/09/29/wsl-containers-now-generally-available/) (29 septembre 2026)
- [Architecture de WSLC](https://devblogs.microsoft.com/commandline/wslc-architecture-deep-dive/) (29 septembre 2026)
- [Annonce de la préversion publique](https://devblogs.microsoft.com/commandline/wsl-container-is-now-available-for-public-preview/) (29 juin 2026)
- [Dépôt microsoft/WSL : versions](https://github.com/microsoft/WSL/releases) et [issues WSLC](https://github.com/microsoft/WSL/issues?q=is%3Aissue+wslc+sort%3Aupdated-desc)
- [Issue de la préversion qui liste l'arbre de commandes](https://github.com/kubernetes-sigs/kind/issues/4208) (lignes « préversion » de la section 3)
- Images et guides : [PostgreSQL](https://hub.docker.com/_/postgres), [Redis](https://hub.docker.com/_/redis), [Solr](https://solr.apache.org/guide/solr/latest/deployment-guide/solr-in-docker.html), [Mailpit](https://hub.docker.com/r/axllent/mailpit), [SonarQube](https://hub.docker.com/_/sonarqube), [Grafana](https://hub.docker.com/r/grafana/grafana), [Gitea](https://docs.gitea.com/installation/install-with-docker), [Meilisearch](https://www.meilisearch.com/docs/learn/self_hosted/install_meilisearch_locally), [Prometheus](https://prometheus.io/docs/prometheus/latest/installation/), [SeaweedFS](https://github.com/seaweedfs/seaweedfs)

## Notes de l'orateur (hors rejeu)

### Choix des démonstrations

Mailpit, Grafana et Gitea par défaut. Le choix définitif se fait après les tests : gardez les trois services qui démarrent le plus vite sur la machine de l'exposé.

### Checklist

À cocher sur la machine de l'exposé. Chaque test réussi fait passer l'étiquette du bloc de **à tester** à **testé**.

- [ ] `wsl --version` affiche 2.9.3 ou une version ultérieure, et `wslc version` répond.
- [ ] `wslc run --rm hello-world` et la démonstration nginx passent.
- [ ] Les ports sont libres et aucune plage réservée ne les touche (section 5).
- [ ] Les trois services en direct démarrent, répondent comme indiqué et tiennent dans la mémoire de la machine virtuelle.
- [ ] Deux conteneurs sur un réseau créé avec `wslc network create` se joignent par leur nom.
- [ ] `wslc run --help` indique si `--restart`, `--env-file` et `--memory` sont pris en charge.
- [ ] `vm.max_map_count` se règle dans la machine virtuelle WSLC et SonarQube démarre.
- [ ] La commande `wslc image pull`, la suppression de conteneurs et de volumes (`container rm`, `volume rm`) existent bien sous ces noms.
- [ ] Le script de téléchargement ci-dessous passe sans erreur « manifest unknown ».
- [ ] Le bind mount `-v "${PWD}:/app"` fonctionne avec PowerShell.
- [ ] Le tag Docker de Mailpit est épinglé.
- [ ] La veille de l'exposé : tags, statuts des issues de la section 7 et étiquettes sont revérifiés.

### Plan B

- Téléchargez toutes les images la veille. Un téléchargement sur le réseau de la salle peut échouer, et les variables proxy de l'hôte ne sont pas reprises (#41794).
- Si un service ne répond pas au bout de 60 secondes, passez au suivant ou à un service de repli de la section 6.
- Enregistrez chaque démonstration en vidéo avant l'exposé.
- Arrêtez les démonstrations précédentes avant d'en lancer une autre : la mémoire de la machine virtuelle est limitée.

### À vérifier avant de rejouer

- Le tag Docker de Mailpit et la commande d'envoi SMTP, qui ne viennent pas des pages consultées.
- Le port 5432 et la commande `psql` pour PostgreSQL, absents de la page Docker Hub consultée.
- Les options `--save` et `--appendonly yes` de Redis.
- Le lancement de Prometheus : la cible `localhost:9090` apparaît-elle dans Status > Targets ?
- La commande `aws s3 ls` de SeaweedFS.
- Les droits d'écriture des volumes nommés (Grafana, Solr, Gitea, RustFS).
- Les guillemets PowerShell des commandes `psql`, `mongosh` et `mariadb`, sous PowerShell 5.1 et sous PowerShell 7.

### Si on vous demande « pourquoi pas Docker Desktop ? »

Aucune comparaison officielle ni testée n'existe. Ce que la documentation établit : WSLC arrive avec WSL sans moteur séparé à installer, sa CLI ressemble à celle de Docker, et il n'a pas encore compose.

### Téléchargement préalable des images

```powershell
$images = 'nginx','axllent/mailpit','grafana/grafana:13.2.3','gitea/gitea:28.0.0',
  'postgres:18.6','redis:8.10.2','solr:10.0.0','sonarqube:26.9.0.129388-community',
  'getmeili/meilisearch:v1.54.3','prom/prometheus:v3.15.0','chrislusf/seaweedfs:4.48',
  'adminer:6.1.1','rabbitmq:4-management','mongo','mariadb:lts',
  'quay.io/keycloak/keycloak:26.8.0','rustfs/rustfs:latest','dpage/pgadmin4:9.18'
foreach ($image in $images) { wslc image pull $image }
```
