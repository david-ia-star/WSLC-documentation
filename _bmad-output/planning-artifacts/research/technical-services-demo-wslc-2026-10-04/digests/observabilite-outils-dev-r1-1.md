# Digest — observabilité et outils de dev (round 1, assistant 2)

Accès : 2026-10-04. Outil : WebFetch seul (pages résumées par un petit modèle : chaînes exactes « telles que retournées »). Les âges relatifs de Docker Hub (« il y a 5 jours ») sont convertis en dates par rapport au 2026-10-04. Aucune requête sur les issues GitHub (budget) : le statut « corrigé depuis » n'est connu que pour pgAdmin 9.18.

Format : constat | source | éditeur | date | confiance | classe

## 1. Grafana (`grafana/grafana`, édition OSS)
- Image `grafana/grafana` (Verified Publisher Grafana Labs). Démarrage : `docker run -d --name=grafana -p 3000:3000 grafana/grafana`. | https://hub.docker.com/r/grafana/grafana | Docker Hub / Grafana Labs | page au 2026-10-04 | haute | image
- Tag `13.2.3` = `13.2` = `latest`, poussé il y a ~5 jours (~2026-09-29), 459,92 Mo linux/amd64. Variantes : `13.2.3-slim` (308 Mo, sans plugins), `13.2.3-ubuntu` (475 Mo), `latest-ubuntu-slim`, `latest-distroless-slim`. Multi-architecture. | https://hub.docker.com/r/grafana/grafana/tags | Docker Hub | 2026-10-04 | haute | version/image
- Dernière release GitHub v13.2.3, publiée le 2026-09-29, release de sécurité (3 CVE dont CVE-2026-13719). Raison d'utiliser 13.2.3 plutôt qu'un tag ancien en cache. | https://api.github.com/repos/grafana/grafana/releases/latest | GitHub / Grafana | 2026-09-29 | haute | version/maintenance
- Port 3000. Identifiants par défaut admin/admin (page Docker Hub seule ; la doc Grafana ne les indique pas). Invite de changement de mot de passe à la première connexion : non vérifiée. | page Hub ci-dessus | moyenne | port/ui
- Interface web sur le port 3000 (vérification : ouvrir http://localhost:3000, page de connexion attendue). `/api/health` : non récupéré.
- Volume de données `/var/lib/grafana`. Défauts : `GF_PATHS_CONFIG=/etc/grafana/grafana.ini`, `GF_PATHS_PROVISIONING=/etc/grafana/provisioning`, `GF_PATHS_PLUGINS=/var/lib/grafana/plugins`. | https://grafana.com/docs/grafana/latest/setup-grafana/configure-docker/ et .../installation/docker/ | Grafana Labs | 2026-10-04 | haute | volume
- Variables obligatoires : AUCUNE. Optionnelles vues : `GF_LOG_LEVEL`, `GF_SERVER_ROOT_URL`, `GF_PLUGINS_PREINSTALL`, `GF_<Section>_<Key>__FILE`. `GF_SECURITY_ADMIN_USER/PASSWORD` : non récupérées (non vérifiées). | idem | haute | env
- Image de base : le tag par défaut est Alpine (« recommandé » par la doc, « smallest and most secure »). Ubuntu via `-ubuntu`, distroless aussi proposé. | page configure-docker | haute | image
- Licence : AGPL-3.0. Gratuit pour une démo (obligations de copyleft réseau seulement si on modifie et héberge). | https://raw.githubusercontent.com/grafana/grafana/main/LICENSE | GitHub | page au 2026-10-04 | haute | licence
- Piège : sans volume, les données ne survivent pas au conteneur ; pour les bind mounts, la doc dit d'utiliser `--user "$(id -u)"`. | page installation/docker | haute | piège
- La doc recommande l'image Enterprise `grafana/grafana-enterprise`, l'OSS étant `grafana/grafana` ; comportement de licence de l'image Enterprise non récupéré.
- Démo en un seul conteneur : fonctionne seul (UI, tableaux de bord, source de données TestData). Pour de vraies métriques, source Prometheus `http://<hôte>:9090` : second conteneur ou fichier de provisioning dans `/etc/grafana/provisioning`. Nom d'hôte inter-conteneurs (`host.docker.internal`) sous wslc : non vérifié. TestData intégré ou plugin : non vérifié. | moyenne | ui/piège
- Temps de démarrage et RAM : non trouvés. Réglages noyau et minimums mémoire : aucun mentionné dans les pages lues.

## 2. Prometheus (`prom/prometheus`)
- Image `prom/prometheus`. Tags `latest` = `v3` = `v3.15.0`, poussé il y a ~9 jours (~2026-09-25) ; variantes `-distroless` et `-busybox`. Aussi `v3.13.4` (correctif de l'ancienne ligne) et `main` (build de dev). Multi-architecture. ~93-109 Mo compressé. | https://hub.docker.com/r/prom/prometheus/tags et .../prom/prometheus | Docker Hub / Prometheus | 2026-10-04 | haute | image/version
- Dernière release GitHub v3.15.0 « 3.15.0 / 2026-09-24 », publiée le 2026-09-25. Aucune mention LTS dans les métadonnées. | https://api.github.com/repos/prometheus/prometheus/releases/latest | GitHub | 2026-09-25 | haute | version/maintenance
- Port 9090. Démarrage Hub : `docker run --name prometheus -d -p 127.0.0.1:9090:9090 prom/prometheus`. La doc : `docker run -p 9090:9090 prom/prometheus` lance avec une « sample configuration ». | https://prometheus.io/docs/prometheus/latest/installation/ | projet Prometheus | page au 2026-10-04 | haute | port
- Configuration `/etc/prometheus/prometheus.yml` (montable). Données `/prometheus`. Aucune variable obligatoire. Si on surcharge la commande, il faut remettre à la main les options par défaut du Dockerfile (avertissement de la doc). | page installation | haute | env/volume/piège
- Interface web intégrée sur le port 9090 (http://localhost:9090/). `/-/healthy` et `/-/ready` : non récupérés.
- Licence : Apache 2.0 (page Docker Hub). Gratuit pour une démo. | haute | licence
- Un conteneur seul permet de montrer la page Graph/Explore avec PromQL sur ses propres métriques (`up`, `prometheus_tsdb_head_series`), si la configuration d'exemple scrute localhost:9090. Cette auto-collecte n'est PAS confirmée explicitement par les pages lues (« sample configuration » seulement) : à vérifier dans Status > Targets. | moyenne | ui
- Image de base du tag par défaut : non indiquée. Temps de démarrage, RAM, réglages hôte : non trouvés.

## 3. Gitea
- Registre : la doc utilise `docker.gitea.com/gitea:28.0.0`. Docker Hub `gitea/gitea` existe aussi (68,6 Mo, mis à jour il y a ~4 jours, ~2026-09-30) et renvoie vers le guide rootless. Tags cités : `:latest`, `:28.0.0`, `:main-nightly`. | https://docs.gitea.com/installation/install-with-docker et https://hub.docker.com/r/gitea/gitea | Gitea / Docker Hub | page au 2026-10-04 | haute | image
- Version : dernière release GitHub v28.0.0, publiée le 2026-09-29. Le passage de 1.x à 28.0.0 est surprenant ; concordant entre la doc et GitHub, mais à vérifier à l'exécution que `gitea/gitea:latest` pointe bien dessus. | https://api.github.com/repos/go-gitea/gitea/releases/latest | GitHub | 2026-09-29 | haute | version
- Rootless : tags `:latest-rootless`, `:1-rootless` (tel qu'écrit dans la doc, peut-être périmé face à la numérotation 28.x), `:28.0.0-rootless`, `:main-nightly-rootless`. Volumes `/var/lib/gitea` (données) et `/etc/gitea` (config). Ports 3000 (web) et 2222 (SSH intégré). UID/GID 1000:1000. Les dossiers de l'hôte demandent `sudo chown 1000:1000 config/ data/` ; les volumes nommés évitent cela. | https://docs.gitea.com/installation/install-with-docker-rootless | Gitea | page au 2026-10-04 | haute | image/port/volume/piège
- Version avec root : données `/data`, ports 3000 (web) et 22 (SSH), variables `USER_UID=1000`, `USER_GID=1000`, `USER` (défaut `git`). Aucune variable obligatoire. Le port 22 entre en conflit avec le SSH de l'hôte : mapper par exemple `-p 2222:22` (inférence de l'assistant, non écrit tel quel). | page install-with-docker | haute | env/port/volume
- SQLite3 par défaut, sans configuration : un seul conteneur suffit (« SQLite3 requires no additional configuration »). Interface web sur le port 3000 : l'assistant d'installation s'affiche à la première visite, SQLite présélectionné (texte non récupéré : non vérifié). Le premier utilisateur devient-il administrateur : non récupéré.
- Piège : les images avec root et sans root ne sont pas compatibles entre elles ; ne pas mélanger les données. | page install-with-docker | haute | piège
- Piège : volumes nommés ou dossiers de l'hôte : ces derniers demandent les bons droits. | haute | piège
- Licence MIT : NON récupérée (non vérifiée). Image de base (Alpine ?) non vérifiée. Temps de démarrage, RAM, réglages hôte : non trouvés.

## 4. pgAdmin (`dpage/pgadmin4`)
- Image `dpage/pgadmin4` (100 M+ de téléchargements, 175,7 Mo, mise à jour il y a ~12 jours, ~2026-09-22). Le tag `latest` est décrit, sans tags datés. | https://hub.docker.com/r/dpage/pgadmin4 | Docker Hub / pgAdmin | page au 2026-10-04 | haute | image
- Version 9.18 publiée le 2026-09-17 (outils PostgreSQL 18.4 inclus). Correctifs de sécurité critiques : contournement d'authentification en mode serveur web, injection d'arguments en sauvegarde/restauration, suivi de liens symboliques ; corrige aussi un problème de connexion lié à Flask-Security-Too. Utiliser `dpage/pgadmin4:9.18` ou `latest` (tag `9.18` non vérifié). | https://www.pgadmin.org/docs/pgadmin4/latest/release_notes_9_18.html | pgAdmin | 2026-09-17 | haute | version/maintenance/piège (corrigé en 9.18)
- Variables obligatoires (verbatim) : `PGADMIN_DEFAULT_EMAIL` et `PGADMIN_DEFAULT_PASSWORD` (ou `PGADMIN_DEFAULT_PASSWORD_FILE`). | https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html | pgAdmin | page au 2026-10-04 | haute | env
- Port 80 (HTTP) par défaut ; 443 avec `PGADMIN_ENABLE_TLS` ; `PGADMIN_LISTEN_PORT` pour le changer. Avec des capacités restreintes (`--cap-drop=ALL`, OpenShift) : repli sur 8080 / 8443. Interface sur http://localhost:<port mappé>, connexion avec l'e-mail et le mot de passe fournis. | idem | haute | port/ui
- Volume de données `/var/lib/pgadmin`. Tourne en UID/GID 5050 : les dossiers de l'hôte demandent `sudo chown -R 5050:5050 <dossier>`. | idem | haute | volume/piège
- Piège de l'adresse e-mail (un domaine `.local` rejeté) : non récupéré, non vérifié ; par précaution une adresse classique du type admin@example.com (suggestion de l'assistant, non sourcée).
- Une démo pgAdmin n'a d'intérêt qu'avec un serveur PostgreSQL : un conteneur seul montre l'interface vide. Joindre PostgreSQL dans la VM demande un second conteneur.
- Licence PostgreSQL : NON récupérée. Image de base : non vérifiée. Temps de démarrage, RAM, réglages hôte : non trouvés.

## Pistes à creuser
- Gitea 28.0.0 : liste des tags Docker Hub (https://hub.docker.com/r/gitea/gitea/tags) et existence de `latest-rootless`.
- Prometheus : https://raw.githubusercontent.com/prometheus/prometheus/main/documentation/examples/prometheus.yml ou le Dockerfile, pour confirmer l'auto-collecte et l'image de base.
- Grafana : `/api/health` ; `GF_SECURITY_ADMIN_PASSWORD` ; possibilité de sauter le changement de mot de passe initial.
- Grafana 13.0.10-ubuntu poussé il y a 5 jours : la ligne 13.0.x semble encore supportée.
- Licences et pièges via les fichiers LICENSE GitHub.

## Cherché et non trouvé
- Temps de démarrage et RAM pour les quatre ; réglages noyau et minimums mémoire dans les pages lues.
- Pièges issus des trackers d'issues (non interrogés).
- Image officielle Docker `_/gitea` : 404 (l'image de Gitea n'est pas une Docker Official Image).
- Releases de `pgadmin-org/pgadmin4` en 404 : la date vient des notes de version de la doc.
- Licences de Gitea et pgAdmin, images de base de Gitea, Prometheus et pgAdmin : non récupérées.
- Rien sur le comportement propre à wslc (non demandé).
