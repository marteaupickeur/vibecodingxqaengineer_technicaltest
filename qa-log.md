# QA Log — Test Reduck (automatisation Snapchat)

> À remplir en continu, dès la première interaction avec reduck.ai. Une entrée par
> incident/observation, même mineure. Ne pas attendre la fin pour se souvenir — noter
> à chaud, avec les étapes exactes pour pouvoir reproduire.

## Catégories

- `SCRIPT` — bug dans un script généré par l'agent (le code lui-même est cassé)
- `SITE` — échec côté site externe (Snapchat a changé son DOM, rate-limit, captcha,
  page qui ne charge pas...)
- `IA` — mauvaise décision de l'agent (mauvaise interprétation de la page ou de la
  consigne, hallucination de sélecteur, etc.)
- `INPUT` — mauvaise entrée/consigne de ma part (prompt ambigu, mal formulé)
- `PRODUIT` — friction Reduck lui-même (UX confuse, doc manquante ou fausse, message
  d'erreur peu clair, setup qui coince, latence...)

## Gabarit d'entrée (à copier pour chaque observation)

```
### [HH:MM] Titre court de l'observation
- Catégorie : SCRIPT / SITE / IA / INPUT / PRODUIT
- Script concerné : (nom du script, ou "setup" / "n/a")
- Action tentée : ...
- Résultat attendu : ...
- Résultat obtenu : ...
- Reproductible : oui/non
  - Étapes de repro (si oui) :
    1. ...
    2. ...
- Sévérité perçue : bloquant / gênant / cosmétique
- Note libre : ...
```

## Journal

```
### [23/09 à 11:00] Plusieurs onglet snapchat.com/web
- Catégorie : SITE
- Script concerné : snap-story-analytics
- Action tentée : Ouverture d'un nouvel onglet de snapchat.com/web alors qu'il avait déjà un d'ouvert
- Résultat attendu : le script devait normalement continuer de dérouler
- Résultat obtenu : Message de snapchat : plusieur onglets ouvert à la fois
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Ouverture manuelle de snapchat.com/web
    2. lancer le script snap-story-analytics qui déclenche une autre ouverture de la même page
- Sévérité perçue : gênant
- Note libre : N\A
```

```
### [23/09 à 11:46] Decouverte de https://profile.snapchat.com/
- Catégorie : SITE
- Script concerné : snap-story-analytics
- Action tentée : Recueil de stats des story/spotlight en echec sur snapchat.com/web
- Résultat attendu : 
    avoir des stats détaillés de mes story/spotlight : Date de création, Spectateurs·rices, Vues, Captures d'écran, Interactions, Statut de modération 
- Résultat obtenu : l'heure du post uniquement
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. lancer le script snap-story-analytics (version sans la redirection vers https://profile.snapchat.com/) qui va recueillir que la date de post du story
- Sévérité perçue : bloquant
- Note libre : Revirement de situation, j'ai faillit abandonné ce script avant de prendre connaissance de https://profile.snapchat.com/ et mettre en place un nouveau script post-reminder (un rappel pour poster à nouveau) qui se baserait uniquement sur la date de post du story
```

```
### [23/09 à 12:20] snap-story-analytics reconstruit sur profile.snapchat.com — promu
- Catégorie : SCRIPT
- Script concerné : snap-story-analytics
- Action tentée : Reconstruction du script sur https://profile.snapchat.com/ (au lieu de
  web.snapchat.com), extraction du tableau "Informations sur le Snap" (Date de création,
  Spectateurs·rices, Vues, Réponses, Captures d'écran, Interactions, Statut de modération).
- Résultat attendu : JSON par Snap avec toutes ces stats, reproductible à froid.
- Résultat obtenu : Fonctionne. Bug intermédiaire trouvé et corrigé : le chargement de la
  liste de Stories est asynchrone (~9,5s à chaud) et dépassait le timeout par défaut de
  10s en session froide — le script concluait alors à tort "aucune Story active" au lieu
  de renvoyer le Snap réel. Corrigé avec `page.setDefaultTimeout(20000)`. Reproduit 2x à
  froid après le fix, résultat stable.
- Reproductible : oui (le bug de timeout l'était aussi, avant le fix)
  - Étapes de repro (si oui) :
    1. Créer un script qui attend le sélecteur de tuile de Story sur profile.snapchat.com
       sans relever le timeout par défaut.
    2. Le lancer en session déjà "chaude" (même navigateur venant de naviguer sur le site)
       → passe.
    3. Le lancer en session fraîche (nouveau run_script sans session ouverte au préalable)
       → time out à 10s, retourne un tableau vide au lieu du vrai contenu.
- Sévérité perçue : bloquant (avant le fix — aurait fait croire à tort à un compte sans
  Story active)
- Note libre : Champ non prévu dans le prompt v1 : le tableau expose aussi "Réponses"
  (replies), en plus de viewers/views/screenshots/interactions/moderation_status. Le
  "story_id" retourné est dérivé de l'URL CDN de la miniature (pas d'ID explicite affiché
  sur la page). "Statut de modération" est vide/null pour ce Snap de test — pas encore vu
  de valeur non vide pour confirmer le format. Un seul Snap disponible sur le compte de
  test à ce stade — les cas à plusieurs Stories/Snaps n'ont pas pu être vérifiés
  empiriquement (le code boucle sur toutes les tuiles trouvées, mais c'est non testé).
```

```
### [23/09 à 13:35] snap-story-analytics étendu à Contenus Spotlight — v2 promue
- Catégorie : SCRIPT
- Script concerné : snap-story-analytics
- Action tentée : Ajout de la couverture de l'onglet "Contenus Spotlight" (repéré
  après coup, manquant dans la v1 du script), en plus de "Stories publiques". Le
  script retourne maintenant `{ stories: [...], spotlights: [...] }`.
- Résultat attendu : `spotlights` avec les stats affichées dans l'UI (Vues,
  Spectateurs·rices, Likes, Partages) par Spotlight actif.
- Résultat obtenu : Fonctionne, reproduit 2x à froid. Différences structurelles
  notables trouvées en construisant :
  - Les tuiles Story ET Spotlight partagent le même test-id
    (`app.brandProfiles.story.tile`) — il faut être sur le bon onglet au moment du
    clic pour savoir ce qu'on ouvre, le test-id seul ne suffit pas à distinguer.
  - Cliquer une tuile Spotlight ouvre une modale in-page (pas de navigation, pas
    d'URL dédiée) avec un panneau "Statistiques Spotlight" — contrairement à une
    tuile Story qui navigue vers une page dédiée avec un tableau. Extraction et
    fermeture (touche Echap) donc différentes des Stories.
  - Bug de course trouvé pendant la construction : juste après l'ouverture de la
    modale, le poster de la vidéo (utilisé pour dériver `spotlight_id`) n'est pas
    encore présent dans le DOM — il faut attendre spécifiquement `video[poster]`,
    pas seulement le conteneur de la modale, sinon `spotlight_id` sort `null` de
    façon aléatoire.
- Reproductible : oui (le bug de course sur `spotlight_id` aussi, avant le fix)
- Sévérité perçue : gênant (le bug de course produisait un `spotlight_id` null de
  façon silencieuse, sans erreur)
- Note libre : Un seul Spotlight disponible sur le compte de test — le cas
  plusieurs Spotlights (boucle sur les tuiles) n'a pas pu être vérifié
  empiriquement, comme pour les Stories. Pas de colonne "date de publication"
  visible dans le panneau Spotlight (contrairement à Stories) — confirmé absent,
  pas juste non extrait.
```

```
### [23/09 à 19:00] snap-unread-digest : caméra qui s'active au chargement de web.snapchat.com + aucun échantillon non lu
- Catégorie : SITE
- Script concerné : snap-unread-digest
- Action tentée : Ouverture de web.snapchat.com en session Reduck pour inspecter la
  liste de conversations (lecture seule, aucun clic sur une conversation).
- Résultat attendu : Voir la liste de conversations avec un ou plusieurs éléments non lus.
- Résultat obtenu :
  - Le panneau caméra de Snapchat Web s'est ouvert de lui-même au chargement, avec
    l'aperçu webcam en direct (aucun clic de ma part). À garder en tête : tout script
    qui charge web.snapchat.com peut allumer la webcam de la machine.
  - Les 3 conversations présentes (My AI, Momar Diagne, Team Snapchat) sont toutes à
    l'état "Reçu" (déjà ouvertes) — aucun élément non lu à observer, donc impossible
    de valider comment l'état non lu est encodé dans le DOM.
  - Structure relevée : pas de data-testid sur web.snapchat.com ; chaque conversation
    est un `role="listitem"` avec un titre, un libellé de statut (français : "Reçu") et
    un `<time datetime="...">` ISO exploitable pour `received_at`.
- Reproductible : oui
- Sévérité perçue : bloquant pour la validation (pas d'échantillon non lu)
- Note libre : Il faut "seeder" le compte dédié depuis le second compte de test :
  envoyer au moins 1 snap photo non ouvert + 1 message texte non lu (idéalement aussi
  un snap vidéo), sans les ouvrir côté compte dédié.
```

```
### [23/09 à 19:10] snap-unread-digest : brouillon validé sur 3 échantillons non lus, en attente de promotion
- Catégorie : SCRIPT
- Script concerné : snap-unread-digest (web.snapchat.com)
- Action tentée : Construction après seeding du compte dédié depuis le second compte :
  snap+chat, chat seul, snap photo seul (un état à la fois, ouverts à la main entre
  chaque pour repasser en "Reçu").
- Résultat attendu : liste des conversations non lues, sans rien ouvrir.
- Résultat obtenu : Brouillon fonctionnel, 3 runs (1 en session, 2 à froid) identiques,
  la conversation est restée "Nouveau Snap" après les runs (rien n'a été ouvert).
  Points trouvés en construisant :
  - Aucun data-testid sur web.snapchat.com, classes hachées : l'état non lu est lu
    structurellement (libellé de statut enveloppé dans un <span> de plus) et le type
    par la forme de l'icône de droite (carré = snap, contour de bulle = chat, icône
    remplie multi-tracés = mixte), pas par le texte français.
  - Le libellé de statut est en français ("Nouveau Snap", "Nouveau chat", "Nouveau :
    Snaps et Chats") : renvoyé tel quel dans `status_text`.
  - Les lignes arrivent au fil de l'eau après le chargement : lire la première ligne
    apparue donne une liste incomplète, d'où une attente "nombre de lignes stable".
    Le chargement prend ~9,5s (proche du timeout par défaut de 10s, comme sur
    profile.snapchat.com) : timeout relevé à 20s.
- Reproductible : oui
- Sévérité perçue : gênant
- Note libre : Limites non vérifiées : états "envoyé/ouvert", liste virtualisée avec
  beaucoup de conversations (seules les lignes rendues sont vues, d'où
  `conversations_scanned`), et le cas "mixte" repose sur un seul échantillon. La
  classification du type est une inférence sur la forme de l'icône, susceptible de
  casser si Snapchat la redessine. Promotion en attente de ta validation (cf.
  prompt : "vérifie avec moi avant de promouvoir").
- Ajout [24/09] snap vidéo : le snap vidéo a le MÊME libellé ("Nouveau Snap") et la
  MÊME icône (carré) qu'un snap photo ; seule la couleur change (violet
  rgb(160, 93, 205) pour la vidéo, rouge pour la photo). Le script distingue donc
  `snap_photo` / `snap_video` par la teinte de la couleur calculée du statut. Vidéo
  vérifiée (3 runs, dont 2 à froid, tous `snap_video`). La couleur de la photo n'a
  été vue qu'à l'écran, sa valeur rgb n'a pas été relevée.
- Promotion [24/09] : validée par l'utilisateur, version promue (v1). Les types
  photo, vidéo, chat et snap+chat ont chacun été observés ; le cas snap+chat l'a été
  avant les derniers changements du script et n'a pas été rejoué depuis.
- Ajout [24/09] retest snap photo : nouvel échantillon photo non ouvert, 2 runs à
  froid, tous deux `snap_photo` (received_at 19:33:06Z, distinct de l'échantillon
  vidéo). Les deux types de snap sont donc vérifiés avec la version à teinte. La
  valeur rgb de la photo n'a toujours pas été relevée (seuls les résultats du
  script l'attestent).
```

```
### [23/09 à 22:10] snap-friend-requests-triage : brouillon validé sur 2 demandes réelles, en attente de promotion
- Catégorie : SCRIPT
- Script concerné : snap-friend-requests-triage (web.snapchat.com)
- Action tentée : Construction du script sur le panneau "Ajouter des amis·es" (bouton de
  l'en-tête), section "Ils m'ont ajouté·e". Le compte dédié avait déjà 2 demandes en
  attente (relation affichée : "Dans Mes Contacts"), pas de seeding nécessaire cette fois.
- Résultat attendu : liste des demandes, sans jamais accepter/refuser/bloquer.
- Résultat obtenu : Brouillon fonctionnel, 3 runs (1 en session, 2 à froid) identiques.
  Preuve qu'aucune action n'a été exécutée : la trace ne contient qu'un seul clic (le
  bouton de l'en-tête qui ouvre le panneau), et les 2 demandes sont toujours en attente
  après les 3 runs (encore renvoyées par le run suivant, et visibles avec "Accepter" à
  l'écran).
  Points trouvés en construisant :
  - Aucun data-testid, classes hachées : les lignes sont repérées par la structure (cellule
    de grille contenant un bouton et une icône de rejet), les champs par l'ordre des
    <span> (nom affiché, pseudo, relation).
  - Le bouton de l'en-tête n'a qu'un `title` localisé ("Voir les demandes d'amis·es") :
    choisi par sa position dans la barre du haut, avec une garde qui lève une erreur au
    lieu de cliquer ailleurs. Cette garde a d'ailleurs échoué au premier essai (bouton pas
    encore rendu au moment de l'évaluation) sans rien cliquer, puis corrigée en attendant
    le bouton. Comportement voulu : échouer bruyamment.
  - Le champ "amis en commun" du prompt n'existe pas tel quel : la ligne n'affiche qu'un
    libellé de relation ("Dans Mes Contacts"). Renvoyé tel quel dans `relation_label` ;
    `mutual_friends` reste null (jamais vu non nul).
- Reproductible : oui
- Sévérité perçue : gênant
- Note libre : Non observés : panneau vide (aucune demande), ligne sans avatar (donc
  `has_avatar: false` non vérifié), ligne avec compteur d'amis en commun, plus de lignes
  que n'en affiche le panneau (liste virtualisée, d'où `requests_scanned`). Le cas "aucune
  demande" est le plus risqué : sans marqueur de liste vide, le script attend 8 s avant de
  rendre []. `section_label` est renvoyé pour qu'un appelant vérifie que les lignes sont
  bien sous "Ils m'ont ajouté·e" (si le panneau affichait des suggestions à la place, elles
  ne seraient pas distinguables autrement).
- Promotion [23/09] : validée par l'utilisateur après comparaison du JSON avec l'écran,
  version promue (v1). Section 3 documentée dans `script-prompts-v3.md`.
```

```
### [23/09 à 22:30] snap-story-analytics : identifiants illisibles remplacés par un libellé (v3 promue)
- Catégorie : INPUT
- Script concerné : snap-story-analytics
- Action tentée : Retour utilisateur : `story_id` et `spotlight_id` n'étaient pas
  parlants (fragments de l'URL CDN de la miniature, type `87vOtLxZBD08f74nDYwTK`).
- Résultat attendu : un identifiant lisible par un humain.
- Résultat obtenu : Remplacés par `label` = date imprimée sur la tuile + rang, par ex.
  `23/09/2026 #1` (choix de l'utilisateur parmi 3 options). Testé à froid 2x, avec pour la
  première fois plusieurs éléments (2 Stories, 2 Spotlights le même jour) : libellés
  distincts, boucle multi-tuiles enfin validée en conditions réelles.
- Reproductible : n/a
- Sévérité perçue : gênant
- Note libre : Compromis assumé : le rang bouge quand du contenu est ajouté ou retiré,
  le `label` n'est donc plus une clé stable (l'ancien hash l'était). Correction en cours de
  route : j'avais écrit "du plus récent au plus ancien", or la Story #1 (11:43) est plus
  ancienne que la #2 (13:56) : l'ordre est celui de la page, plus ancien d'abord pour les
  Stories. Le format de la date de tuile suit la locale du site (motif de chiffres). La
  lacune notée plus haut sur le cas "plusieurs éléments" est levée pour Stories et
  Spotlights ; le cas "aucun élément" reste non observé. Prompt mis à jour dans
  `script-prompts-v4.md` (section 1).
```

### Setup

(à remplir dès la création du compte reduck.ai, l'installation de l'extension, le
pairing, et `claude mcp add reduck`)

### Reconnaissance manuelle Snapchat

(à remplir pendant l'exploration à la main du compte dédié, avant de lancer l'agent)

### Construction des scripts

(une sous-section par script : `snap-story-analytics`, `snap-unread-digest`,
`snap-friend-requests-triage`)

## Synthèse (à remplir en fin de test, pour le QA report)

- Ce qui a bien marché :
- Ce qui a été cassé / confus :
- Frictions produit récurrentes (pattern qui revient plusieurs fois) :
- Reconnaissance explicite du risque de ban Snapchat et mesures prises (compte dédié,
  rythme lent) :
  - Le risque est reconnu : automatiser un compte Snapchat va à l'encontre de l'usage
    normal du service et peut mener à une restriction ou un bannissement. D'où un compte
    dédié, jamais le compte personnel.
  - Mesure à appliquer entre les lancements : simuler une utilisation normale du compte
    (ouvrir l'appli ou le site à la main, consulter ses conversations et ses Stories,
    laisser passer du temps) entre deux lancements de scripts, au lieu de les enchaîner en
    rafale. Objectif : que l'activité du compte ne ressemble pas à une suite de passages
    automatiques réguliers.
  - À noter honnêtement pour le QA report : pendant la construction des scripts, plusieurs
    runs ont été lancés à la suite sur le même compte (souvent 3 ou plus par script :
    essai en session puis 2 runs à froid, plus les relevés de découverte). Cette mesure
    n'était donc pas appliquée de façon systématique pendant la phase de test ; elle vaut
    surtout pour l'usage courant des scripts une fois promus.
