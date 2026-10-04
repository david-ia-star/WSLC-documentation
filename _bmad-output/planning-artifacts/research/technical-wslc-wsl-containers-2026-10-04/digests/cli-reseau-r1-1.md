# Digest — référence CLI, intégration et réseau (round 1, assistant 2)

Accès : 2026-10-04. Les pages Microsoft Learn ont été retournées en texte complet (fiables). Les blogs et pages GitHub sont des résumés : les commandes issues de ces pages sont « telles qu'extraites ». À revérifier avec `wslc <commande> --help` sur une vraie machine en 3.0.1 avant l'exposé.

## Contexte de version
- WSL 3.0.1 (2026-09-29) = version GA. 2.9.13 (pré-version, 2026-09-25), 2.9.12 et 2.9.11 = ligne préversion.
- Une issue ouverte le 2026-10-03 mentionne `wslc 3.0.1.0` sur Windows 10.0.26300.9457, noyau 6.18.40.1-1.
- Tout ce qui date de juin 2026 ou de WSL 2.9.x est périmé (> 1 mois) : marqué PÉRIMÉ.

## A. Référence CLI

1. Prérequis : WSL 2.9.3 ou plus ; `wsl --update` ; `wsl --version` ; `wslc version` ; test `wslc run --rm hello-world`. La page Learn ne mentionne plus `--pre-release` (libellé périmé : préversion). | https://learn.microsoft.com/en-us/windows/wsl/tutorials/wsl-containers | Microsoft Learn | 2026-09-29 | haute | commande
```powershell
wsl --update
wsl --version
wslc version
wslc run --rm hello-world
```
2. Exemples officiels, verbatim. | https://learn.microsoft.com/windows/wsl/wsl-container et tutoriel | Microsoft Learn | 2026-09-29 | haute | commande
```powershell
wslc run --rm -it ubuntu:latest bash -c "echo Hello world from WSL container!"
wslc image ls                      # page de présentation
wslc image list                    # page tutoriel
wslc run -it --rm -d -p 8080:80 --name web nginx       # présentation
wslc run -d --rm -p 8080:80 --name web nginx           # tutoriel
curl localhost:8080
wslc container ps                  # présentation
wslc container list                # tutoriel
wslc exec web cat /etc/os-release
wslc container stop web
```
Les deux orthographes `ls`/`ps` et `list` figurent dans la doc officielle. À cause de `--rm`, nginx est supprimé après `stop`.
3. Découverte (encadré du tutoriel) : `wslc --help`, `wslc <COMMAND> --help`, `wslc image list`, `wslc container list`, `wslc container list --all`, `wslc stats`. | tutoriel | Microsoft Learn | 2026-09-29 | haute | commande
4. Build : `wslc build -t <tag> .`, fichier par défaut `Containerfile`. | tutoriel | Microsoft Learn | 2026-09-29 | haute | commande
```powershell
wslc build -t helloworld-django .
wslc run -d --rm -p 8000:8000 --name django helloworld-django
wslc container logs django
wslc exec django uname
wslc container stop django
```
La syntaxe du Containerfile est celle d'un Dockerfile (`FROM`, `WORKDIR`, `COPY`, `RUN`, `EXPOSE`, `CMD`). Une issue tierce cite aussi `image build`. Un fichier nommé `Dockerfile` ou l'option `-f` : non vérifiés.
5. Inspection et logs : `wslc container inspect <container-id>`, `wslc container logs <container-id>`, `wslc image inspect <image>`. | tutoriel | haute | commande
6. Nettoyage : `wslc container prune`, `wslc image prune`. | tutoriel | haute | commande
7. Arbre complet des commandes (issue tierce « Use wslc as a backend for kind », date non capturée, période préversion : PÉRIMÉ par rapport à 3.0.1). | https://github.com/kubernetes-sigs/kind/issues/4208 | GitHub (tiers) | moyenne | commande
   - Container : `run, create, start, stop, kill, rm, exec, attach, logs, list, inspect, stats, prune, export`
   - Image : `build, pull, push, load, save, import, tag, list, inspect, rm, prune`
   - Network : `create, list, inspect, rm, prune`
   - Volume : `create, list, inspect, rm, prune`
   - Autres : `session {list, run, shell, enter, terminate}`, `registry {login, logout}`, `settings`, `system`, `version`
   - Selon l'issue, `exec` n'accepte que `-i -t -e -u -w -d` ; `inspect` et `list` renvoient du JSON brut sans `--format`.
8. Nouveautés GA / notes 3.0.x. | https://github.com/microsoft/WSL/releases | GitHub, Microsoft | 2026-09-29 | moyenne-haute (page résumée) | commande/option
   - Nouvelles commandes : `wslc events`, `wslc container restart`, `wslc network connect`, `wslc network disconnect`, `wslc system info`.
   - Options « parité Docker » ajoutées : `pull --all-tags`, `image list --digests`, `image list --all`, `inspect --size`, `container list --size`, `container logs --details`, `push --all-tags`, `cp --follow-link`, `--quiet` sur `image load`/`image push`/`container cp`, `wslc version --format`.
   - Autres lignes : healthchecks, UDP et IPv6 pour les ports exposés, correctifs (structure de sous-répertoires perdue sur volume VHD ; fichiers appartenant à root sur bind mounts virtiofs ; collision de passerelle de repli).
