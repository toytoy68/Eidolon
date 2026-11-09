# Étude d'intégration des priorités — Eidolon (2026-11-09)

**État : conception / étude de risques, aucun essai machine effectué dans ce lot.** Base : [qualification 02/11](veille-2026-11-02-qualification.md), qui rapporte déjà 21 nouveaux tests de contrats (8 Memory et 13 Core), ainsi qu'une limite majeure de restauration complète de `clients.sqlite3`. Ne pas répéter le travail existant. La X399 a désormais été annoncée à 128 Go physiques : confirmer la RAM libre et le fonctionnement des VM au banc, ne pas se fonder sur les anciens chiffres de 78/110 Gio.

## Architecture cible, sans adoption des runtimes externes

```text
Clients (profils / mobile / bureau / voix)
      | identité authentifiée et canal identifié
      v
Core gateway / session owner (état et événements durables)
      | demande d'action structurée, identifiant immuable, budget, contexte
      v
DARME : validation déterministe par défaut-deny
      | identités, ACL, chemin canonique, origine, durée, consentement
      v
Exécuteur isolé, permissions minimales, sortie/timeout contrôlés
      | preuves et traces signées / datées
      +--> Memory Engine (source, date du propos, date événement, date import)
      +--> logs/mesures locaux (sans secrets)
LLM : raisonnement uniquement ; jamais détenteur des secrets, du shell ni des droits.
```

### A. P1 Darme — Sandlock vs isolation native

