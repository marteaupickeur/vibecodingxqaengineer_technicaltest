# Log — Test Reduck (automatisation Snapchat)

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
### [JJ/MM] Titre court de l'observation
- Catégorie : SCRIPT / SITE / IA / INPUT / PRODUIT
- Script concerné : (nom du script, ou "setup" / "n/a")
- Action tentée : ...
- Résultat obtenu : ...
- Reproductible : oui/non
  - Étapes de repro (si oui) :
    1. ...
    2. ...
- Note libre : ...
```

## Journal

```
### [23/09] Plusieurs onglet snapchat.com/web
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
### [23/09] Decouverte de https://profile.snapchat.com/
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
### [23/09] Timeout par défaut trop court sur profile.snapchat.com
- Catégorie : SCRIPT
- Script concerné : snap-story-analytics
- Action tentée : Script reconstruit sur profile.snapchat.com.
- Résultat obtenu : La liste de Stories met ~9,5 s à charger, le timeout par défaut est de 10 s. Testé en session déjà "chaude" ça passait ; en session fraîche le script concluait à tort "aucune Story active" (tableau vide). Corrigé avec un timeout de 20 s, reproduit 2x à froid.
- Reproductible : oui (uniquement à froid, avant le fix)
- Note libre : Le tableau expose aussi "Réponses", absent du prompt v1. "Statut de modération" est toujours vide : format jamais vu.
```

```
### [23/09] Spotlight, structure différente des Stories
- Catégorie : SCRIPT
- Script concerné : snap-story-analytics
- Action tentée : Ajout de l'onglet "Contenus Spotlight", oublié dans la première version.
- Résultat obtenu : Fonctionne, reproduit 2x à froid. Trois différences avec les Stories :
  - les tuiles Story et Spotlight partagent le même test-id : seul l'onglet actif dit ce qu'on ouvre ;
  - un Spotlight s'ouvre dans une modale (pas de page dédiée, fermée par Echap) ;
  - l'identifiant du Spotlight (lu sur l'image de la vidéo) sortait parfois `null`, car l'image n'est pas encore là à l'ouverture de la modale. Corrigé en attendant cette image.
- Reproductible : oui (avant le fix)
- Note libre : Aucune date de publication dans le panneau Spotlight (confirmé absent).
```

```
### [23/09] Webcam allumée au chargement, aucun non lu pour valider
- Catégorie : SITE
- Script concerné : snap-unread-digest
- Action tentée : Lecture seule de la liste de conversations sur web.snapchat.com.
- Résultat obtenu : Le panneau caméra s'ouvre tout seul avec l'aperçu webcam en direct, sans aucun clic : tout script qui charge cette page peut allumer la webcam. Les 3 conversations étaient toutes déjà lues, donc rien pour observer l'état non lu.
- Reproductible : oui
- Note libre : Il faut envoyer au compte dédié un snap photo, un snap vidéo et un message texte depuis un second compte, sans les ouvrir. Aucun data-testid sur web.snapchat.com.
```

```
### [23/09] Lecture de l'état non lu par la structure, pas par le texte
- Catégorie : SCRIPT
- Script concerné : snap-unread-digest
- Action tentée : Construction après seeding (snap photo, snap vidéo, chat, snap+chat, un état à la fois).
- Résultat obtenu : Fonctionne, 3 runs identiques dont 2 à froid ; la conversation reste "Nouveau Snap" après les runs, donc rien n'a été ouvert. Points trouvés :
  - pas de data-testid, classes hachées : l'état non lu et le type sont déduits de la structure et de l'icône, pas du texte français ;
  - un snap photo et un snap vidéo ont le même libellé et la même icône : seule la couleur diffère (rouge / violet), d'où une distinction par la teinte ;
  - les lignes arrivent au fil de l'eau (~9,5 s) : attente que leur nombre soit stable, timeout de 20 s.
- Reproductible : oui
- Note libre : Non vérifié : états "envoyé/ouvert", listes longues (liste virtualisée, d'où `conversations_scanned`). Le cas "mixte" repose sur un seul échantillon. Le type dépend de l'icône : peut casser si Snapchat la redessine.
```

```
### [23/09] Pas d'"amis en commun" sur la page
- Catégorie : SCRIPT
- Script concerné : snap-friend-requests-triage
- Action tentée : Lecture des demandes d'ami en attente, sans jamais accepter, refuser ni bloquer.
- Résultat obtenu : Fonctionne, 3 runs identiques dont 2 à froid. La trace ne contient qu'un seul clic (le bouton d'ouverture du panneau) et les demandes sont toujours en attente après les runs. Points trouvés :
  - la page n'affiche qu'un libellé de relation ("Dans Mes Contacts"), pas un nombre d'amis en commun : `mutual_friends` reste `null` ;
  - le bouton d'ouverture n'a qu'un `title` localisé : choisi par sa position dans la barre du haut, avec une garde qui échoue au lieu de cliquer ailleurs. Elle s'est déclenchée au premier essai (bouton pas encore rendu), sans rien cliquer ; corrigé en attendant le bouton.
- Reproductible : oui
- Note libre : Non observés : panneau vide (le script attend 8 s avant de rendre `[]`), demande sans avatar, listes longues.
```

```
### [24/09] Identifiants illisibles
- Catégorie : INPUT
- Script concerné : snap-story-analytics
- Action tentée : Retour utilisateur : `story_id` et `spotlight_id` (fragments d'URL d'image) ne sont pas parlants.
- Résultat obtenu : Remplacés par un libellé "date de la tuile + rang" (ex. `23/09/2026 #1`). Testé à froid 2x, pour la première fois avec plusieurs éléments (2 Stories, 2 Spotlights) : libellés distincts.
- Reproductible : n/a
- Note libre : Le rang change quand du contenu est ajouté ou retiré : le libellé n'est plus une clé stable. L'ordre est celui de la page (plus ancien d'abord pour les Stories).
```

```
### [24/09] Dashboard bien plus lent, le script tombe puis renvoie des `null`
- Catégorie : SITE
- Script concerné : snap-story-analytics
- Action tentée : Relancer le script promu, signalé comme défaillant. Tous les runs du 24/09 échouaient, alors que tous ceux de la veille avaient réussi.
- Résultat obtenu : Trois causes empilées :
  - profile.snapchat.com est 2 à 3 fois plus lent que la veille : l'événement `load` arrive à 7-14 s (limite de navigation : 10 s) et la première tuile à 21-34 s (9,5 s la veille). Le script échouait dès son premier `goto` ; en plus l'option `domcontentloaded` était ignorée par le `goto` de Reduck, qui attendait `load`.
  - une fois la navigation passée, l'attente de 20 s des tuiles aurait été dépassée, et le `try/catch` autour aurait renvoyé `stories: []` sans erreur, donc un compte avec du contenu vu comme vide. Un écran « vide » s'affiche d'ailleurs brièvement pendant le chargement d'un onglet : l'absence de tuiles ne prouve rien.
  - la classe CSS hachée qui identifiait les lignes de statistiques Spotlight a été renommée en une nuit : toutes les valeurs Spotlight sortaient `null`, sans erreur.
- Reproductible : oui
- Note libre : Corrigé : navigation avec délai relevé, un seul chargement à froid au lieu de trois, lecture des statistiques par la structure (titre + valeur voisine) et non par une classe, et un champ `warnings` qui signale un onglet non chargé au lieu de le renvoyer vide en silence. Testé à froid 2x, valeurs identiques à la modale. Contrepartie : appeler `page` directement supprime le détail des étapes dans la trace Reduck, qui est maintenant presque vide. Le cas « compte sans contenu » n'est toujours pas observé.
```
