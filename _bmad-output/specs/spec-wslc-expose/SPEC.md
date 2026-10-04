---
id: SPEC-wslc-expose
companions:
  - services.md
  - known-limits.md
  - ../../planning-artifacts/research/technical-wslc-wsl-containers-2026-10-04/research.md
  - ../../planning-artifacts/research/technical-services-demo-wslc-2026-10-04/research.md
sources: []
---

> **Canonical contract.** This SPEC and the files in `companions:` are the complete, preservation-validated contract for what to build, test, and validate. Source documents listed in frontmatter are for traceability — consult them only if you need narrative rationale or prose color this contract intentionally omits.

# Exposé WSLC : document Markdown de commandes

## Why

**Opportunité à saisir.** WSLC (WSL Containers), le runtime de conteneurs Linux intégré à WSL, est stable depuis le 2026-09-29 (WSL 3.0.1) : plus de moteur séparé à installer, une CLI proche de Docker. Des développeurs qui connaissent Docker doivent en comprendre en 15 à 20 minutes ce qui change, ce qui manque et comment le rejouer eux-mêmes. Le livrable est un seul document Markdown, support de l'exposé et référence rejouable ; le ton informe honnêtement (avantages et limites), il ne cherche pas à convaincre d'adopter WSLC.

## Capabilities

- **CAP-1**
  - **intent:** Le lecteur comprend ce qu'est WSLC, son statut (stable depuis le 2026-09-29) et l'installe.
  - **success:** Sur un Windows à jour, `wsl --update`, `wslc version` et `wslc run --rm hello-world` s'exécutent comme écrit et affichent le message de bienvenue.
- **CAP-2**
  - **intent:** Un développeur Docker retrouve ses réflexes avec `wslc` et voit ce qui diffère.
  - **success:** Un tableau Docker → `wslc` couvre au moins `run`, `exec`, `logs`, liste des conteneurs (`ps` ou `list`), `build` (fichier `Containerfile`) et `prune` ; la démo nginx répond à `curl localhost:8080`.
- **CAP-3**
  - **intent:** Le lecteur démarre chacun de dix services de développement en une commande et vérifie qu'il fonctionne : PostgreSQL, Redis, Solr, Mailpit, SonarQube, Grafana, Gitea, Meilisearch, Prometheus, SeaweedFS.
  - **success:** Chaque recette de `services.md` donne la commande, la vérification et l'étiquette « testé » ou « à tester » ; trois démos au plus sont montrées en direct, les autres sont à lire ; aucun conflit de port dans le plan de ports.
- **CAP-4**
  - **intent:** Le lecteur sait ce qui ne marche pas encore et quoi éviter.
  - **success:** Une section liste au moins : compose absent, DNS entre conteneurs non confirmé, `--restart` non confirmé, `--gpus all` en échec sous Alpine, réglages hôte de SonarQube, MinIO à éviter ; chaque limite est datée et sourcée, d'après `known-limits.md`.
- **CAP-5**
  - **intent:** Le lecteur peut rejouer le document tel quel, en sachant sur quelle version il a été écrit.
  - **success:** La version de WSL (3.0.1) et la date figurent en tête ; chaque bloc de commandes porte son étiquette de vérification ; la syntaxe est PowerShell, avec des volumes nommés.

## Constraints

- Le document vise WSLC en disponibilité générale (WSL 3.0.1, 2026-09-29), est daté, et épingle chaque image sur un tag précis : pas de `latest` flottant, pas d'instructions de préversion (`--pre-release`).
- Aucun bloc de commandes n'est présenté comme « testé » sans exécution réelle sous Windows.
- Un service = une commande `wslc run`. Pas de compose, et pas de démo entre conteneurs tant que le DNS par nom n'est pas testé : chaque démo en direct montre un service seul.
- Les ports ne se chevauchent pas : Gitea sur 3001 et Adminer sur 8081, pour éviter les conflits avec Grafana et les autres.
- Les volumes sont nommés, pas des dossiers Windows (performances de virtiofs, droits d'utilisateur propres à chaque image).
- SonarQube n'est jamais montré en direct sans test préalable : il exige `vm.max_map_count=524288` et le réglage dans la VM WSLC est inconnu.
- Chaque limite citée est datée et sourcée ; une limite sans source ni date n'entre pas dans le document.
- Le document est en français, dans un seul fichier Markdown, en syntaxe PowerShell.

## Non-goals

- Pas de diapositives.
- Pas de comparaison chiffrée avec Docker Desktop ou Podman (aucune source officielle ni testée).
- Pas de stack multi-conteneurs ni de compose.
- Pas de durcissement ni de configuration de production : mots de passe d'exemple et modes de développement (`start-dev`, `MEILI_ENV=development`).
- Pas de couverture de l'API développeurs (paquet NuGet `Microsoft.WSL.Containers`) ni des contrôles d'entreprise (Intune, liste de registres, Defender).
- Aucune affirmation sur le runtime sous-jacent (containerd ou autre), non établi.
- Pas de GPU ni d'USB en démo : cités comme limites seulement.
- Pas de migration de données depuis un autre outil vers WSLC.
- Pas de démo de MinIO, MailHog, OpenSearch, Garage ni MySQL ; Keycloak, RustFS, pgAdmin, Adminer, RabbitMQ, MongoDB et MariaDB en annexe seulement.
- Pas de pédagogie des conteneurs ni de Docker : le public les connaît.

## Success signal

- Un développeur qui connaît Docker, installant WSLC à partir du seul document, lance au moins trois des services (par exemple Mailpit, Grafana et Gitea) avec les commandes telles qu'écrites, vérifie chacun comme indiqué, et sait nommer deux limites actuelles (compose absent, DNS entre conteneurs non confirmé). Le jour de l'exposé, aucune commande montrée en direct n'a échoué, et chaque bloc du document porte une étiquette « testé » ou « à tester » avec la version de WSL et la date.

## Assumptions

- « solar », dans la demande d'origine, désigne Apache Solr ; Solr est conservé en plus de SonarQube.
- Trois démos en direct au plus sur 15 à 20 minutes ; les sept autres services restent en lecture dans le document.
- Démos en direct proposées par défaut : Mailpit, Grafana, Gitea (rapides, avec interface web, sans réglage hôte) ; choix final après les tests.
- Le critère « aucune commande en direct n'a échoué » est conservé tel quel ; la variante « tout échec est expliqué par une limite déjà documentée » n'a pas été retenue.
- Le public est composé de développeurs qui connaissent Docker et travaillent sous Windows avec PowerShell.

## Open Questions

- Les commandes seront-elles testées sous Windows avant l'exposé ? Sans test, tous les blocs restent « à tester » et le critère « aucune commande en direct n'a échoué » n'est pas vérifiable.
- Le DNS par nom de conteneur fonctionne-t-il sous WSLC sur un réseau défini par l'utilisateur ?
- Comment régler `vm.max_map_count` dans la VM d'une session WSLC, et SonarQube démarre-t-il ?
- Les options de `wslc run` (`--restart`, `--env-file`, valeurs de `--memory`) : que montre `wslc run --help` sur 3.0.1 ?
- Quels sont les prérequis Windows (build, virtualisation) ? Aucune source lue ne les donne.
- L'image `prom/prometheus` se collecte-t-elle elle-même par défaut ?
- Quelles sont les trois démos en direct retenues ?
