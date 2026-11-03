
# Projet : Suivi des connexions (widget Grist)

## Objectif

Widget personnalisé Grist pour le Partenariat simplifié de France Travail. Il affiche la liste des utilisateurs référencés et leur date de dernière connexion, pour que le profil FT National voie si l'outil est bien utilisé. Le widget ne fait que LIRE : il n'écrit jamais dans le document. La date est écrite par le bandeau commun (autre dépôt), dans la colonne Derniere_connexion.

## Technologies

HTML, CSS et JavaScript sans framework, aucune dépendance npm sans mon accord. Un seul fichier HTML assemblé à partir de src/ par build.js, publié sur GitHub Pages et chargé par Grist. Tests avec le lanceur de tests de Node. Design system France Travail (https://designsystem.francetravail.io/). Les commandes de build et de test seront créées au premier sprint de code : ne pas les inventer avant.

## Données (table Utilisateurs, document Grist)

Colonnes, noms exacts : Prenom, Nom, Email, Profil, Role, DR, DT, Adresse_gestion, Derniere_connexion. Il n'existe pas de colonne agence : ne pas en inventer.

- Le widget affiche : Prenom, Nom, Profil, DR, DT, Adresse_gestion (adresse e-mail) et Derniere_connexion.
- Email n'est JAMAIS lue : une règle d'accès Grist cache l'e-mail de chacun sauf le sien (le bandeau en dépend). Role n'est pas affichée sauf décision contraire.
- Adresse_gestion est une colonne de formule, lisible seulement par les propriétaires et par le profil « FT National » (écrit exactement ainsi). Les autres profils ne voient que leur propre ligne.
- Derniere_connexion : type Date et heure. Valeur vide, nulle ou 0 : afficher « Jamais connecté ». Valeur en secondes depuis 1970 (convention de Grist).
- Ne fais aucune hypothèse sur d'autres colonnes ou tables : demande-moi.

## Décisions de la première version

- Tri par dernière connexion, la plus récente en premier ; « Jamais connecté » pour les dates vides.
- Pas de filtre, pas d'export CSV (un export de données personnelles demande l'accord du chef de projet).
- Le widget n'affiche pas le bandeau : sur la page, le bandeau est dans un autre widget (bandeau-seul.html du dépôt du bandeau), le widget de suivi est en dessous.
- Accès au document demandé : le plus faible qui suffit pour lire la table, jamais d'écriture. À confirmer par le développeur avec la documentation de Grist.
- À valider avec le chef de projet : affichage des e-mails ; résumé « X personnes sur Y se sont connectées » (non ajouté sans mon accord).

## Commandes

- Lancer l'application : à créer. Serveur local de test possible avec python3 -m http.server [port], puis widget personnalisé dans le document Grist de développement.
- Lancer les tests : à créer (lanceur de tests de Node).
- Assembler : à créer (build.js). Après chaque changement de src/ : build, tests, puis commiter les sources ET le fichier généré.

## Mode de travail : PAS À PAS (ne jamais changer)

- Je suis débutante en code. Explique simplement, en français.
- Un seul agent travaille à la fois : celui que j'appelle avec @.
- Un agent ne lance JAMAIS un autre agent de lui-même.
- Fais uniquement la tâche demandée. N'ajoute rien de non demandé.
- Si tu veux t'écarter de la demande ou d'un document fourni, dis-le AVANT d'agir et attends mon accord.
- Le designer décide (fiche dans docs/ux/), le développeur applique sans choisir d'autre valeur, le relecteur vérifie. Une valeur qui semble fausse : s'arrêter et me le signaler.
- À la fin de chaque tâche, ARRÊTE-TOI et termine par :
  1. Fichiers lus
  2. Fichiers modifiés ou créés
  3. Tests lancés et résultat (ou "aucun test lancé")
  4. Risques restants
- Ne fais jamais de commit ni de push : c'est moi qui les fais.

## Interface : mode clair, mode sombre, accessibilité (RGAA)

- Toute interface fonctionne dans le mode clair ET le mode sombre de Grist. Toujours définir ensemble la couleur du texte et celle du fond, jamais l'une sans l'autre.
- Design system France Travail imposé : utiliser ses composants et ses variables. Ne jamais inventer de couleur, de police, de taille, d'icône ni de composant. Une valeur sans source est marquée « à valider ».
- RGAA : contraste d'au moins 4,5:1 (texte normal) et 3:1 (grand texte, contours) ; tout se fait au clavier (Tab, Entrée, Échap) ; focus toujours visible ; chaque bouton a un nom accessible ; titres de page ; mouvement réduit respecté. Un tableau de données utilise un vrai  avec en-têtes de colonnes.
- États à prévoir : chargement, liste vide, erreur de lecture (message clair, sans donnée personnelle).
- Chaque écran est vérifié sur grand écran, sur petit écran et au clavier.

## Sécurité et données personnelles (RGPD)

- Aucun secret (mot de passe, clé, token) dans le code : uniquement dans .env.
- Ne lis pas et n'affiche pas le contenu de .env.
- Aucune donnée personnelle réelle : uniquement des données de test fictives (exemple : test.un@exemple.fr). Toute ligne de test créée est supprimée après le test.
- Ce widget affiche des noms, des adresses e-mail et des dates de connexion : ne collecter et n'afficher que le nécessaire ; ne jamais écrire ces données dans la console ni les envoyer à un autre site. Toute nouvelle collecte ou nouvelle colonne affichée est signalée avant d'être codée.
- Tout texte venant des données est inséré comme du texte (textContent), jamais comme du code.
- Aucun script ni fichier chargé depuis un nouvel hébergeur sans mon accord.
- Le widget ne contient aucune écriture dans le document (pas d'applyUserActions).
- @auditeur-securite relit avant toute publication, et toujours avant la première fusion dans main d'un dépôt public.

## Production et publication

- main est ce qui est servi en production : on ne fusionne jamais dans main pour tester. La version de test est publiée dans un dépôt de test séparé (remote « preview »), utilisé par le document TEST. main ne reçoit que ce qui a été validé par le chef de projet.
- Après une fusion dans main, attendre jusqu'à 10 minutes (GitHub Pages) avant de vérifier.
- En PROD, les règles d'accès Grist sont posées par le chef de projet.

## Tests

- Tests automatiques : à créer, lancés après chaque changement de code. « Tests proposés » ne veut pas dire « tests passés ».
- Tests humains : le recetteur écrit un petit carnet à la fin de CHAQUE fonction, avant le commit, pas tout à la fin.
- Dans un carnet, le résultat obtenu décrit ce qui a été vu et où, jamais seulement « réussi ». Chaque test indique le commit testé. Un test = un comportement.
- Tests sur le document de développement ou de test, jamais en production.
- Cas à prévoir : National (toutes les lignes, e-mails visibles), régional (sa seule ligne), date vide, liste vide, erreur de lecture, petit écran, clavier, mode sombre.

## Git et journal

- Toujours une branche (feature/..., fix/..., chore/..., docs/...), jamais de commit direct sur main.
- Messages de commit : feat:, fix:, test:, docs:, chore:
- REVIEW_LOG.md : une fiche par tâche ; après chaque commit, son numéro est ajouté dans la ligne « Commit ».
- Dans ce dépôt, la squad et REVIEW_LOG.md sont commités.

## Conventions

- Noms de fichiers et de variables en anglais, commentaires en français.
- Les noms de colonnes Grist s'écrivent exactement comme dans la table (Prenom, Derniere_connexion…).