9. Blog GA. | https://blogs.windows.com/windowsdeveloper/2026/09/29/wsl-containers-now-generally-available/ | Windows Developer Blog | 2026-09-29 | moyenne-haute (résumé) | commande/option
   - `wslc.exe` = CLI ; `container.exe` = alias intégré.
   - Nouveautés GA : `wslc container restart`, `wslc container cp` (transfert d'archive tar), `wslc system info`, `wslc network connect|disconnect`, `wslc network create` (options de pilote), `wslc events`.
   - `create` et `run` : `--stop-timeout` (`-1` = infini) ; `--mount` supporté ; healthchecks ; chemin de stockage de la session `wslc` par défaut configurable.
   - Entreprise : contrôles d'accès Intune, liste d'autorisation de registres, plugin Defender for Endpoint (le code d'erreur `WSLC_E_REGISTRY_BLOCKED_BY_POLICY` vient de boxofcables, pas de ce blog).
10. **Compose non implémenté.** « Our top feature request for WSLc is adding compose support », objectif : `wsl compose up` sur un `compose.yaml` existant. Absent au GA. | blog GA | Windows Developer Blog | 2026-09-29 | moyenne-haute | limitation
11. Exemples de la préversion (juin 2026), PÉRIMÉS (WSL 2.9.3). | https://devblogs.microsoft.com/commandline/wsl-container-is-now-available-for-public-preview/ | Windows Command Line blog | 2026-06-29 | moyenne | commande
```powershell
wslc run -d --name=webtop -e PUID=1000 -e PGID=1000 -e TZ=Etc/UTC -p 3000:3000 -p 3001:3001 lscr.io/linuxserver/webtop:ubuntu-kde
wslc run --rm --gpus all pytorch/pytorch:2.5.1-cuda12.4-cudnn9-runtime python -c "import torch; print(torch.cuda.is_available())"
wslc run --rm -it -v /dev/ttyS4:/dev/ttyUSB0 alpine
wslc.exe system session terminate
wslc --version
```
Ce blog utilise `wslc --version` alors que le tutoriel Learn utilise `wslc version` : à vérifier sur 3.0.1.
12. Référence tierce, formes vues dans un billet de préversion. | https://boxofcables.dev/wslc-a-native-linux-container-runtime-for-windows | boxofcables.dev | 2026-06-02 | basse-moyenne, PÉRIMÉ (ère WSL 2.9.0) | commande
   - Formes courtes : `wslc run ubuntu:24.04`, `wslc pull postgres:16`, `wslc logs`, `wslc stop`, `wslc rm`, `wslc session shell`.
   - Options : `image list --filter`, `logs --since --until --timestamps`.
   - Valeurs par défaut rapportées (aussi dans l'issue kind) : VM de 2 vCPU, 2000 Mo de RAM, VHDX dynamique de 32 Go.
   - Le billet ne montre aucune sortie réelle.
13. Options refusées (wslc 2.9.13) : `wslc container run` rejette `--pids-limit`, `--cap-add`, `--cap-drop`, `--security-opt`, `--privileged`, `--read-only` ; accepte `--memory`, `--cpus`, `--ulimit`. | extraits de recherche GitHub (dépôts tiers, ex. https://github.com/sokolaidev/maf-extensions/pull/1474) | GitHub | ~sept. 2026 | basse-moyenne (extrait seul, préversion, non retesté sur 3.0.1) | limitation/option
14. Pas de `--device` ni de `--privileged` (passage USB). | blog juin 2026 et issue kind | moyenne, PÉRIMÉ | limitation

## B. Intégration et comportement

15. Publier des ports et y accéder depuis Windows : `-p hôte:conteneur` et `curl localhost:8080` depuis PowerShell fonctionnent ; `http://localhost:8000/` fonctionne depuis un navigateur Windows. | tutoriel | Microsoft Learn | 2026-09-29 | haute | réseau
```powershell
wslc run -d --rm -p 8080:80 --name web nginx
curl localhost:8080
```
L'issue kind liste `-p` avec IP hôte et UDP. 3.0.1 ajoute UDP et IPv6 pour les ports exposés (notes de version). Moyenne.
16. Variables d'environnement : `-e CLE=VALEUR` (exemple de juin 2026, point 11) ; `exec` accepte aussi `-e`. Non présent dans le texte Learn GA. | moyenne | réseau/commande
17. Bind mount depuis un chemin Windows (blog d'architecture) : | https://devblogs.microsoft.com/commandline/wslc-architecture-deep-dive/ et https://www.comss.ru/page.php?id=22219 (miroir russe) | Windows Command Line blog, 2026-09-29 | moyenne-haute | volume
```
wslc container run -v C:\Windows\System32\drivers\etc:/volume -it debian:latest ls /volume
```
Syntaxe : `-v C:\chemin:/chemin_conteneur` (lettre de lecteur, antislashs côté hôte). Même forme dans boxofcables : `-v C:\Users\me\project:/workspace ubuntu:24.04`. Règles de guillemets pour chemins avec espaces : non trouvées.
18. Volumes nommés et persistance (VHD) : | même blog d'architecture | moyenne-haute | volume
```
wslc volume create --driver vhd -o SizeBytes=200000000 my-volume
wslc container run -v my-volume:/volume -it debian:latest findmnt /volume
```
   - Types de volumes : VHD de session (`%AppData%\Local\wslc\sessions`), montages virtiofs (« environ deux fois plus rapides » que Plan9), volumes VHD (système de fichiers Linux natif, taille imposée), volumes nommés partagés par nom entre conteneurs.
   - Sous-commandes `volume` : `create, list, inspect, rm, prune`. virtiofs plus rapide que Plan9 mais pas identique à la vitesse native de WSL.
   - Correctifs de notes de version : bind mounts virtiofs appartenant à root et sous-répertoires perdus sur volumes VHD, corrigés en 3.0.x (donc présents en 2.9.x). Moyenne.
19. `--mount` supporté sur `run` et `create` (blog GA). Syntaxe exacte non récupérée. | moyenne | volume
20. Réseau entre conteneurs et DNS :
   - Le composant « consomme » gère DNS, routage UDP/TCP et mappage de ports, au nom de l'utilisateur de la session.
   - boxofcables (basse confiance, PÉRIMÉ) : modes réseau None, NAT et « virtio proxy », plus bridge, host, none et jonction à l'espace de noms d'un autre conteneur.
   - Côté CLI : `wslc network create|list|inspect|rm|prune`, et `network connect|disconnect` depuis 3.0.x. `network create` accepte `--driver --subnet --gateway --internal --label -o` ; `run` accepte `--net`. `--hostname` et `--tmpfs` existent (issue kind).
   - **NON trouvé** : aucun exemple documenté de DNS par nom de conteneur sur un réseau défini par l'utilisateur. Comportement non vérifié : risque pour l'exposé. | basse | réseau
21. Politique de redémarrage : NON documentée dans les pages lues. L'issue kind (préversion) signale l'absence de `--restart`. 3.0.1 n'ajoute que la commande ponctuelle `wslc container restart`, pas une politique. À tester : `wslc run --help | findstr restart`. | moyenne (absence), PÉRIMÉ | limitation
22. Limites de ressources : `--memory` et `--cpus` existent sur `wslc container run` (extraits GitHub, WSL 2.9.13). VM par défaut 2 vCPU / 2 Go ; CPU et mémoire de session réglés via l'API `SessionSettings` (`CpuCount = 4, MemoryMB = 4096` dans l'exemple Learn). Syntaxe des valeurs (`512m`) non vérifiée. | basse-moyenne | option
23. GPU : `--gpus all` existe. En 3.0.1.0, `--gpus all` fait échouer la création du conteneur sur les images Alpine/musl (le hook GPU traite le code retour de `ldconfig` comme fatal). Debian/Ubuntu fonctionnent. | https://github.com/microsoft/WSL/issues/41791 | GitHub microsoft/WSL, ouverte le 2026-10-03 | moyenne-haute (page résumée) | limitation
24. Là où ça coince :
   - Note Learn : garder le code du projet sur le système de fichiers Linux, car les fichiers Windows ralentissent « significativement » les outils Linux. | tutoriel | haute
   - Rapport de préversion : conteneurs ne joignant pas certains services de l'hôte selon la configuration réseau ; « collision de passerelle de repli » corrigée en 3.0.x.
   - Autres issues sur 3.0.1 : #41777 (nouvelles sessions en échec avec WSAENOBUFS après la reprise de Modern Standby), #41653. Titres seuls, non lus. Basse.
   - Préversion (PÉRIMÉ) : le blog dit que la CLI n'était « pas prête pour la production », l'API l'étant.

## Pistes à creuser
- Capturer le vrai texte de `wslc --help` et `wslc run --help` en 3.0.1 (aucune page de référence CLI primaire trouvée).
- Notes de version 3.0.1 en entier ; dossier docs de microsoft/WSL.
- Issue #41653 et le suivi avec `label:wslc` : réseau, DNS, redémarrage.
- https://wsl.dev/api-reference/ et https://aka.ms/wslc-samples.
- microsoft/mxc `docs/wsl/wsl-container-getting-started.md` (SDK WSLC ; WSL 2.9.9+).
- https://devblogs.microsoft.com/commandline/ pour les suites 3.0.x.

## Cherché et non trouvé
- Page de référence CLI officielle listant toutes les options (ni learn.microsoft.com/windows/wsl/reference ni wsl.dev).
- Sortie brute de `wslc --help` ou `wslc run --help`.
- Politique de redémarrage, DNS par nom sur réseau utilisateur, syntaxe exacte de `--mount`, `--memory` et `--cpus`.
- Prise en charge de `buildx`, `docker compose`, Dockerfile (vs Containerfile) : seul le Containerfile du Learn et la feuille de route compose.
- Options de `registry login`.
- Transcriptions de sorties de commandes réelles dans les sources primaires.
- Options de `save` et `load` au-delà de `--quiet` sur `load`.
