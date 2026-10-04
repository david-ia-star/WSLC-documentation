# Revue de `docs/wslc.md` (bmad-review, 2026-10-04)

Lenses exécutées : adversarial (22 constats), edge-case-hunter (44), structure (20 recommandations), prose (20 corrections principales et 14 mineures). Verification-gap non exécutée (elle ne vise que le code). Aucune gravité n'est attribuée, par conception. Les recoupements entre lenses sont signalés plutôt que dédoublonnés.

## Recoupements entre lenses (signal fort)

| Sujet | Lenses | Idée commune |
|---|---|---|
| Attente de démarrage avant la vérification | edge-case, adversarial | `curl.exe`, `psql`, `redis-cli`, `/health` lancés juste après `run -d` peuvent échouer le temps que le service démarre |
| Nettoyage des relances | edge-case, adversarial | Pas de `container rm`, `volume rm`, `image pull` dans le tableau ; un nom ou un volume de la répétition casse la relance (mot de passe Grafana déjà changé, base PostgreSQL d'une autre version, assistant Gitea déjà passé) |
| Tags non vérifiés | edge-case, adversarial | Plusieurs tags n'ont pas été confirmés sur le registre ; Mailpit est non épinglé ; l'annexe garde des tags flottants |
| Gitea : URL et port SSH | edge-case, adversarial | L'assistant propose `localhost:3000` et le port 22 alors que l'hôte publie 3001 et 2222 |
| Commande de test de Mailpit et de SeaweedFS | edge-case, adversarial | `mail.txt` sans exemple de contenu ; client `aws` non requis ni configuré (région, identifiants) |
| Plan B de la démo | edge-case, adversarial | Aucune règle si un service met plus de 60 s, si un téléchargement échoue ou si le réseau de la salle est lent |
| SonarQube dans le corps du guide | adversarial, structure | Un service qui peut ne pas démarrer figure parmi les services principaux |
| Notes de l'orateur dans le guide du lecteur | adversarial, structure | Checklist, déroulé et choix en attente se mêlent au contenu rejouable |
| « 2.9.3 ou plus » face à « 3.0.1 » | adversarial, structure, prose | Deux énoncés différents du minimum de version |
| « seau » pour « bucket » | adversarial, prose | Le terme « bucket » est le terme courant ; la commande utilise `my-bucket` |

Note : « 2.9.13 » signalé comme faute de frappe possible par la lens prose est une vraie version (préversion du 25 septembre 2026, relevée dans la recherche). Ce n'est pas une erreur.

## Lens adversarial

| # | Où | Problème | Correction proposée |
|---|---|---|---|
| 1 | §5 / §8 | Aucun plan B si `wslc run` échoue en direct (ERROR_TIMEOUT #41740, téléchargement, réseau) | Sous-section « Plan B » : télécharger toutes les images la veille, enregistrer les démos, bascule après 60 s |
| 2 | §2 | Prérequis Windows inconnus ; « 2.9.3 ou plus » contre version de référence 3.0.1 ; pas de sortie attendue ni de cas « `wslc` introuvable » | Donner le minimum exact, la sortie de `wsl --version`, la marche à suivre si `wslc` manque |
| 3 | §3 | Correspondances sans marque « vérifié » ; lignes manquantes (pull, rm, start, volume, network, cp, tag, push) ; volumes qui s'accumulent | Colonne Vérifié / supposé ; lignes manquantes ; bloc de nettoyage |
| 4 | §4 | `curl.exe` juste après `run -d` ; l'effet de `--rm` n'est pas démontré | `Start-Sleep 2` ; `wslc container list --all` après l'arrêt ; sortie attendue |
| 5 | §5 PostgreSQL | Volume réutilisé entre versions ; mot de passe `secret` ; port publié sur toutes les interfaces ; `psql` avant la fin de l'initialisation | Attente `pg_isready` ; avertissement sur `pgdata` ; `127.0.0.1:5432:5432` à tester |
| 6 | §5 sécurité | Services publiés avec identifiants faibles (admin/admin, secret, guest/guest) ; invite du pare-feu Windows possible en direct | Note « Sécurité des démos » ; liaison sur 127.0.0.1 à tester ; répondre à l'invite avant l'exposé |
| 7 | §5 Gitea | URL de base et port SSH de l'assistant (3000 et 22) ne correspondent pas aux ports publiés (3001 et 2222) | Indiquer les champs à changer, ou `-e GITEA__server__ROOT_URL=... -e GITEA__server__SSH_PORT=2222` (à tester), ou retirer `-p 2222:22` |
| 8 | §5 Grafana | « corrige trois failles » sans lien ; « seul, Grafana montre… » non vérifié ; l'écran de changement de mot de passe prend du temps | Lien vers les notes de version ; démo de 60 s scénarisée ; `GF_SECURITY_ADMIN_PASSWORD` |
| 9 | §3 / §6 | « existent dans `wslc run` » affirmé sans réserve alors que §6 dit « non vérifié » ; `--env-file` apparaît seulement dans la checklist | Aligner §3 sur §6 ; expliquer ou retirer `--env-file` ; marquer « (préversion) » |
| 10 | §8 déroulé | 16 à 19 minutes sans Q&R ni conclusion ; ordre des trois démos non fixé ; pas de réponse prête à « pourquoi pas Docker Desktop ? » | Ajouter conclusion et questions ; budget par démo ; une phrase de réponse |
| 11 | §5 SonarQube | Dans le corps du guide alors qu'il peut ne pas démarrer ; pas de volume ni de limite mémoire | Le déplacer en annexe tant que le test n'est pas fait |
| 12 | §5 Mailpit, SeaweedFS | Commandes de test « sans source » ; `mail.txt` sans exemple ; client AWS non requis | Fournir `mail.txt` par here-string PowerShell ; préciser le client AWS |
| 13 | §5 tags | Tags cités depuis une seule vérification, jamais confirmés sur le registre | Script PowerShell `wslc image pull` sur tous les tags, lancé la veille |
| 14 | §5 Redis | Valkey et licence sans lien ; `--save 60 1` décrit mais absent de la commande | Citer la page de licence ; montrer la commande complète avec `--save` |
| 15 | §6 Compose | « deux sources » sans les nommer ; aucune solution de contournement | Nommer les sources ; paragraphe « Que faire en attendant ? » avec un script de démarrage/arrêt |
| 16 | §7 | « Aucun conflit » trop fort : nginx (8080) contre Keycloak (8080) s'il reste actif ; ports 9000/9001 et annexe en prose ; plages de ports réservées par Windows | Tout mettre dans le tableau ; vérifier `netsh interface ipv4 show excludedportrange protocol=tcp` |
| 17 | §8 / en-tête | Étiquettes « testé » manuelles sans journal ; notes de l'orateur mêlées au guide | Séparer guide du lecteur et notes de l'orateur ; garder un `TESTS.md` (date, build Windows, version WSL, résultat) |
| 18 | §1 / §6 | VM : mémoire et CPU par défaut non indiqués ; aucun exemple de montage de dossier du projet | Un exemple `-v "${PWD}:/app"` à tester ; une ligne sur la mémoire de la VM et `.wslconfig` |
| 19 | §5 Meilisearch | Affirmation sur des tags « en retard » non datée ; `/health` sans attente | Dater ou retirer ; attente ; sortie attendue à vérifier |
| 20 | Annexe | Tags flottants contre la règle d'épinglage ; `<clé>` et `<secret>` non collables dans PowerShell (`<` redirige) ; mot de passe en clair en ligne de commande | Épingler ou signaler ; valeurs de démonstration ; ne pas exposer le mot de passe |
| 21 | Sources | Numéros d'issues sans lien ; chemin interne `_bmad-output/...` | Un lien par issue ; retirer le chemin ; revérifier §6 la veille |
| 22 | Général | Vocabulaire hybride (« Allow All », « start-dev », « seau ») ; pas de schéma d'architecture ; précautions répétées partout | « bucket » partout ; un schéma ; un seul tableau des points non vérifiés |

## Lens edge-case-hunter

| # | Où | Condition qui expose le problème | Correction proposée |
|---|---|---|---|
| 1 | §4 nginx | Port 8080 déjà pris sur Windows | `netstat -ano \| findstr ":8080"` ; `-p 8081:80` en repli |
| 2 | §4 nginx | Relance après arrêt anormal : le nom `web` existe encore | `wslc container stop web` puis `wslc container rm web` (vérifier avec `--help`) |
| 3 | §4 nginx | Image non en cache : premier téléchargement lent en direct | `wslc image pull` pendant la vérification préalable |
| 4 | §4 nginx | `curl.exe` avant que nginx accepte les connexions | `Start-Sleep 2` ou boucle de relance |
| 5 | §2 | `wsl --update` sans droits d'administrateur, WSL plus ancien que 2.9.3, virtualisation désactivée | Vérifier avant l'exposé ; PowerShell en administrateur |
| 6 | §2 | Terminal ouvert avant la mise à jour : `wslc` non reconnu | Ouvrir une nouvelle session PowerShell ; repli `wslc.exe` |
| 7 | §5 Mailpit | Relance : le conteneur `mailpit` arrêté existe encore (pas de `--rm`) | `stop` puis `rm`, ou ajouter `--rm` |
| 8 | §5 Mailpit | `axllent/mailpit` non épinglé alors que le texte cite v1.31.4 | `axllent/mailpit:v1.31.4` après vérification du format du tag |
| 9 | §5 Mailpit | `mail.txt` absent, vide ou sans en-têtes ni CRLF | Here-string PowerShell + `Set-Content` |
| 10 | §5 Grafana | Volume nommé créé propriétaire root alors que Grafana tourne sous un autre UID | Vérifier les droits : `wslc run --rm -v grafana-data:/var/lib/grafana grafana/grafana:13.2.3 ls -ld /var/lib/grafana` |
| 11 | §5 Grafana | Relance avec un volume existant : mot de passe déjà changé | `wslc volume rm grafana-data` avant la répétition (vérifier la commande) |
| 12 | §5 Grafana | Port 3000 occupé sur Windows | `netstat` ; `-p 3002:3000` |
| 13 | §5 Gitea | Tag 28.0.0 ou nom d'organisation non vérifié ; droits sur `/data` | Télécharger l'image la veille ; vérifier l'écriture sur `/data` |
| 14 | §5 Gitea | L'assistant propose ROOT_URL et SSH sur 3000 et 22 | Champs de l'assistant ou `GITEA__server__ROOT_URL` |
| 15 | §5 Gitea | Port 2222 ou 22 en conflit avec OpenSSH de Windows ou sshd de WSL | `netstat` ; retirer `-p 2222:22` si le SSH n'est pas montré |
| 16 | §5 PostgreSQL | Volume restant d'une autre version majeure ou d'une initialisation échouée | `wslc volume rm pgdata` avant la relance |
| 17 | §5 PostgreSQL | `psql` avant la fin du démarrage (premier démarrage avec redémarrage interne) | Boucle `pg_isready` |
| 18 | §5 PostgreSQL | Guillemets doubles imbriqués et point-virgule dans `-c "select version();"` | `-c 'select version();'` (vérifier PS 5.1 et PS 7) |
| 19 | §5 PostgreSQL | Port 5432 pris par un PostgreSQL local | `netstat` ; `-p 5433:5432` |
| 20 | §5 Redis | `--save 60 1` mentionné sans commande complète ; `ping` en concurrence avec le démarrage | Commande complète avec `redis-server --save 60 1` |
| 21 | §5 Solr | Volume `/var/solr` propriétaire root alors que Solr tourne sous 8983 ; relance avec un cœur existant | Vérifier les droits ; supprimer le volume avant relance |
| 22 | §5 Solr | Premier démarrage lent (JVM) ; tag non vérifié | Télécharger d'avance ; attendre la réponse de `/solr/` |
| 23 | §5 SonarQube | Sans `vm.max_map_count` : échec des contrôles d'amorçage d'Elasticsearch | `-e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true` en repli ; lire `wslc container logs sonarqube` |
| 24 | §5 SonarQube | Premier démarrage de plusieurs minutes ; conflit du port 9000 | Attendre `http://localhost:9000/api/system/status` = UP |
| 25 | §5 SonarQube | Concurrence mémoire avec Grafana et Gitea | Arrêter les démos précédentes |
| 26 | §5 Meilisearch | `/health` avant que le service soit prêt | `Start-Sleep 2` ou relance |
| 27 | §5 Meilisearch | Volume créé avec une autre version : base incompatible | `wslc volume rm meili-data` avant relance avec un autre tag |
| 28 | §5 Prometheus | Tag v3.15.0 non vérifié ; auto-collecte non confirmée | Vérifier le tag et la page Targets avant la démo |
| 29 | §5 SeaweedFS | Ports et droits du volume ; tag et commande par défaut non confirmés | Télécharger d'avance ; `wslc container logs seaweed` |
| 30 | §5 SeaweedFS | Client `aws` absent ou sans région ni identifiants | Variables d'environnement PowerShell + `aws s3 ls --endpoint-url ...` |
| 31 | §7 | Keycloak (8080) contre nginx (8080) si `web` tourne encore | Arrêter `web` ou déplacer Keycloak |
| 32 | §7 | « Aucun conflit » sans contrôle côté hôte | `Get-NetTCPConnection -State Listen` avant la démo |
| 33 | §8 | La checklist ne couvre ni le téléchargement de toutes les images ni le nettoyage | Ajouter : téléchargement préalable, `container prune`, suppression des volumes, ports libres |
| 34 | §8 | Réseau lent ou proxy de la salle | Toutes les images en local ; essai sans réseau |
| 35 | §6 | Variables proxy de l'hôte non reprises (#41794) : pas de téléchargement derrière un proxy d'entreprise | Passer `-e HTTP_PROXY` ou télécharger hors proxy |
| 36 | §8 déroulé | Aucune marge pour un échec ou les questions | Règle de repli : si un service n'est pas prêt en 60 s, passer à l'annexe ou à une sortie enregistrée |
| 37 | Annexe | Placeholders `<clé>` et `<secret>` ; tags flottants | Valeurs concrètes entre guillemets ; tags épinglés |
| 38 | Annexe MongoDB | Guillemets et accolades de `--eval` dans PowerShell | `--eval 'db.runCommand({ping:1})'` |
| 39 | Annexe MariaDB | `-psecret` pendant l'initialisation ; client `mariadb` supposé | Attendre ; essayer `mysql` si absent |
| 40 | Annexe Keycloak | Port 8080 contre nginx ; mémoire de la VM | Arrêter `web` ; vérifier la mémoire de la VM |
| 41 | Annexe RustFS | Volume propriétaire root alors que le conteneur tourne sous `10001:10001` | Droits ou lancement sans volume |
| 42 | Annexe Adminer | Résolution du nom de conteneur non confirmée et pas de valeur de repli | Adresse de l'hôte (échoue sous Alpine, #41769) |
| 43 | §2 | `wsl --update` peut ne pas installer la version stable selon le canal | Indiquer la sortie attendue de `wsl --version` et un repli (`--web-download`) |
| 44 | §3 | Pas de lignes `container rm`, `volume list/rm`, `image pull`, `container start` | Ajouter les lignes avec la syntaxe vérifiée |

## Lens structure

Modèle retenu : tutoriel linéaire (§1 à 5) avec sections de référence (§3, §6, §7, annexe). 2 512 mots ; le §5 pèse environ 40 % pour trois démos.

| Opération | Cible | Effet |
|---|---|---|
| MOVE + MERGE | Les sept services « à lire » du §5 et l'annexe fusionnent en un seul tableau de référence (Service, Commande, Interface, Piège, Statut) | Le corps ne garde que les trois démos en direct |
| MOVE | §8 (checklist et déroulé) vers une section « Notes de l'orateur (hors rejeu) » en fin de fichier | Environ 228 mots sortent du chemin de lecture |
| MOVE | Déroulé minuté en tête (« Au programme ») | Le lecteur voit le plan d'abord |
| CONDENSE | Introduction : l'avertissement « rien n'est testé » en bandeau | Environ 15 mots en moins |
| CONDENSE | « Statut : à tester » répété 14 fois : une phrase par section, une étiquette individuelle seulement pour les exceptions | Environ 35 mots en moins |
| MOVE | Remarques de provenance des puces vers un encadré « À vérifier avant de rejouer » | Environ 50 mots en moins |
| MOVE + CONDENSE | Les limites qui touchent des développeurs Docker (compose, `--restart`, DNS) juste après le tableau du §3 | L'information qui conditionne l'adoption arrive plus tôt |
| MOVE | Lignes « Runtime sous-jacent » et « Comparaison Docker Desktop ou Podman » vers une liste « Questions sans réponse » | Ce sont des lacunes de documentation, pas des limites |
| MERGE | « Réglages hôte » (§6), puce SonarQube et case du §8 | Un seul endroit pour le même constat |
| MOVE | « Fichiers Windows » du §6 vers le §1, près de virtiofs | Explique le choix des volumes nommés |
| MOVE + MERGE | Plan de ports en tête du §5, avec les ports de l'annexe en lignes | Supprime la phrase finale redondante |
| CONDENSE | Dates répétées (4 octobre, 29 septembre) | Une fois en introduction |
| CUT ou MOVE | Détails secondaires des services non joués (licences, failles, variantes) | Environ 60 mots |
| CONDENSE | Gabarit constant par service : Interface, Test, Piège | Lecture en survol |
| CUT | Dernière ligne des Sources (chemin interne) | Hors périmètre du lecteur |
| PRESERVE | §1 dernier paragraphe ; §3 tableau Docker vers `wslc` ; §2 et §4 | À ne pas toucher |

Questions : quelles lignes du §6 sont dites à voix haute dans les 3 minutes prévues ? « 2.9.3 ou plus » contre « 3.0.1 » : un seul énoncé ?

Réduction nette estimée : environ 250 à 270 mots (10 %). Environ 530 mots (services non joués) et 228 mots (notes de l'orateur) sortent du chemin de lecture principal. Le corps du guide tomberait à environ 1 700 mots, dont environ 900 de contenu montré.

## Lens prose

| Où | Original | Révisé | Raison |
|---|---|---|---|
| §5 Redis | « qui n'a pas d'image officielle Docker sous le nom `valkey` » | « Valkey (`valkey/valkey:9`), image publiée par le projet et absente des images officielles de Docker » | La proposition se contredit en apparence |
| §6 Compose | « Microsoft en fait la première demande… » | « Microsoft la classe comme première demande… » | « en fait » se lit comme « en réalité » |
| §5 SonarQube et §6 | « côté hôte » ; « Réglages hôte » | « ces réglages noyau dans la machine virtuelle » ; « Réglages noyau de la machine virtuelle » | « hôte » désigne Windows ailleurs |
| §2 et §5 | « pages lues » | « pages consultées » | Même formule que les Sources |
| §5 et §8 | « la veille » | « la veille de l'exposé » | Repère non défini dans le corps |
| §5 | « en direct » et « à lire » | « montrés en démonstration » et « seulement décrits dans ce document » | Contexte d'exposé inconnu du lecteur |
| §3 build | « ne sont pas vérifiés » | « La prise en charge … n'est pas vérifiée » | Accord sur le bon sujet |
| §5 et §8 Prometheus | « Dans Status, puis Targets… » | « Ouvrez **Status** > **Targets** pour vérifier… » | Notation des menus |
| §5 Prometheus | « Que l'image l'utilise par défaut n'est pas confirmé » | « On ne sait pas si l'image l'utilise par défaut » | Complétive trop lourde |
| §8 | « répond contre SeaweedFS » | « fonctionne avec SeaweedFS » | Calque de l'anglais |
| §8 | « ce que deviennent `--restart`… » | « indique si `--restart`… sont pris en charge » | Vague |
| §5 SeaweedFS | « seau » | « compartiment » ou « bucket » | Calque ; cohérence avec `S3_BUCKET` |
| §5 Gitea, Annexe | « correspondance », « remappé » | « redirection », « redirigé » | Anglicisme ; un seul mot |
| §5 SonarQube, Annexe Keycloak | « Ne pas lancer… », « Prévoir… » | « Ne le lancez pas… », « Prévoyez… » | Registre : impératif de vouvoiement |
| Annexe RabbitMQ | « Le statut des variables… est ambigu » | « La prise en charge des variables… est ambiguë » | « statut » est déjà pris |
| §1 | « CLI », « virtiofs », « VHD » | Développer à la première occurrence | Lecteur « expert accessible » |
| §1 | « date de la préversion » | « est périmée : elle date de la préversion » | Dire la conséquence |
| §5 SeaweedFS | « n'authentifie plus personne » | « il n'exige plus aucune authentification » | Contre-intuitif |
| §6 Fichiers Windows | « ralentissent », « Plan9 » | « L'accès aux fichiers… ralentit », « Plan 9 » | Sujet logique et graphie |
| §5 Solr | « plus sûr », « À tester. » | « plus prudent » ; supprimer l'étiquette redondante | « sûr » évoque la sécurité |

Quatorze corrections mineures (fragment d'ouverture du §4, « Lancé seul, Grafana… », « version 17 et les précédentes », glose d'AOF, « rejetées par la version 2.9.13 », « Plan 9 », uniformisation de « démo » et « démonstration », « Version 1.31.4 » sans « v » hors des tags, « La commande `curl.exe` n'est pas tirée des pages consultées », « Cette commande ne vient d'aucune source », « Prévoyez », « redirection », « revérifier » et « retester » à unifier, « 2.9.3 ou ultérieure »).

Préservé : vouvoiement, ton « expert accessible », absence d'humour et de tiret cadratin, étiquettes **testé** et **à tester**.

## Suite donnée à la revue

Lots A et B appliqués à `docs/wslc.md` le 2026-10-04 sur demande de David.

- **Lot A (correctifs) :** fonction `Wait-Http` et attente `pg_isready` ; lignes de nettoyage dans le tableau Docker vers `wslc` (marquées « préversion » faute de confirmation) ; bloc « Repartir de zéro » ; vérification des ports côté Windows ; exemple de `mail.txt` par here-string ; identifiants AWS ; consignes pour l'assistant de Gitea ; guillemets simples pour `psql` et `mongosh` ; `redis-server --save 60 1` dans la commande ; note de sécurité (127.0.0.1, pare-feu) ; exemple de montage de dossier ; corrections de prose (vocabulaire « machine virtuelle », « bucket », impératifs, glose de CLI, virtiofs, AOF) ; liens par issue.
- **Lot B (restructuration) :** « Au programme » en tête ; plan de ports en section 5 avec les ports de l'annexe ; trois démonstrations en détail ; un seul tableau de référence pour les 14 autres services (gabarit Service, Commande, Interface et test, Piège, Rôle) ; tableau des limites avec colonne « À dire » ; « Fichiers Windows » déplacé en section 1 ; notes de l'orateur, checklist et plan B en fin de fichier ; chemin interne retiré des sources.
- **Décisions prises à la place de David :** lignes « À dire » du tableau des limites = compose, DNS entre conteneurs, `--restart`, réglages noyau ; un seul fichier avec une section « Notes de l'orateur » en fin ; minimum de version expliqué une fois (§2).
- **Non appliqué :** schéma d'architecture, script de démarrage et d'arrêt complet pour remplacer compose, journal `TESTS.md`, démonstration scénarisée de Grafana, version séparée du guide pour le public. Toutes les commandes restent à tester.
