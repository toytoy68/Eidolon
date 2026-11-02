# Veille open source Eidolon — pistes à étudier (2026-11-02)

**Statut : étude uniquement.** Ce document consigne des pistes issues de la veille hebdomadaire ; il ne valide ni leur fonctionnement sur notre matériel ni leur intégration dans Eidolon. Aucun déploiement, fusion ou changement de code n'est autorisé par ce document.

## Suivi du 2 novembre

La vérification des sources et les qualifications exécutées après cette note sont
dans [le compte rendu de qualification](veille-2026-11-02-qualification.md).
Le statut final ci-dessous décrit le dépôt initial du matin, pas ces travaux ultérieurs.

## Règles d'évaluation

Pour chaque piste : conserver les liens vers les sources primaires (release, commit, issue), la version/commit exact, distinguer **annonce amont**, **compatibilité à vérifier**, **test local réussi**, **décision d'architecture** ; contrôler licence et dépendances, mesurer RAM/VRAM, CPU/GPU, latence, consommation, frais de maintenance et surface d'attaque. Préférer des prototypes isolés et réversibles. Ne pas donner aux LLM un accès direct au système : Core/Darme restent les seuls médiateurs d'actions autorisées.

## Priorités

### P1 — Memory Engine : Hindsight, preuve et chronologie
- Source : https://github.com/vectorize-io/hindsight
- Signal de veille (à reconfirmer sur commits/releases) : corrections fin octobre sur dates associées aux preuves, concurrence et exécuteur séparé pour embeddings locaux ; release mentionnée v1.1.0.
- **Opportunité** : enrichir notre campagne de tests sur la fidélité des sources et la temporalité (« hier », changement de date de référence, versions contradictoires), indexation, citations et provenance.
- **Compatibilité** : principes transposables à notre Memory Engine Python/FTS5/hybride ; pas besoin d'adopter l'implémentation complète. Licence MIT rapportée : à vérifier dans le tag/commit étudié.
- **Risques et maintenance** : dépendances supplémentaires, comportement de réindexation, dette de migration des schémas, régression de latence ; étudier les tests plutôt que remplacer le moteur.
- **Étude proposée** : jeux de tests datés et contradictoires, références obligatoires, mesures rappel/précision/latence, puis comparaison avant/après des garde-fous de preuve.

### P1 — Core : reprise de session et isolation des identités (Hermes)
- Source : https://github.com/NousResearch/hermes-agent
- Signal de veille (à confirmer par commit) : correction annoncée début novembre sur le routage d'une session restaurée et conservation d'écouteurs webhook.
- **Opportunité** : tests de restauration, séparation stricte des profils et canaux, absence de réponse au mauvais utilisateur après reboot.
- **Compatibilité** : idées et tests potentiellement transposables sans installer Hermes. Licence MIT rapportée, à contrôler.
- **Risques et maintenance** : mauvaise association profil/session, fuite inter-utilisateurs, complexité du suivi des événements.
- **Étude proposée** : scénarios « profil A/B », reprise après panne, webhooks restaurés, cas d'attaque et tests de non-régression.

### P2 — V100 / NVLink : comparaison des moteurs d'inférence
- Sources : https://github.com/ggml-org/llama.cpp ; https://github.com/1115714829/llama.cpp-v100
- Signal de veille : fork ciblant Volta/SM70, publication 2.0.1.1 mentionnée le 23 octobre, revendications de fonctionnement en 2× V100 16 Go avec CUDA 12.6/NCCL 2. **Ce sont des affirmations amont, non vérifiées sur Eidolon.**
- **Opportunité** : comparaison Ollama/llama.cpp officiel/fork Volta sur 1 et 2 V100 ; vérifier précisément les performances NVLink et l'intérêt du parallélisme `layer` vs `tensor`.
- **Compatibilité** : Threadripper 1950X/78 Gio RAM et futures V100 ; vérifier PCIe/SXM2, topologie réelle NVLink, driver, CUDA, versions ggml, modèle et quantification. La RAM annoncée par l'auteur (~63 Gio) laisserait peu de marge sur 78 Gio pour l'OS et les VM.
- **Licence** : fork GPL-3.0 rapporté ; vérifier les obligations avant toute redistribution ou intégration. Examiner séparément les licences des modèles.
- **Risques et maintenance** : divergence du fork, corrections upstream non reprises, régressions CUDA/Flash Attention/CUDA Graphs, consommation et compatibilité des kernels SM70.
- **Étude proposée** : commencer par `layer`, jeu fixe de prompts, même modèle/quant, contexte 16K puis 32K ; mesures tok/s, TTFT, RAM, VRAM, watts, stabilité, erreurs et comportement après redémarrage. `tensor` et NVLink uniquement en phase isolée et instrumentée.

