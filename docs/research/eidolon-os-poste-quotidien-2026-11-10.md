# Étude : Eidolon-OS comme poste quotidien
Date : 10 novembre 2026  
Statut : propositions à étudier, enregistrées à la demande de toytoy.  
Périmètre : bureau, jeux, intégration Eidolon et récupération après mise à jour.
Aucune migration, installation ou modification du démarrage n'est réalisée par cet ajout.

## 1. Objectif

Préparer une expérience quotidienne cohérente : ouvrir une session, utiliser
Eidolon, naviguer, discuter et jouer avec peu de réglages manuels. Bazzite sert
de référence d'intégration et de comparaison, sans décision de remplacer la base
Debian d'Eidolon-OS.

La vidéo motivant cette étude montre une expérience utilisateur sur une ancienne
configuration Intel avec Radeon RX 5600 XT. Elle ne valide ni les performances
sur NVIDIA ni la compatibilité des jeux et mods personnels de toytoy.

## 2. Propositions retenues pour étude

| Axe | Proposition | Preuve attendue avant adoption |
|---|---|---|
| Bureau | Profil de poste Eidolon reproductible : applications, raccourcis, audio, notifications et connexion à Core | Installation reproductible et scénarios usuels sans terminal |
| Jeux | Intégrer les lanceurs utiles, les versions de compatibilité et des profils par jeu | Partie réelle, sauvegarde et périphériques validés pour chaque jeu |
| Pilotes | Détecter la configuration et proposer une combinaison pilote/noyau testée | Affichage, accélération, stabilité et reprise après veille vérifiés |
| Mises à jour | Étudier une mise à jour préparée avec possibilité de revenir au système précédent | Retour arrière après panne simulée, sans perte des données utilisateur |
| Compatibilité | Tableau daté par jeu, version, moteur et configuration | Distinguer lancement, fonctionnement partiel et validation complète |
| Ressources | Raccorder ultérieurement le gestionnaire matériel aux priorités du poste | Jeux et interface réactifs avec services Eidolon actifs |

Les services IA n'ont pas à être tous exécutés sur le poste. L'interface locale
peut rejoindre les services du réseau et doit signaler clairement leur absence.
Ne pas imposer le démarrage des hôtes de calcul à chaque ouverture de session.

L'intégration suit la séparation existante : le LLM propose, les composants
déterministes contrôlent les actions et Darme applique les autorisations.
Un jeu ou un lanceur ne doit pas obtenir les privilèges d'administration d'Eidolon.

## 3. Choix de base à comparer

| Option à étudier | Intérêt | Coût ou limite à mesurer |
|---|---|---|
| Debian avec profil bureau Eidolon | Continuité avec Eidolon-OS et réutilisation du travail existant | Intégration des jeux/pilotes et récupération système à construire ou assembler |
| Bazzite comme poste client Eidolon | Référence intégrée pour les jeux et retour à un déploiement système précédent | Adaptation des composants Eidolon au modèle de maintenance de cette base |
| Base gérée par images, personnalisée pour Eidolon | Déploiements reproductibles et versionnement de l'ensemble système | Maintenance des images, signatures, pilotes et procédure de récupération |

Commencer par comparer les deux premières options. La troisième reste une piste
ultérieure si le bénéfice justifie la maintenance. Ne pas transposer les outils
de retour arrière de Bazzite directement sur Debian sans conception spécifique.

## 4. Mises à jour et récupération

Bazzite documente le retour à un déploiement antérieur. Pour Eidolon, l'objectif
est de tester une propriété équivalente, pas d'annoncer un mécanisme déjà acquis.

Séparer trois ensembles :
1. système de démarrage, noyau, pilotes et applications système ;
2. applications/services Eidolon, versions de configuration et schémas de données ;
3. données utilisateur, sauvegardes de jeux, documents et éventuels disques de VM.

Une réversion du système n'annule pas nécessairement une migration de base de
données. Prévoir compatibilité ascendante/descendante ou restauration cohérente
de l'application et des données concernées. Sauvegardes indépendantes conservées.

Recette : démarrage normal, mise à jour réussie, échec de démarrage, pilote
graphique défaillant, espace disque insuffisant et service Eidolon incompatible.
Vérifier le démarrage de secours, la restauration et la conservation des fichiers.
Les simulations destructives se font dans un environnement jetable.

## 5. Secure Boot et Fortnite : état documentaire

Sources officielles relues le 10 novembre 2026 :
- Bazzite documente Secure Boot avec enregistrement de la clé Universal Blue.
  Le désactiver durablement n'est pas une règle générale d'installation.
- Epic a annoncé le 14 octobre 2026 EAC Linux v2 et un pilote prévu dans Fortnite.
  L'annonce consultée décrit une arrivée future, pas une disponibilité validée
  de Fortnite sur Eidolon-OS ou Bazzite.
- Les exigences annoncées comprennent notamment noyau/modules signés avec des
  clés reconnues, TPM 2.0, Secure Boot, measured boot et protections du noyau.
  Elles restent susceptibles d'évoluer.

