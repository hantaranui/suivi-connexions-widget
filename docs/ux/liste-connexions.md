# Fiche de conception : liste des connexions

Statut : version consolidée par le designer, à valider par Hanta avant codage. Une seule version : cette fiche ne contient plus de proposition remplacée.
Sources : CLAUDE.md du dépôt ; design system France Travail (https://designsystem.francetravail.io/, pages Table, Alert, Variables CSS) ; feuille de style https://cdn.francetravail.fr/studio/design-system/css/styles.css (en-tête « version 1.56.0 », lue le 03/11/2026). Les pages du DS lues plus tôt portaient la version 1.16.0 : les valeurs de variables citées plus bas ont été relues dans la feuille 1.56.0 et sont identiques.
Skill UI/UX : aucun dans .claude/skills/ (dossier absent), donc aucun appliqué.
Règle : toute valeur sans source est marquée « à valider ».

## Décisions

1. **Pas de script externe** : ni `ft-autoloader.js` ni aucun autre. Chargement et erreur sont du texte avec les classes CSS du DS (pas de `<ft-spinner>` ni de `<ft-alert>`).
2. **Une seule feuille de style externe** : `https://cdn.francetravail.fr/studio/design-system/css/styles.css`, chargée comme dans le widget de la Page d'accueil. Aucun autre hébergeur (§8).
3. **Mode sombre : fond clair conservé dans le widget.** Texte et fond définis ensemble dans les deux modes de Grist (§6).
4. **Pas de résumé « X sur Y »** dans la première version.
5. **Format de date validé** : `JJ/MM/AAAA à HH:MM`, heure de Paris (Europe/Paris).
6. **Petit écran** : défilement horizontal du conteneur du tableau, à vérifier à l'écran (320 px environ, zoom 200 %, clavier).
7. **Non National, Adresse_gestion** : trois cas (vide, absente, en erreur), même affichage : « — » avec le texte caché « non disponible ». Aucun message d'erreur pour ces cas.
8. **Recommandations du designer que Hanta peut modifier** : `<caption>` visible ; Nom en `<th scope="row">` (§3). Textes en §9.

Reste à valider par Hanta : couleur de contour (§6), balise de chargement des styles (§8), textes (§9), recommandations du point 8. Ce que Grist renvoie réellement pour Adresse_gestion chez un non National reste à observer sur le document de développement, sans bloquer le codage.

## 1. Utilisateur et objectif

- Qui : le profil « FT National » (voit toutes les lignes). Les autres profils voient seulement leur propre ligne.
- Ce qu'il veut : voir d'un coup d'œil qui s'est connecté récemment et qui ne s'est jamais connecté.
- Étapes : 1. la page s'ouvre, 2. le widget charge, 3. le tableau s'affiche. Aucune action (pas de filtre, pas d'export, pas d'écriture). Lecture seule.

## 2. Données de personnes affichées et pourquoi

| Colonne Grist | Affichée | Pourquoi |
|---|---|---|
| Prenom, Nom | oui | identifier la personne |
| Profil, DR, DT | oui | situer la personne dans l'organisation |
| Adresse_gestion | oui (adresse e-mail) | pouvoir contacter la personne ; lisible seulement par les propriétaires et « FT National » |
| Derniere_connexion | oui | objectif du widget |
| Email | JAMAIS lue | règle d'accès Grist |
| Role | non affichée | sauf décision contraire |

Aucune autre colonne, aucune colonne agence. Rien n'est écrit dans la console ni envoyé ailleurs.

## 3. Maquette en texte (grand écran)

Ordre de lecture et ordre du clavier : titre de page (h1) → tableau.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Suivi des connexions                                         (h1)            │
│                                                                              │
│ Liste des utilisateurs et de leur dernière connexion        (caption)        │
│ ┌────────┬────────┬────────┬────┬────┬──────────────────┬──────────────────┐ │
│ │Prénom  │Nom     │Profil  │DR  │DT  │Adresse e-mail    │Dernière connexion│ │
│ ├────────┼────────┼────────┼────┼────┼──────────────────┼──────────────────┤ │
│ │Test    │Un      │Profil A│DR 1│DT 1│test.un@exemple.fr│03/11/2026 à 14:32│ │
│ │Test    │Deux    │Profil B│DR 2│DT 2│test.deux@exemple…│28/10/2026 à 09:05│ │
│ │Test    │Trois   │Profil B│DR 2│DT 3│—                 │Jamais connecté   │ │
│ └────────┴────────┴────────┴────┴────┴──────────────────┴──────────────────┘ │
└──────────────────────────────────────────────────────────────────────────────┘
```

Les noms, adresses, profils, DR et DT ci-dessus sont fictifs. La dernière ligne montre le cas « — » (adresse non disponible).

### Le tableau

- Un vrai `<table class="table">` dans un `<div class="table-responsive">` (structure du composant Table du DS). Pas de JavaScript du DS.
- `<caption>` **visible** (recommandation du designer, modifiable par Hanta) : « Liste des utilisateurs et de leur dernière connexion ». Raison : elle nomme le tableau pour tout le monde, et pas seulement pour les lecteurs d'écran ; le coût est une ligne de texte. Si Hanta préfère la masquer, la variante « titre masqué » du DS s'applique.
- `<thead>` avec `<th scope="col">` : Prénom, Nom, Profil, DR, DT, Adresse e-mail, Dernière connexion.
- `<tbody>` : **Nom** en `<th scope="row">` (recommandation du designer, modifiable par Hanta), les autres cellules en `<td>`. Raison : le Nom identifie la ligne ; un lecteur d'écran annonce « Nom, Adresse e-mail, valeur ». Prénom reste la première colonne visuelle en `<td>`.
- Pas de colonne « Actions ». Pas de tri cliquable en v1 : le tri est fixe.

### Tri et dates

- Tri fixe : Derniere_connexion décroissante (la plus récente en haut). Les « Jamais connecté » sont en bas. Ordre secondaire entre dates identiques : Nom puis Prénom (à valider).
- La valeur Grist est en secondes depuis 1970 : le code la multiplie par 1000.
- Format (validé) : `JJ/MM/AAAA à HH:MM`, heure d'Europe/Paris (été et hiver gérés). Exemple : « 03/11/2026 à 14:32 ».
- Valeur vide, nulle ou 0 : texte « Jamais connecté ». Le sens passe par le texte, pas par la couleur.
- Utilisation de `<time datetime="…">` : à valider avec dev-interface.

### Utilisateur qui n'est pas National

- Il voit le tableau avec une seule ligne : la sienne.
- Adresse_gestion : valeur vide, absente (clé non renvoyée) ou en erreur (lecture refusée) : le développeur gère les trois de la même façon. Affichage : « — » avec un texte caché pour lecteur d'écran « non disponible » (classe `sr-only` du DS, présente dans la feuille). Aucun message d'erreur ; les autres colonnes s'affichent normalement.
- Pas de message « vous ne voyez que votre ligne » en v1 (à valider : un court texte pourrait éviter la confusion).

## 4. États

Un seul état visible à la fois, dans une zone annoncée aux lecteurs d'écran. Chargement et erreur sont du texte avec les classes CSS du DS ; aucun web component.

| État | Ce que voit la personne | Mise en œuvre (classes du DS) |
|---|---|---|
| Chargement | Texte « Chargement de la liste… » à la place du tableau, sans animation | Paragraphe de texte dans une zone `aria-live="polite"`. Pas de spinner |
| Liste vide | « Aucun utilisateur à afficher. », sans tableau | Paragraphe de texte dans la même zone. Le DS n'a pas de composant « état vide » |
| Erreur de lecture | Titre « Impossible de lire la liste » + message d'explication. Aucun nom, aucune adresse, aucun détail technique | `div.alert.alert-error` > `div.alert-body` > `.alert-title` + `.alert-content`, avec `role="alert"`. Ces classes existent dans la feuille 1.56.0 ; la structure HTML exacte et la valeur du fond (`--ft--bg-error`, non lue) sont **à valider** |
| Normal | Le tableau | Table |

Mouvement réduit : le chargement n'a aucune animation, donc rien à désactiver.

## 5. Petits écrans

- Le tableau reste un `<table>`, pas de cartes en v1. `table-responsive` du DS donne `overflow-x:auto` sur le conteneur : seul le conteneur défile, jamais la page.
- Le conteneur est atteignable au clavier (`tabindex="0"`) avec un nom accessible (`role="region"` + `aria-labelledby` pointant vers la caption). Focus visible. À valider avec dev-interface.
- Aucune colonne masquée.
- Vérification : grand écran, petit écran (environ 320 px : valeur à valider), zoom 200 %, clavier.

## 6. Mode clair, mode sombre et couleurs

Le DS ne décrit pas de mode sombre global. Décision : le widget occupe tout son cadre avec un **fond clair du DS**, et la couleur du texte et celle du fond sont définies ensemble pour chaque élément, dans les deux modes de Grist. Aucune couleur sombre n'est inventée.

Variables du DS (feuille 1.56.0, définies sur `:root`) :

| Rôle | Variable | Valeur |
|---|---|---|
| Fond du widget (page) | `--ft-color-background-layout-body` | #ffffff |
| Texte du tableau | `--ft-color-text-content-strong` | #121212 |
| Fond des en-têtes | `--ft-color-background-neutral-weakest` | #f5f5f8 |
| **Contour du tableau et séparateurs de lignes** | `--ft-color-border-neutral-stronger` | **#888b99** |
| Texte du titre d'erreur | `--ft-color-text-error-default` | #ba251a |

| Élément | Mode clair | Mode sombre de Grist | Contraste calculé |
|---|---|---|---|
| Texte du tableau, « Jamais connecté », « — » | #121212 sur #ffffff | identique (fond clair conservé) | 18,73:1 |
| En-têtes de colonnes | #121212 sur #f5f5f8 | identique | 17,22:1 |
| Contour et séparateurs | #888b99 sur #ffffff | identique | 3,39:1 (3,11:1 sur le fond des en-têtes) |
| Titre d'erreur | #ba251a sur #ffffff | identique | 6,24:1 |
| Fond du bloc d'erreur | `--ft--bg-error` : valeur non lue | identique | **à valider** (à mesurer à l'écran) |

### Couleur du contour (point résolu)

- Le contour par défaut du DS, `--ft-color-border-neutral-default` (#cdcfd8), donne **1,55:1** sur blanc : insuffisant pour 3:1.
- Les variables de bordure neutres, mesurées sur blanc : `neutral-strong` #b5b8c5 = 1,98:1 (insuffisant) ; **`--ft-color-border-neutral-stronger` #888b99 = 3,39:1 (suffisant)** ; `neutral-strongest` #5b5d65 = 6,57:1 (plus marqué).
- **Choix : `--ft-color-border-neutral-stronger` (#888b99)**, calculé avec la formule de luminance WCAG. Atteint 3:1 sur #ffffff et sur #f5f5f8. Marqué « à valider » pour deux raisons : le DS utilise #e6e7ec / #cdcfd8 pour ses tableaux (c'est donc un écart volontaire du widget), et le rendu visuel (lignes plus marquées) doit être jugé à l'écran. Le DS pose lui-même la bordure et les couleurs de cellules : le widget doit les redéfinir avec une règle assez précise pour l'emporter.
- Si Hanta refuse ce choix : supprimer le contour extérieur et garder un simple séparateur de lignes (à valider). Le développeur ne choisit jamais lui-même une autre valeur.

Les contrastes ci-dessus sont des calculs ; ils restent à confirmer à l'écran dans Grist, en mode clair ET sombre, par le relecteur ou le recetteur.

## 7. Accessibilité RGAA

- Titre de page : un seul `<h1>` « Suivi des connexions ». Langue `fr`.
- Ordre du clavier (Tab) : le conteneur défilant du tableau est le seul arrêt. Aucun bouton, lien ou champ.
- Échap : aucune fenêtre ni menu. Sans objet.
- Focus visible sur le conteneur défilant. Style du focus : celui du DS, **valeur à valider** (non lue).
- Noms accessibles : tableau nommé par sa caption ; région défilante nommée par la caption ; l'alerte a un titre.
- Images : aucune. Pas d'icône.
- Information jamais portée par la couleur seule : « Jamais connecté » et « non disponible » sont écrits.
- Mouvement réduit : aucune animation.
- Zoom 200 % et texte agrandi : le tableau n'est ni coupé ni masqué.
- Taille du texte : `--ft-font-size-text` et `--ft-font-family-body` du DS ; aucune autre taille. (La feuille donne deux valeurs de `--ft-font-size-text`, 0.875rem et 1rem, selon la largeur d'écran ; le développeur ne choisit pas : c'est la feuille qui décide.)

## 8. Chargement des styles du design system

### Balise exacte (décision de Hanta, identique au widget de la Page d'accueil)

```html
<link href="https://cdn.francetravail.fr/studio/design-system/css/styles.css" rel="stylesheet">
```

- Placée dans le `<head>` du modèle HTML (src/), **avant** la balise `<style>` du widget.
- **Aucun** `<script>` externe, en particulier pas `ft-autoloader.js`. Aucun autre hébergeur. Les polices Marianne et les icônes sont référencées par la feuille elle-même avec des chemins relatifs, donc elles viennent du même hébergeur `cdn.francetravail.fr` : pas de nouvel hébergeur (à confirmer dans l'onglet réseau au premier essai).
- Pas d'attribut `integrity` (SRI) : le fichier est mis à jour par France Travail à son propre rythme (en-tête `Last-Modified` du 01/10/2026, cache 1 jour) et un hash bloquerait la feuille à chaque mise à jour. Risque noté en §8.
- Le serveur répond avec `Access-Control-Allow-Origin: *`, donc aucune restriction de partage entre origines pour cette feuille.

### Si la feuille ne charge pas

Le widget doit rester lisible. Le fichier HTML contient donc un bloc `<style>` de repli, écrit avec les variables du DS **et** leur valeur du DS comme valeur de secours, au format `var(--nom, valeur)`. Quand la feuille est là, les variables existent et la valeur de secours est ignorée ; quand elle est absente, la valeur de secours s'applique. Aucune valeur n'est inventée : chaque valeur vient du tableau du §6.

| Élément | Règle de repli (valeurs de secours = valeurs du DS) |
|---|---|
| Page (`body`) | fond `var(--ft-color-background-layout-body, #ffffff)`, texte `var(--ft-color-text-content-strong, #121212)`, police `var(--ft-font-family-body, system-ui, sans-serif)` (valeur de secours lue dans la feuille) |
| Tableau (`table`) | largeur 100 %, `border-collapse: collapse` |
| Cellules (`th`, `td`) | bordure 1 px `var(--ft-color-border-neutral-stronger, #888b99)`, marge intérieure `var(--ft-spacer-xs2, 0.5rem)` |
| En-têtes (`thead th`) | fond `var(--ft-color-background-neutral-weakest, #f5f5f8)`, texte `var(--ft-color-text-content-strong, #121212)`, texte aligné à gauche |
| Titre d'erreur | texte `var(--ft-color-text-error-default, #ba251a)` |

- Ces règles utilisent des sélecteurs d'éléments (sans classe) : celles du DS, basées sur les classes (`.table`, `.alert`…), l'emportent quand la feuille est chargée. En revanche, le contour du §6 doit rester prioritaire sur le DS : le développeur applique la même variable `--ft-color-border-neutral-stronger` aux classes concernées.
- Le repli ne contient pas de détection en JavaScript : il agit seul, sans script.
- Le conteneur défilant (`overflow-x: auto`) est aussi dans le repli, sinon le petit écran casserait sans la feuille.
- Texte et fond toujours définis ensemble (règle commune), y compris dans le repli.

### Ce que cela implique pour le fichier HTML assemblé

- Le fichier assemblé par build.js n'est **plus entièrement autonome** : il dépend de cdn.francetravail.fr pour le rendu complet. Si le CDN est en panne ou bloqué, le tableau s'affiche avec le repli, moins soigné mais lisible.
- La balise `<link>` est copiée telle quelle par build.js (elle n'est pas intégrée au fichier). Le bloc de repli, lui, est écrit dans le fichier.
- Un nouvel hébergeur est contacté à chaque ouverture : l'adresse de la page du widget peut apparaître dans les journaux du CDN (aucune donnée de personne n'est envoyée). Accord de Hanta donné pour cet hébergeur et la feuille, pas pour autre chose.
- Le CDN pourrait être bloqué par une politique de sécurité de la page (CSP) de Grist ou de l'entreprise : à vérifier au premier essai dans le document de développement.
- Une mise à jour du DS peut changer le rendu sans changement de notre code (la feuille est passée de 1.16.0 à 1.56.0 entre deux lectures). À revérifier à chaque recette.

## 9. Textes affichés (à valider par Hanta)

| Emplacement | Texte proposé | Remarque |
|---|---|---|
| Titre de page (h1) | Suivi des connexions | |
| Caption (visible) | Liste des utilisateurs et de leur dernière connexion | recommandation : visible |
| En-têtes de colonnes | Prénom · Nom · Profil · DR · DT · Adresse e-mail · Dernière connexion | |
| Chargement | Chargement de la liste… | texte seul, sans animation |
| Liste vide | Aucun utilisateur à afficher. | |
| Erreur, titre | Impossible de lire la liste | |
| Erreur, message | La liste n'a pas pu être chargée. Rechargez la page. Si le problème continue, contactez le chef de projet. | aucune donnée de personne |
| Date jamais renseignée | Jamais connecté | |
| Adresse non disponible | — (visible) + « non disponible » (réservé aux lecteurs d'écran) | |

## 10. Composants du design system France Travail

| Besoin | Composant / classes DS | Existe ? | Remarque |
|---|---|---|---|
| Tableau | Table : `div.table-responsive` > `table.table`, `caption`, `th scope` | Oui | Pas de JavaScript. Pas de variante en v1 |
| Chargement | — (texte simple) | Texte seul | Décision : pas de spinner |
| État vide | — | Non | Paragraphe de texte |
| Erreur de lecture | Alert : classes `alert`, `alert-error`, `alert-body`, `alert-title`, `alert-content` | Classes oui | Structure HTML exacte à valider ; le DS ne décrit que le composant `ft-alert` |
| Texte caché | `sr-only` | Oui (feuille 1.56.0) | Pour « non disponible » |

## 11. Cas à tester (liste courte)

1. National : toutes les lignes, adresses e-mail visibles, triées de la plus récente à la plus ancienne.
2. Non National : une seule ligne ; Adresse_gestion vide, absente ou en erreur : « — » + « non disponible », sans message d'erreur.
3. Date vide, nulle (0) : « Jamais connecté », placé en bas.
4. Date normale : jour et heure d'Europe/Paris corrects (été et hiver).
5. Liste vide : « Aucun utilisateur à afficher. »
6. Erreur de lecture : message clair sans donnée personnelle.
7. Chargement : texte visible puis remplacé par le tableau.
8. Petit écran : tableau lisible, défilement dans le conteneur seulement.
9. Clavier : arrêt sur le conteneur défilant, focus visible, aucun piège.
10. Mode clair et mode sombre de Grist : texte lisible, contrastes mesurés (dont contour #888b99).
11. Lecteur d'écran : en-têtes annoncés, caption lue, Nom annoncé comme en-tête de ligne.
12. Un texte contenant `<b>` dans un nom reste affiché comme texte (textContent).
13. Feuille du DS bloquée (hors ligne ou bloquée par le navigateur) : le tableau reste lisible avec le repli.

## 12. Règles pour dev-interface

- Appliquer cette fiche sans choisir d'autre valeur. Une valeur « à valider » : s'arrêter et demander à Hanta.
- Aucun script externe. Une seule feuille externe : la balise du §8.
- Texte des données inséré uniquement avec `textContent`.
- Aucune écriture dans le document, aucune lecture de Email, aucun `console.log` de données.
- Colonnes lues : Prenom, Nom, Profil, DR, DT, Adresse_gestion, Derniere_connexion (noms exacts).

## 13. Points restant à valider par Hanta

1. Contour du tableau : `--ft-color-border-neutral-stronger` (#888b99, 3,39:1), ou suppression du contour au profit d'un séparateur de lignes ?
2. Caption visible et Nom en `th scope="row"` : recommandations acceptées ?
3. Tableau des textes du §9.
4. Structure HTML de l'erreur (classes `alert`) et fond d'erreur : à valider et à mesurer à l'écran.
5. Absence de SRI sur la feuille du DS : acceptée ?
6. Ordre secondaire du tri (Nom puis Prénom) et message « vous ne voyez que votre ligne ».
