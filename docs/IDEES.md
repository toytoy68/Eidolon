# Idées Eidolon

## MC-Create — agent d'architecture Minecraft (20 octobre 2026)

**Statut : idée / projet futur**, sans retarder la bêta Core et Memory.

**Objectif :** créer un agent spécialisé MC-Create, pilotant un moteur indépendant « Minecraft Architectural Generator » en Python. L'agent conçoit les bâtiments à partir d'exigences textuelles ; le moteur déterministe place les blocs, génère les plans et exporte des fichiers .schematic compatibles WorldEdit / Minecraft 1.12.2, notamment Nomifactory CEu 1.7.7.

### Composants

- Agent LLM : interprétation des besoins, styles, dimensions, orientation et contraintes.
- Générateur paramétrique : grille, axes de symétrie, modules de murs, fenêtres, escaliers, ascenseurs, toits, pièces et réseaux techniques.
- Skills extensibles : architecture, structure, circulation, décoration, industrie GregTech, blocs/mods Minecraft, WorldEdit et contrôle qualité.
- Validateurs : symétrie, dimensions, collisions, continuité des planchers, portes, circulation horizontale et verticale, accessibilité de chaque pièce depuis l'entrée, validité NBT et compatibilité des blocs.
- Aperçus avant import : plans, coupes et rendus 3D simplifiés.
- Eidolon Memory : conserver plans, retours utilisateur, erreurs, corrections et tests de non-régression ; apprentissage par évolution des règles/skills/code, sans réentraîner nécessairement le LLM.

### Banc d'essai V6 — résidence baroque de Void City

Reprendre les enseignements des prototypes V2 à V5 : volume et façade satisfaisants, mais escaliers incomplets, chambres absentes, galeries mal raccordées et défauts de symétrie. Exiger six validations : (1) symétrie, (2) accès à toutes les pièces, (3) escaliers/ascenseurs fonctionnels à tous les niveaux, (4) cohérence des hauteurs et des planchers, (5) schematic WorldEdit 1.12.2 valide, (6) aucune destruction involontaire de l'usine existante. Conserver le hall traversant reliant résidence et usine.

### Autonomie et sécurité

1. Mode Architecte : génération et tests hors serveur uniquement.
2. Mode Constructeur supervisé : pose sur monde de test après approbation, sauvegarde et retour arrière.
3. Mode Constructeur autonome borné : uniquement dans des zones explicitement autorisées, journalisation, limites et restauration.

### Ordre de réalisation proposé

Prototype Python autonome → bibliothèque de composants et tests → résidence V6 validée → API/skill Eidolon Core + mémoire → éventuelle connexion contrôlée au serveur Minecraft de test.

**Règle :** une construction ne doit pas être déclarée terminée parce qu'elle est visuellement réussie ; elle doit aussi être accessible, cohérente, fonctionnelle et testée.


## LFM3 — candidat léger pour Eidolon (3 novembre 2026)

**Statut : cible de travail acceptée par toytoy ; à qualifier.** Cet ajout planifie
une étude, sans installation, benchmark exécuté ou changement du modèle actif.
Périmètre initial : **LiquidAI/LFM3-1.7B texte**, quantification 4 bits si le moteur
retenu la prend correctement en charge. Les variantes audio/vision/omni sont
différées jusqu'à une décision distincte fondée sur les résultats texte.

### Objectif et place dans l'architecture

Évaluer un petit modèle local pour reconnaître les intentions, extraire des
informations, proposer des appels d'outils structurés et sélectionner des preuves
avec contrôle du sujet et des dates. La réussite reste à mesurer ; aucun
remplacement du modèle de discussion générale ou de Memory Engine n'est décidé.

Le LLM propose ; Eidolon valide les schémas, paramètres, droits et préconditions.
Darme et les contrôles déterministes restent responsables des limites
d'exécution. Le modèle ne reçoit pas d'accès direct au système.

### Campagne prévue

1. Vérifier les sources officielles, la licence et la compatibilité effective du
   runtime avec cette architecture. Épingler les révisions du modèle, du runtime
   et de la quantification ; consigner les paramètres et empreintes des fichiers.
2. Préparer un essai isolé sur X399 / Threadripper 1950X avec les réglages stables
   existants. Relever la RAM disponible, les VM actives et la topologie CPU.
   Commencer sur CPU ; un essai GPU dépendra du matériel disponible et du support
   constaté, sans déduire cette compatibilité de la seule taille du modèle.
3. Comparer à Gemma3:4b sur les mêmes entrées et preuves, notamment les cas
   temporels, G294 et les erreurs de sujet décrits dans la
   [qualification du 2 novembre](research/veille-2026-11-02-qualification.md).
   Ne pas comparer directement les latences historiques sur VM100 à celles de
   X399 : refaire la référence sur le même banc.
4. Mesurer séparément : sélection documentaire, respect du sujet, distinction
   date du propos / événement / import, qualité de réponse en français,
   abstention ou clarification, validité JSON et pertinence des appels d'outils.
   Exécuter les appels uniquement contre des outils simulés durant cette recette.
5. Mesurer TTFT, latence totale p50/p95, débit, temps CPU, RAM/VRAM, température,
   erreurs et stabilité. Commencer avec des contextes modestes et identiques ;
   ne pas prendre la fenêtre maximale annoncée comme cible opérationnelle.
6. Alterner référence et candidat, répéter les essais, conserver les réponses
   brutes et les échecs. Fixer les seuils d'acceptation avant la campagne ; faire
   une revue humaine des cas ambigus plutôt que se fier au seul score automatique.
7. Surveiller la puissance secteur Shelly et intégrer l'énergie sur la durée des
   essais. Distinguer consommation totale de la machine et surcoût par rapport
   à une référence comparable. En présence de Nomifactory-test, relever TPS,
   MSPT p50/p95/p99 et pics, avec le même monde et la même charge : séquence Nomi
   seul, puis Nomi + inférence. Convenir des seuils d'arrêt avant cohabitation.

### Conditions de décision

- Aucun statut « validé » sur la seule base de benchmarks du fournisseur.
- Gain de qualité ou de ressources démontré face à la référence, sans régression
  critique sur sujet, dates, abstention et contrôle des actions.
- Coût CPU, mémoire, énergie et effet sur les ticks compatibles avec les budgets
  fixés avant l'essai ; retrait propre du candidat possible.
- Rapport reproductible avec versions, contexte, quantification, matériel,
  mesures, réponses brutes et limites ; conclusion « retenir », « spécialiser »
  ou « écarter » selon les tâches.
- Une intégration éventuelle fera l'objet d'une étape ultérieure après cette
  qualification. Aucun achat de 2950X n'est requis pour cette étude.

### Références

- [Fiche du modèle LFM3-1.7B](https://huggingface.co/LiquidAI/LFM3-1.7B)
- [Annonce officielle LFM3](https://www.liquid.ai/blog/lfm3)

Références consultées lors de la revue du 3 novembre 2026. Les transcriptions
vidéo comportent des erreurs de noms et de benchmarks : utiliser les sources
officielles pour préparer le protocole. Environ 1 Go concerne les poids texte
en 4 bits, pas toute la mémoire d'exécution ; aucune vitesse sur X399 n'est
encore établie.
