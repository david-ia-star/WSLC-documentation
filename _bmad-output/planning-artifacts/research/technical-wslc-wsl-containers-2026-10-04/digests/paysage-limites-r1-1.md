# Digest — paysage, maturité, limites (round 1, assistant 1)

Accès : 2026-10-04. Outils : WebFetch/WebSearch. Les pages GitHub et certaines pages sont résumées par un petit modèle : revérifier avant de citer dans l'exposé.

**Point clé : la piste de départ est périmée.** WSLC est en disponibilité générale (GA) depuis 2026-09-29 avec WSL 3.0.1. `--pre-release` n'est plus nécessaire (valable du 2026-06-29 au 2026-09-28).

## Constats
Format : constat | source | éditeur | date de publication | confiance | classe

1. WSLC = deux parties : la CLI intégrée `wslc.exe` et une API de conteneurs WSL pour les développeurs d'applications Windows. | https://learn.microsoft.com/windows/wsl/wsl-container | Microsoft Learn | 2026-09-29 | haute | autre
2. Prérequis : « WSL version 2.9.3 or higher ». Mise à jour : `wsl --update`. Version : `wsl --version`. Binaire : `wslc version`. Test : `wslc run --rm hello-world`. | https://learn.microsoft.com/windows/wsl/tutorials/wsl-containers | Microsoft Learn | 2026-09-29 | haute | version
3. GA : WSL 3.0.1, publiée le 2026-09-29, « Latest » sur GitHub. Note : « WSL containers are generally available. » | https://github.com/microsoft/WSL/releases/tag/3.0.1 | Microsoft (GitHub) | 2026-09-29 | haute (page résumée) | version
4. Annonce GA. Fonctions citées : redémarrage de conteneur, `wslc container cp`, `wslc system info`, `wslc network connect` / `wslc network disconnect`, `wslc events`, montages, chemins de stockage configurables, accès fichiers Windows jusqu'à 2x plus rapide, nouveau mode réseau `consomme`, alias `container.exe`. Pas de comparaison avec Docker Desktop ou Podman. | https://blogs.windows.com/windowsdeveloper/2026/09/29/wsl-containers-now-generally-available/ | Microsoft Windows Developer Blog | 2026-09-29 | haute | fonction
5. Chronologie : annonce Build 2026 le 2026-06-02 (extrait de recherche, non vérifié en source primaire) ; préversion publique le 2026-06-29 avec WSL 2.9.3 (`wsl --update --pre-release`) ; GA visée « Fall 2026 », livrée le 2026-09-29. | https://devblogs.microsoft.com/commandline/wsl-container-is-now-available-for-public-preview/ | Microsoft DevBlogs | 2026-06-29 | haute (06-29 et GA) ; moyenne (06-02) | version
6. Architecture : chaque session WSLC tourne dans sa propre VM dédiée (créée via HCS). Un processus par utilisateur, `wslcsession.exe`, exécute toutes les opérations de session. VHD de session dans `%AppData%\Local\wslc\sessions`. Partage de chemins Windows via virtiofs. Volumes VHD = système de fichiers Linux natif : `wslc volume create --driver vhd -o SizeBytes=200000000 my-volume`. Réseau : gestionnaire « Consommé » en processus utilisateur (DNS, TCP/UDP, mappage de ports). | https://devblogs.microsoft.com/commandline/wslc-architecture-deep-dive/ | Microsoft DevBlogs | 2026-09-29 | haute (page résumée) | fonction
7. Runtime sous-jacent (containerd ou custom) : NON établi. Ne rien affirmer. | même source que 6 | — | — | — | autre
8. Les conteneurs utilisent le noyau Linux de WSL 2 (`wslc exec django uname`). | tutoriel (voir 2) | Microsoft Learn | 2026-09-29 | haute | fonction
9. Commandes vérifiées dans les pages Learn (à citer telles quelles) :
   - `wslc run --rm -it ubuntu:latest bash -c "echo Hello world from WSL container!"`
   - `wslc image ls` / `wslc image list`
   - `wslc run -it --rm -d -p 8080:80 --name web nginx`
   - `wslc run -d --rm -p 8080:80 --name web nginx`
   - `wslc container ps` / `wslc container list` / `wslc container list --all`
   - `wslc container stop web`
   - `wslc exec web cat /etc/os-release`
   - `wslc stats`
   - `wslc --help` / `wslc <COMMAND> --help`
   - `wslc build -t helloworld-django .` (fichier `Containerfile`)
   - `wslc container logs django`
   - `wslc container inspect <container-id>` / `wslc image inspect <image>`
   - `wslc container prune` / `wslc image prune`
   - GPU : `--gpus all` (blog préversion, issue #41791)
   | https://learn.microsoft.com/windows/wsl/tutorials/wsl-containers et /wsl-container | Microsoft Learn | 2026-09-29 | haute | commande
10. `wslc build` est supporté. Issue #41735 « Build support for --secret flag » fermée le 2026-10-01 : des options de build restent à compléter. | tutoriel + issues GitHub | Microsoft Learn / GitHub | 2026-09-29 / 2026-10-01 | moyenne | fonction
11. Compose N'EST PAS supporté. Annonce GA : « Our top feature request for WSLc is adding compose support, and this will be our focus for our next iterations ». Issue #41754 (ouverte, 2026-10-01) : Visual Studio affiche « Docker Compose is not supported with the WSL container runtime (wslc). » | https://blogs.windows.com/windowsdeveloper/2026/09/29/wsl-containers-now-generally-available/ ; https://github.com/microsoft/WSL/issues/41754 | Microsoft / GitHub | 2026-09-29 / 2026-10-01 | haute (manque de fonction) ; moyenne (détails de l'issue) | limitation
12. Limites de l'époque préversion : USB non supporté ; les conteneurs ne joignaient pas les services de l'hôte Windows. Non revérifié au GA (périmé). Issue #41769 : `host.wslc.internal` non résolu dans les conteneurs musl (Alpine). | https://devblogs.microsoft.com/commandline/wsl-container-is-now-available-for-public-preview/ ; issues | Microsoft DevBlogs / GitHub | 2026-06-29 | moyenne (périmé) | limitation
13. Issues WSLC ouvertes au 2026-10-03/04 (178 résultats, page 1 lue seulement ; liste résumée, à revérifier) : #41794 (proxy hôte / miroirs de registre), #41793 (`container cp` échoue sur les liens symboliques), #41791 (`--gpus all` plante sur Alpine/musl), #41740 (`wslc run` renvoie ERROR_TIMEOUT), #41769 (`host.wslc.internal` sur musl), #41748 (/dev/dri non exposé), #41754 (compose dans VS 2026). | https://github.com/microsoft/WSL/issues?q=is%3Aissue+wslc+sort%3Aupdated-desc | GitHub (microsoft/WSL) | 2026-10-01 à 2026-10-04 | moyenne | limitation
14. Cadence de versions : 3.0.1 (29 sept, Latest) ; 2.9.13 (25 sept, pré-version : `wslc events`, `--all-tags`, `--digests`) ; 2.9.12 (14 sept) ; 2.9.11 (8 sept) ; 2.9.10 (4 sept : system info, healthcheck) ; 2.9.9 (25 août) ; 2.9.8 (24 août : network connect/disconnect, `container cp`, UDP/IPv6). Ligne stable 2.7.x (2.7.14 le 11 sept). | https://github.com/microsoft/WSL/releases | Microsoft (GitHub) | jusqu'au 2026-09-29 | moyenne-haute (tableau résumé) | version
15. Contrôles entreprise : bascule Intune « Allow WSL containers access », liste d'autorisation de registres, surveillance Defender for Endpoint, GPO/ADMX en préversion. | blogs GA et préversion | Microsoft | 2026-09-29 | haute | fonction
16. API développeurs : paquet NuGet `Microsoft.WSL.Containers` (`dotnet add package Microsoft.WSL.Containers`, `using Microsoft.WSL.Containers;`). Projection C++/WinRT en préversion, sujette à changements incompatibles. Objets : `WslcService`, `Session`, `Container`, `Process`. Flux : `WslcService.GetMissingComponents()`, `new Session(new SessionSettings("MyApp", @"C:\WslcData") { CpuCount = 4, MemoryMB = 4096 })`, `session.Start()`, `session.PullImageAsync(...)`, `session.CreateContainer(...)`, `container.Start()`, `container.Stop(Signal.SIGTERM, ...)`, `session.Terminate()`. Références non lues : https://wsl.dev/api-reference/ et https://aka.ms/wslc-samples. | https://learn.microsoft.com/windows/wsl/wsl-container | Microsoft Learn | 2026-09-29 | haute | fonction
17. Comparaison avec Docker Desktop / Podman : AUCUNE comparaison officielle trouvée. Seul un billet d'opinion d'avant lancement (falcao.org, 2026-06-20), périmé et peu fiable. | http://falcao.org/posts/wsl-containers-what-changes-for-wsl2-users/ | blog personnel | 2026-06-20 | basse | autre
18. Dev containers VS Code : extension v0.462.0-pre-release ou plus récente (préversion). Non revérifié au GA. | blog préversion | Microsoft DevBlogs | 2026-06-29 | moyenne (périmé) | compatibilité
19. Secondaire : https://itsfoss.com/news/wslc-general-availability/ (2026-09-30) reprend les sources primaires. | It's FOSS | 2026-09-30 | moyenne | autre

## Pistes à creuser
- Issues GitHub étiquetées wslc, pages 2 à 8 : réseau (DNS, MTU sous VPN, relais de ports), `container cp`, GPU.
- Notes de version complètes 3.0.1 vs 2.9.13 ; quel canal est « stable » ensuite.
- https://wsl.dev/api-reference/ et https://aka.ms/wslc-samples.
- Référence CLI complète (`wslc --help`) : aucune page Learn dédiée trouvée.
- Prérequis Windows (build, virtualisation) : non trouvés ; voir https://learn.microsoft.com/windows/wsl/install.
- Code source `src/windows/wslc` pour établir le runtime.
- L'alias `container.exe` au GA (cité surtout en préversion).
- Suite de l'issue #41754 côté Visual Studio.

## Cherché et non trouvé
- Runtime sous-jacent (containerd, runc ou custom).
- Comparaison officielle ou testée avec Docker Desktop, Podman, Rancher Desktop.
- Build Windows ou SKU requis ; exigence Hyper-V.
- Liste précise des limites réseau au GA.
- Statut de résolution des issues ouvertes.
- Confirmation primaire de la date Build du 2026-06-02.
