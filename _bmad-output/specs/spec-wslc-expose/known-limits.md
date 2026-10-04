# Limites connues de WSLC et des démos

Relevées le 2026-10-04, sur WSL 3.0.1 (GA du 2026-09-29). Chaque limite doit être datée et sourcée dans le document final ; sources dans les rapports de recherche adoptés.

| Limite | Détail | Statut | Source (rapport) |
|---|---|---|---|
| Compose absent | « Our top feature request for WSLc is adding compose support » : objectif `wsl compose up` sur un `compose.yaml` existant, prévu aux prochaines itérations. Visual Studio affiche « Docker Compose is not supported with the WSL container runtime (wslc) » (issue #41754, ouverte) | Vérifié (deux sources) | WSLC, section 5 |
| DNS entre conteneurs | Aucun exemple documenté de résolution par nom sur un réseau défini par l'utilisateur ; `wslc network create|connect|disconnect` existent. Risque pour toute démo où un service en appelle un autre | Non confirmé : test réel requis | WSLC, section 3 |
| Politique `--restart` | Absente en préversion ; 3.0.1 n'ajoute que la commande ponctuelle `wslc container restart` | Non revérifié sur 3.0.1 | WSLC, section 3 |
| Options refusées | `--privileged`, `--cap-add`, `--cap-drop`, `--security-opt`, `--read-only`, `--pids-limit` rejetées (2.9.13) ; `--memory`, `--cpus`, `--ulimit` acceptées | Confiance basse (source tierce, non retestée sur 3.0.1) | WSLC, section 3 |
| `--gpus all` sous Alpine | Fait échouer la création du conteneur sur les images Alpine/musl (issue #41791, ouverte) ; Debian/Ubuntu fonctionnent. Jamais démontré | Ouvert | WSLC, section 5 |
| Passage USB | Non supporté en préversion (`--device` et `--privileged` absents) ; jamais démontré | Périmé, non revérifié | WSLC, section 5 |
| Hôte vu d'un conteneur | `host.wslc.internal` non résolu dans les conteneurs musl (issue #41769, ouverte) ; accès aux services de l'hôte limité en préversion | Ouvert | WSLC, section 5 |
| Autres issues ouvertes | `wslc container cp` échoue sur les liens symboliques (#41793) ; `wslc run` renvoie ERROR_TIMEOUT (#41740) ; variables proxy de l'hôte et miroirs de registre non pris en charge (#41794) | Ouvert (178 résultats, première page lue) | WSLC, section 5 |
| Performances des fichiers Windows | Garder le code sur le système de fichiers Linux ; les fichiers Windows ralentissent « significativement » les outils Linux ; virtiofs plus rapide que Plan9 mais pas natif | Documenté (Learn) | WSLC, sections 3 et 5 |
| Réglages hôte exigés par SonarQube | `vm.max_map_count=524288` et ulimits ; réglage dans la VM WSLC inconnu. OpenSearch (`vm.max_map_count=262144`) écarté pour la même raison | Non résolu | Services, sections 2 et 3 |
| MinIO | Dépôt GitHub archivé le 2026-04-25, « no longer maintained » : à éviter ; remplaçant retenu : SeaweedFS | Vérifié (plusieurs sources) | Services, section 2 |
| MailHog | Non maintenu depuis ~2022 selon des sources secondaires ; successeur : Mailpit | Confiance moyenne | WSLC, section 4 |
| Runtime sous-jacent | Containerd ou runtime propre : non établi ; aucune affirmation permise | Inconnu | WSLC, section 1 |
| Comparaison Docker Desktop / Podman | Aucune comparaison officielle ni testée ; ne pas en présenter | Absente | WSLC, section 1 |