### P2 — Core : OpenHands Software Agent SDK
- Source : https://github.com/OpenHands/software-agent-sdk
- Signal de veille : version 1.59.0 annoncée le 28 octobre avec mécanismes ACP, profils, identifiants d'événements et budgets de conversation (à vérifier dans les notes de release).
- **Opportunité** : s'inspirer des contrats d'outils, journaux d'événements, budgets et contrôles d'autorisation.
- **Compatibilité** : étude d'API et de tests sans nouveau runtime agentique. Licence MIT rapportée, à vérifier.
- **Risques et maintenance** : double moteur d'agents, dépendances et privilèges superflus, contournement potentiel de la séparation LLM/système.
- **Étude proposée** : grille comparative des interfaces avec Eidolon Core ; réimplémentation minimale seulement si gain démontré.

### P3 — Darme : Falco en capteur facultatif
- Source : https://github.com/falcosecurity/falco
- Signal de veille : version 0.45.2 annoncée le 29 octobre ; vérifier le changelog de sécurité avant toute décision.
- **Opportunité** : détection de comportements de processus Linux pour alimenter Darme, sans autorité d'action pour le LLM.
- **Compatibilité** : Debian 13/Proxmox à tester ; dépendance aux mécanismes noyau/eBPF et capacités système. Licence Apache-2.0 rapportée, à vérifier.
- **Risques et maintenance** : privilèges élevés, événements nombreux, faux positifs, empreinte CPU/RAM, règles à maintenir.
- **Étude proposée** : capteur passif sur VM de test, métriques CPU/RAM, bruit de logs, alertes pertinentes, coûts de mise à jour ; abandonner si le gain ne justifie pas la charge.

## Projets conservés sous surveillance (aucune adoption proposée à ce stade)

| Projet | Lien | Point à suivre |
|---|---|---|
| Open-Sandbox | https://github.com/colinlikescode/Open-Sandbox | Isolation des outils, ergonomie et surface d'attaque |
| agent-memory-bench | https://github.com/hamza-dev-tech/agent-memory-bench | Méthodologie d'évaluation mémoire reproductible |
| Graphiti | https://github.com/getzep/graphiti | Graphes temporels, dépendances et coût d'exploitation |
| DCGM Exporter | https://github.com/NVIDIA/dcgm-exporter | Compatibilité métriques Tesla V100, topologie, VRAM/puissance |
| agent-sandbox | https://github.com/agent-sandbox/agent-sandbox | Sandboxing et politique de privilèges |
| Hindsight | https://github.com/vectorize-io/hindsight | Provenance, récupération temporelle et embeddings |
| Hermes Agent | https://github.com/NousResearch/hermes-agent | Séparation des profils et reprise des sessions |
| OpenHands SDK | https://github.com/OpenHands/software-agent-sdk | Contrats agent/outils et auditabilité |

## Critères de passage « étude » → « candidat à intégrer »

1. Source et version exactes vérifiées, licence et obligations documentées.
2. Prototype isolé, sans mutation de Core/Memory/Darme/OS de production.
3. Bénéfice chiffré sur nos tests (sécurité, rappel, latence, débit, énergie) et comparaison avec solution actuelle.
4. Budget dépendances/maintenance et procédure de retrait.
5. Validation explicite du propriétaire avant implémentation, fusion ou déploiement.

**État au 02/11/2026 : toutes les pistes restent EN ÉTUDE ; aucune n'est validée, intégrée ou déployée.**
