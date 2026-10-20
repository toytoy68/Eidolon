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