**Constat** : dépôt [Sandlock](https://github.com/multikernel/sandlock), licence annoncée Apache-2.0 ; dernier tag confirmé à l'étude `v0.9.0` (13/10). La mention `v0.9.1` / « approbation différée » de la veille initiale n'est **pas confirmée**. Ne pas confondre le produit Sandlock avec le mécanisme noyau Landlock.

**Hypothèse** : exécuter les seuls outils accordés par Darme en processus isolé, sans accès direct LLM→OS. Le choix initial doit être *minimal* : comptes Linux non privilégiés + systemd sandboxing (NoNewPrivileges, ProtectSystem, PrivateTmp, RestrictAddressFamilies, bind mounts contrôlés) / éventuellement Landlock ; Sandlock n'est retenu que s'il prouve un gain mesurable. Les mécanismes doivent être vérifiés dans l'environnement réel (Debian 13 / noyau utilisé / Proxmox vs KVM). Ne pas appliquer sur l'hôte ni casser les accès MCP existants.

**Prototype de test (non exécuté)** : VM jetable sans secrets ni réseau extérieur ; outil simulant lecture autorisée `/work/allowed`, lecture interdite `/root` et `/etc/shadow`, écriture hors `/work`, ouverture de socket interdite, commande enfant et dépassement de timeout ; tous les refus attendus doivent être fiables. Evaluer fuite par symlink, chemins `../`, race TOCTOU, héritage de descripteurs, variables d'environnement, redirection stdout/stderr, arrêt complet du groupe de processus.

**Décision gate** : (1) 100 % des essais d'évasion du lot refusés, (2) pas de permission supplémentaire au LLM, (3) retrait sans état résiduel, (4) surcharge CPU/RAM/latence mesurée contre service systemd natif. Aucun déploiement si un test d'évasion passe.

**Maintenance/licence** : projet pré-1.0 (API potentiellement instable), confinement Linux complexe ; transposer le moins de dépendances possible ; vérifier licences du binaire et dépendances dans la référence retenue.

### B. P1 Core — session propriétaire unique / Hermes

**Source** : [Hermes v0.23.0](https://github.com/NousResearch/hermes-agent/releases/tag/v0.23.0) ; principe de session tenue par passerelle et canaux clients. Il s'agit d'une **inspiration** et non d'une recommandation d'installer Hermes.

**État préexistant** : les essais du 02/11 couvrent déjà une restauration SQLite signée, séparation à deux clients, rotation/révocation, idempotence et refus de liaison modifiée ; **lacune documentée** : restauration simultanée de `clients.sqlite3` peut ressusciter une ancienne révocation. Les webhooks et le reboot physique ne sont pas qualifiés.

**Contrat cible** : `actor_id`, `profile_id`, `channel_id`, `session_id`, `request_id`, `approval_id`, `policy_version`, `auth_epoch`, `expires_at` ; choix de noms provisoire. Le propriétaire de session est validé par Core, pas par une métadonnée libre du LLM. Une approbation est liée à l'identité, à l'opération normalisée, à la ressource, au profil, à un nonce et à une durée ; consommation unique et journal immuable. Les révocations doivent être comparées à une autorité **hors snapshot restauré** (registre d'époques monotones ou service équivalent sauvegardé séparément) ; un ancien snapshot est mis en quarantaine jusqu'à réconciliation, jamais autorisé automatiquement.

**Tests recommandés** : restauration snapshot N-1 après révocation N ; client A essaie de rejouer autorisation B ; duplication de request_id ; compétition de deux canaux sur une même session ; ancien webhook et webhook inattendu ; arrêt/redémarrage entre décision et exécution ; rejeu après expiration. Réussite = aucun droit restauré, aucun mélange inter-profil, action au plus une fois ou idempotence prouvée, journal traçable.

**Maintenance** : sérialisation des sessions et conservation de l'autorité imposent quelques tables/états ; veiller à la gestion d'un point de défaillance unique. Coût estimé qualitativement « modéré », pas chiffré avant prototype.

### C. P1 Memory — fidélité temporelle / Hindsight

**Source** : [Hindsight v1.2.0](https://github.com/vectorize-io/hindsight/releases/tag/v1.2.0). Éviter de confondre corrections du tag et correctifs hors tag sur `main`.

**État préexistant** : 8 tests datation/provenance déjà écrits, mais Gemma3:4b n'a atteint que 3/6 sur les cas temporels dans le jeu déjà testé et a confondu plusieurs sujets. Une consigne de prompt plus détaillée n'a pas apporté de progrès ; ne pas recommencer cette approche sans changement structurel.

**Contrat de preuve proposé** (schéma documentaire, pas code actif) :
```json
{
  "fact_id": "immutable-id", "subject_id": "stable-subject",
  "claim": "text", "source_ref": "document/line",
  "utterance_at": "ISO-8601 or null",
  "event_at": "ISO-8601 or null",
  "ingested_at": "ISO-8601",
  "supersedes": ["other-fact-id"],
  "authorization_scope": "profile-or-tenant",
  "confidence": "derived-from-evidence-not-LLM-alone"
}
```
Temps de propos, événement et import distincts ; `null` inconnu jamais converti en « maintenant ». Conflits conservés avec preuve et sujet ; suppression de source entraîne invalidation cohérente de l'index dérivé ; éviter la suppression non intentionnelle d'historique utile.

**Banc** : 1) deux homonymes / projets proches, 2) source « hier » importée 30 jours après, 3) faits contradictoires même jour, 4) fuseau + changement d'heure, 5) suppression de source, 6) 2 imports concurrents, 7) reprise pendant indexation, 8) abstention sans preuve. Comparer sélection de document, sélection de sujet, attribution temporelle, réponse finale **séparément**. N'appliquer aucune migration du stockage sans sauvegarde, rollback et comparatif.

**Décision gate** : zéro fuite inter-profil, aucune fabrication de dates inconnues, amélioration mesurée sur jeux aveugles et absence de régression du banc FR100 ; métriques de latence et RAM à établir. Licence Hindsight MIT annoncée, dépendances à contrôler si importées.

### D. P2 Observabilité GPU — NVML/nv-monitor vs DCGM 4.9.0

