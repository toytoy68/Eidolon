# Veille du 2 novembre 2026 — qualification et décisions de travail

Cette suite de l'étude `b6d1760` distingue les annonces amont, les contrats déjà
présents dans Eidolon et les expériences exécutées. La X399 est restée éteinte.
Les tests de code utilisent des répertoires temporaires synthétiques ; les essais
Gemma utilisent une copie isolée de Core sur l'ancienne VM100. Aucun paquet de
Hindsight, Hermes, OpenHands, Falco ou du fork V100 n'a été installé dans Eidolon.

## Sources vérifiées

Les fichiers LICENSE racine ont été lus aux références ci-dessous. Ces lectures
ne constituent pas un audit des licences de toutes les dépendances ou modèles.
Les SHA complets sont aussi conservés dans [sources.json](veille-2026-11-02-sources.json).

| Projet | Référence vérifiée | Constat utile |
|---|---|---|
| Hindsight | [v1.1.0, `a5a619b`](https://github.com/vectorize-io/hindsight/releases/tag/v1.1.0), MIT | Release du 21 octobre. Le correctif du 30 octobre [`7b46a07`](https://github.com/vectorize-io/hindsight/commit/7b46a07c82f997903230f0fb33deafe9dfd2e22b) conserve les dates de l'événement et de sa mention jusque dans les extraits bruts. Il est **156 commits après le tag**, pas livré par ce tag. |
| Hermes | [`10d34c5`](https://github.com/NousResearch/hermes-agent/commit/10d34c53d9678e9a8e6ce9887b598136a663298c), puis [`4d26e0c`](https://github.com/NousResearch/hermes-agent/commit/4d26e0cceaed0a08ede833caec0ec6d6ca77a4b8), MIT | Une route supposée ne devient pas une autorité par sa sauvegarde. Après reprise, vérifier le propriétaire et le transport ; les écouteurs webhook partagés constituent un cas explicite distinct. |
| OpenHands SDK | [v1.59.0, `ccae18c`](https://github.com/OpenHands/software-agent-sdk/releases/tag/v1.59.0), MIT | [PR 6257](https://github.com/OpenHands/software-agent-sdk/pull/6257) : budget transmis et persisté par le serveur, avec différence entre champ absent et `null`. [`d0d9d81`](https://github.com/OpenHands/software-agent-sdk/commit/d0d9d81324210414462fb676b377e0563a08a0cb) : le masquage des secrets ne doit pas casser les identifiants d'événements. |
| Falco | [0.45.2, `7625e20`](https://github.com/falcosecurity/falco/releases/tag/0.45.2), Apache-2.0 | Actualisation des magasins de confiance après eMudhra et d'OpenSSL. Cela ne démontre aucun gain de détection ou coût acceptable sur notre machine. |
| llama.cpp-v100 | [v100-2.0.1.1, `f17d925`](https://github.com/1115714829/llama.cpp-v100/releases/tag/v100-2.0.1.1), GPL-3.0 | Publication du 23 octobre confirmée. CUDA 12.6/NCCL 2 et environ 63 GiB hôte concernent la configuration annoncée par l'auteur, dont un contexte 262144 et une topologie particulière ; aucune extrapolation de débit à la X399. |

Autre piste Hindsight vérifiée : [`a461a00`](https://github.com/vectorize-io/hindsight/commit/a461a00c0265955b8805dffab9d06bf30deb1ba8)
borne les workers d'embeddings locaux et propage leur contexte d'appel. Le moteur
actuel d'Eidolon appelle un service d'embeddings : ne pas copier ce pool de threads
sans reproduire d'abord le même problème dans le processus concerné.

## Ce que les tests protègent déjà

Bases : Memory `0691e68b452fb79c671ab10b9a6c054a67e6d4d3`, Core
`04c384ddad2fb18f61afbd490ea0f913e7d0b509`. Les tests sont écrits pour Eidolon ;
aucun code amont n'a été copié.

- Memory : dates de provenance, observation et création conservées après lecture
  disque, troncature et sérialisation ; une date absente reste absente ; deux
  occurrences de 02 h 30 lors du changement d'heure restent distinctes grâce aux
  offsets ; contradictions visibles et sources remplacées étiquetées ; budget
  insuffisant = retrait de l'élément complet, pas suppression de ses réserves.
- Core : vraie restauration d'une sauvegarde SQLite signée, puis API à deux
  clients. Propriétaires, séparation des conversations, rejeu idempotent,
  révocation et rotation des clés restent effectifs ; une liaison de profil
  modifiée est refusée au redémarrage.
- Traversée Memory → HTTP réel en boucle locale → validation Core → prompt :
  dates et provenance conservées, y compris l'absence de date source.
- Tests négatifs par modification en mémoire du code : supprimer les dates fait
  échouer deux cas ; neutraliser le contrôle de propriétaire fait échouer le
  scénario de restauration. Aucun de ces sabotages n'est conservé dans les sources.

**Limite de sécurité :** restaurer seulement les conversations ne restaure pas
`clients.sqlite3`. Les tests de révocation ne prouvent pas la sécurité d'un retour
à un instantané complet qui restaurerait aussi d'anciennes clés ou autorisations.
Ce scénario nécessite une recette distincte de révocation après reprise globale.
Ni les webhooks Hermes, ni un redémarrage physique de la X399 ne sont testés ici.

## Lacunes précises et ordre de travail

| Priorité | Travail concret | Critère de validation |
|---|---|---|
| P1 | Date du propos d'origine distincte de la date d'import et de l'événement. Formaliser son origine pour les archives ; ne pas supposer que tout `provenance.timestamp` a le même sens. | « Hier » change de référence avec la date du propos, jamais avec celle de l'import ; source non datée = date indéterminée. |
| P1 | Sélection du bon sujet et de la bonne preuve avant réponse. Les échecs FR100 ne sont pas corrigés par le seul transport des métadonnées. | Identifiants explicites, personnes/projets voisins, paraphrases et contrôles négatifs ; score documentaire et qualité des réponses mesurés séparément. |
| P1 | Concevoir un accès historique explicite, avec date de référence et provenance d'autorisation. Le bridge courant refuse le mode historique et impose l'heure serveur ; `BetaRuntime.recall_for` demande le mode opérationnel. | Contrat et tests de portée avant activation. Un rappel historique des enregistrements actuels ne doit pas prétendre reconstruire toutes leurs anciennes révisions. |
| P1 | Recette de reprise globale des profils/clefs, en plus de la restauration des conversations. | Une révocation récente ne doit pas être annulée silencieusement par un ancien instantané complet ; aucun repli sur un profil par défaut. |
| P2 | S'inspirer d'OpenHands pour un budget durable par conversation et des identifiants immuables. | Compteur conservé après reprise, refus avant travail au-delà du budget, idem sur rejeu ; les textes sensibles restent masqués sans modifier les références. |
| P3 | Évaluer Falco comme capteur passif facultatif. | Comparaison sans/avec sur même VM jetable : CPU/RSS, événements perdus, bruit, pertinence des alertes, retrait propre. Aucun droit d'action ajouté au LLM. |
| Différé | Comparer officiel/fork Volta après réception des V100. | Même modèle/quantification, topologie mesurée, contextes 16K/32K, répétitions alternées, TTFT/tok/s/RAM/VRAM/énergie/stabilité ; séparation `layer` / `tensor`. |

Core possède déjà des limites de contexte et de durée par tentative, ainsi qu'un
quota web durable par profil. Ce n'est pas un budget global de conversation, ni
une mesure en joules : aucune équivalence avec le budget monétaire OpenHands
n'est revendiquée. Pas de remplacement du moteur d'agents proposé.

La mention de **78 GiB de RAM** dans la note initiale est une photographie ancienne.
Le dernier relevé de la campagne X399 du 2 novembre donne environ **110 GiB visibles**.
La RAM réellement disponible devra être remesurée avec les VM prévues avant un
essai V100 ; les 63 GiB cités par l'auteur ne sont pas une exigence générale.

Open-Sandbox, agent-memory-bench, Graphiti, DCGM Exporter et agent-sandbox restent
en surveillance. Le présent lot ne constitue pas leur audit de code ni leur essai.

## Résultats de génération et décision

21 nouveaux tests de contrat sont passés : 8 Memory, 13 Core. Suites ciblées :
Memory 66 tests + 5 sous-tests, aucun ignoré ; Core 123 tests + 34 sous-tests,
13 anciens tests ignorés faute de age. Les nouveaux scénarios de restauration
signée OpenSSL sont exécutés. Il ne s'agit pas d'une nouvelle suite complète.

62 réponses réelles de Gemma3:4b ont été examinées sur VM100, avec sources
synthétiques fournies : six cas temporels, 22 G294 avant, puis six cas temporels,
six cas supplémentaires et 22 G294 avec une consigne de preuve plus explicite.
Résultat : 3/6 sur les cas temporels avant comme après ; 15/22 sur G294 dans les
deux conditions, mais une régression sur la demande d'action. La consigne est
**rejetée**, conservée comme patch documentaire et retirée du code proposé.

Le modèle confond date de propos et date d'événement, prend une date d'import
pour une date du fait et répond parfois sur une autre personne. La conservation
des métadonnées est donc nécessaire mais insuffisante. La notation automatique
a aussi été corrigée pour une abstention avec « donc » ; les mêmes réponses
passent de 2/6 à 3/6 après renotation, sans changement du modèle. Le second jeu
donne 3/6 automatiquement, 4/6 après lecture d'une autre abstention correcte.
Ces petits échantillons ne sont pas des estimations générales de précision.

Les livrables retenus sont les tests, les cas reproductibles, l'amélioration du
banc et cette qualification. Aucune intégration amont ni modification du modèle
actif n'est validée. Le prochain lot doit porter sur la sélection structurée de
la preuve et du temps, avec contrôle du sujet, avant génération.

## Livrables proposés

- [Memory — PR #6](https://github.com/toytoy68/Eidolon-Memory-Engine/pull/6),
  commit `d30512dbda60651ec3cb0cf9afc634dc39140e38` : huit tests et rapport.
- [Core — PR #6](https://github.com/toytoy68/Eidolon-Bootstrap-Framework/pull/6),
  commit `d7fb71028091ead3a6435959c884eab6224ac297` : treize tests, banc et réponses brutes.

Ces deux propositions sont en brouillon, empilées sur les branches d'audit
précédentes. Leurs arbres Git publiés sont identiques aux arbres testés localement.
Aucune fusion ou activation en production. Les résultats ci-dessus proviennent
des exécutions décrites ; ils ne prétendent pas à une nouvelle certification CI
de tous les dépôts.