Hypothèse d'architecture : personnaliser le bureau et les services sur une base
compatible serait plus simple à qualifier que maintenir un noyau profondément
modifié. Ce n'est pas une garantie de compatibilité anti-triche.
Une signature personnelle ou Secure Boot activé ne suffisent pas à prouver
l'acceptation par Epic ; vérifier la chaîne complète et les exigences publiées.

Avant toute expérimentation réelle, vérifier disponibilité, distributions prises
en charge, exigences du pilote NVIDIA et fonctionnement du jeu. Conserver une
solution Windows utilisable tant que les jeux indispensables ne sont pas validés.
Le suivi est une tâche d'étude à reprendre ; aucune nouvelle veille automatique
n'est créée par cette note.

## 6. Matrice de qualification du poste

Cible connue d'après les échanges : PC principal « mjolnir », i7-11700K,
64 Go de RAM et RTX 3080 Ti MSI à trois ventilateurs. Relever la configuration
réelle avant essai, notamment écrans, résolutions et fréquences ; ne pas utiliser
les mesures de la 3080 Ti FE du banc X399 comme référence directe.

| Usage | Vérifications |
|---|---|
| Bureau quotidien | Navigateur, documents, audio, micro, périphériques et notifications |
| Plusieurs écrans | Disposition, fréquences différentes, plein écran, retour bureau, veille/réveil |
| Eidolon | Connexion, voix, autorisations, comportement en cas d'indisponibilité de Core |
| Minecraft / Nomi | Lanceur, Java, mods, connexion au serveur, stabilité et sauvegardes |
| Skyrim SE moddé | Copie du profil de mods, outils utilisés, chargement des sauvegardes, stabilité longue |
| Fortnite | Disponibilité et support officiels, puis partie réelle ; bloquant si indispensable et non validé |
| Jeux secondaires | Satisfactory, Palworld ou autres jeux effectivement retenus par toytoy |

Ne pas confondre compatibilité du lanceur, du jeu, de ses mods et de l'anti-triche.
Le lancement d'une fenêtre n'est pas une validation complète.

## 7. Protocole comparatif

- Même machine, jeu, sauvegarde, parcours, résolution et réglages graphiques.
- Consigner versions du système, noyau, pilote, jeu et couche de compatibilité.
- Fixer V-Sync, limite d'images et paramètres d'upscaling ; les rendre identiques.
- Séparer premier lancement/compilation/chargement et essais à chaud.
- Répéter au moins trois fois, alterner l'ordre des systèmes et mesurer FPS moyens,
  1 % low, temps de trame p95/p99, saccades, CPU/GPU, RAM/VRAM et températures.
- Mesurer consommation secteur uniquement si une prise est réellement associée
  au poste ; ne pas attribuer les mesures du Shelly X399 à mjolnir.
- Comparer le jeu seul puis le jeu avec navigateur, communication et Eidolon actifs.
- Mesurer aussi veille/réveil, temps de récupération et nombre d'actions utilisateur.
- Fixer les seuils acceptables par jeu avant les essais, selon l'écran et les
  attentes de toytoy ; aucune promesse générale de gain de FPS.

La remontée des FPS dans un menu ne permet pas d'exclure une limitation CPU/GPU :
une simulation mise en pause change la charge. Les chiffres de la vidéo ne
remplacent donc pas cette recette comparative.

## 8. Déroulement proposé

1. Inventaire et liste des usages indispensables ; tableau de compatibilité initial.
2. Maquette d'interface et installation reproductible en environnement isolé.
   Une VM permet de tester les parcours, pas de conclure sur les performances GPU.
3. Essais matériels sur disque dédié avec sauvegardes vérifiées et accès Windows
   préservé ; documenter le démarrage et le retour à la configuration initiale.
4. Qualification des jeux, périphériques, services et mises à jour.
5. Comparaison Debian/Bazzite et décision motivée : retenir, adapter ou reporter.
6. Migration du poste uniquement après validation des usages indispensables et
   de la récupération. Aucun changement de l'hôte X399 requis pour cette étude.

Lien avec [l'étude de gestion matérielle](gestion-materielle-eidolon-2026-11-08.md) :
prioriser l'interactivité, réserver les ressources et mesurer les effets de
cohabitation. Les profils du poste joueur et de l'hôte de calcul restent distincts.

## 9. Sources

- [Documentation Bazzite](https://docs.bazzite.gg/) : intégration et retour aux déploiements précédents.
- [Secure Boot Bazzite](https://docs.bazzite.gg/General/Installation_Guide/secure_boot/) : procédure officielle.
- [Epic : EAC Linux v2, annonce du 14 octobre 2026](https://onlineservices.epicgames.com/news/introducing-easy-anti-cheat-linux-v2).
- [Vidéo discutée](https://www.youtube.com/watch?v=mLHLNYv_zk4) : retour d'expérience, pas preuve de compatibilité du poste cible.

Ces propositions sont enregistrées en étude. Aucun résultat d'essai local,
aucune migration vers Bazzite et aucune compatibilité Fortnite ne sont déclarés validés.