**Sources** : [nv-monitor](https://github.com/wentbackward/nv-monitor) (MIT annoncé) ; [DCGM Exporter 4.9.0](https://github.com/NVIDIA/dcgm-exporter/releases/tag/4.9.0) (Apache-2.0 annoncé). L'objectif est un rapport compatible avec nos V100 et les Shelly, pas une refonte Prometheus obligatoire.

**Architecture privilégiée** : collecteur en lecture seule par nœud, conversion vers un schéma **interne stable** en unités SI (`gpu_power_watts`, `gpu_memory_bytes`, `nvlink_bytes_per_second` si réellement exposé), provenance source/version, horodatage ; agrégation énergie par intégration `W × Δt` avec étiquetage des lacunes de mesure. Corréler à Shelly pour comparer GPU, hôte et puissance prise ; ne jamais assimiler watts GPU à watts maison.

**Risques** : DCGM 4.9.0 change 23 noms / 11 unités et nécessite prudence d'alignement hôte/libdcgm 4.7.1. Capteurs V100 absents possibles (NVLink, compteurs XID, santé) ; nv-monitor expose Prometheus sans TLS natif : écouter sur loopback et protéger au travers du réseau existant, jamais exposer anonymement. Ne pas interpréter « métrique absente » comme « zéro ».

**Banc** : échantillons 1 s et 5 s, 15 min repos + 15 min charge constante par carte (si fenêtre approuvée), comparaison avec `nvidia-smi`, `nvidia-smi topo -m`, journaux XID et Shelly, coût CPU/RSS/perte d'échantillons, temps de mise en service/suppression. Retenir l'outil le plus simple qui couvre les métriques V100 réellement disponibles.

### E. P2 llama.cpp V100 / NVLink

**Séquence préparatoire seulement** : figer topologie GPU, pilote, CUDA, VRAM, version llama.cpp officielle/fork Volta, modèle exact, quantification, flags ; pas de test automatique sur X399 ni sur GPU en cours d'usage. Vérifier issues exactes des bugs Flash-Attention, CUDA Graphs, `tensor`. Démarrer référence `layer` puis `tensor` isolé, mesurer TTFT, tokens/s, VRAM/RAM, watts, incidents, stabilité longue durée. En cas de hang : arrêt de la seule tâche de test, sans redémarrer les services Eidolon. Fork GPL-3.0 : coût de distribution/maintenance distinct de l'usage interne.

### F. P2 ProjectMem et OpenHands

Étudier uniquement le **registre de décisions** (hypothèse, expérience, résultat, preuves, statut, date, validateur, rollback) et la **fin de vie des processus** (timeouts, descendants, budgets et identifiants immuables). Aucune mémoire de travail brute, secret, token, IP privée ou sauvegarde ne doit être publiée dans GitHub public. Éviter l'intégration d'un second framework agentique. Préférer un format Markdown/JSON local, versionné et expurgé.

## Ordre de réalisation et critères d'arrêt

| Lot | Livrable suivant (non exécuté ici) | Blocage avant intégration |
|---|---|---|
| 1 | Contrat d'autorité et matrice d'évasion Darme, comparaison sandbox natif/Sandlock | Tests d'isolation et absence de contournement |
| 2 | Contrat restauration totale Core et scénarios de révocation | Autorité des révocations conservée après rollback |
| 3 | Jeux de preuves Memory ; résultats par étage | Gain reproductible, aucune date inventée |
| 4 | Schéma métrique V100 commun ; matrice DCGM/NVML | Compatibilité V100 et unités vérifiées |
| 5 | Protocole benchmark V100 reproduisible | Accord sur fenêtre de test, topologie et arrêt |
| 6 | Registre d'expériences ProjectMem-compatible | Aucune exposition de secrets |

**Conclusion à ce stade :** intégrer des *contrats, tests et éventuellement capteurs isolés* si les mesures justifient leur coût ; ne pas adopter Hermes, Hindsight, OpenHands ou Sandlock en bloc. Aucun de ces projets n'a été installé ni testé sur les machines Eidolon par cette étude ; aucun résultat de performance n'est revendiqué.
