# Prompts à donner à l'agent (Claude Code + Reduck MCP) pour les 3 scripts

> Rappels avant de lancer quoi que ce soit :
> - Compte Snapchat **dédié**, jamais le perso.
> - Demander explicitement à l'agent de rester lent / d'espacer les actions, et de
>   privilégier peu d'itérations de test.
> - Noter dans `logs.md` toute friction dès le setup (compte reduck.ai, extension,
>   `claude mcp add reduck --transport http --scope user https://mcp.reduck.ai`).
> - Un script se construit en plusieurs passes normalement (brouillon → testé →
>   promu). Ne pas s'attendre à ce que ça marche du premier coup — c'est justement ce
>   qu'il faut documenter dans les logs si ça coince.

---

## 1. `snap-story-analytics`

**Prompt à donner à l'agent :**

> Utilise Reduck MCP pour créer un script nommé `snap-story-analytics` sur mon compte
> Snapchat (déjà connecté dans mon Chrome via l'extension Reduck). Le script doit :
> - Aller sur **`https://profile.snapchat.com/`** (le profil public / dashboard
>   créateur) : c'est là que se trouvent les stats détaillées de mes Stories/
>   Spotlight actifs — `web.snapchat.com` n'expose que l'heure de publication, pas
>   les vues/captures/interactions (voir les logs du 23/09).
> - Le dashboard a deux onglets à couvrir séparément, avec des stats différentes par
>   type de contenu — ne pas les fusionner dans un seul tableau :
>   - **Stories publiques** : pour chaque Snap actif, extraire la date/heure de
>     publication, le nombre de spectateurs·rices, le nombre de vues, le nombre de
>     réponses, le nombre de captures d'écran (screenshots), le nombre
>     d'interactions, et le statut de modération.
>   - **Contenus Spotlight** : pour chaque Spotlight actif, extraire le nombre de
>     vues, le nombre de spectateurs·rices, le nombre de likes, et le nombre de
>     partages (pas de réponses/captures/interactions/modération sur Spotlight — ces
>     champs n'existent pas pour ce type de contenu).
> - Retourner un objet JSON avec deux tableaux séparés :
>   `{ stories: [{ story_id, posted_at, viewers, views, replies, screenshots,
>   interactions, moderation_status }], spotlights: [{ spotlight_id, views, viewers,
>   likes, shares }] }`.
> - Ne fais aucune action d'écriture (pas de like, pas de réponse, pas de partage) —
>   uniquement de la lecture de données déjà visibles pour moi en tant que propriétaire
>   du compte.
> - Teste le script sur mon compte actuel avant de le promouvoir, et montre-moi le
>   JSON obtenu pour que je vérifie qu'il correspond à ce que je vois à l'écran.

**Points à vérifier après génération (avant promotion) :**
- Les chiffres du JSON correspondent-ils exactement à ce qui est affiché dans l'appli/
  le web pour chaque Story et chaque Spotlight ?
- Que se passe-t-il si je n'ai aucune Story active, et séparément aucun Spotlight
  actif (tableaux vides propres, ou erreur) ?
- Le profil public/créateur est-il bien activé sur le compte dédié ? Si
  `profile.snapchat.com` n'affiche rien pour ce compte, c'est probablement la cause
  (à vérifier avant de conclure à un bug du script).
- La session de `web.snapchat.com` suffit-elle à être authentifié sur
  `profile.snapchat.com`, ou faut-il une étape de login séparée ?
- En repli, si `profile.snapchat.com` est indisponible pour ce compte,
  `web.snapchat.com` reste utilisable pour au moins récupérer la date de post.
- Le chargement de la liste de tuiles (Stories comme Spotlight) est asynchrone et
  peut prendre près de 10 secondes — vérifier que le script n'utilise pas un timeout
  par défaut trop court, sous peine de conclure à tort à "aucun contenu actif" (voir
  les logs du 23/09).

---

## 2. `snap-unread-digest`

**Prompt à donner à l'agent :**

> Utilise Reduck MCP pour créer un script nommé `snap-unread-digest` sur mon compte
> Snapchat. Le script doit :
> - Aller sur `https://web.snapchat.com/` (liste des conversations et snaps reçus).
>   Aucun onglet Snapchat Web ne doit être déjà ouvert : le site n'autorise qu'un
>   onglet à la fois (voir les logs du 23/09).
> - Repérer les conversations marquées comme non lues (snap non ouvert, message non
>   lu). La page n'a ni `data-testid` ni classes stables : lire l'état non lu et le
>   type de contenu à partir de la structure (forme et couleur de l'icône), pas du
>   texte français des libellés.
> - Pour chacune, extraire : le pseudo de l'expéditeur, le type de contenu (snap
>   photo, snap vidéo, message texte, ou mixte quand la ligne résume plusieurs types
>   à la fois), le libellé de statut tel qu'affiché, et l'horodatage de réception
>   (date ISO lue sur la ligne).
> - Retourner un objet JSON, avec les non lus triés du plus ancien au plus récent :
>   `{ conversations_scanned, unread: [{ conversation_id, sender, content_type,
>   status_text, received_at }] }`, où `content_type` vaut `snap_photo`, `snap_video`,
>   `snap`, `chat`, `mixed` ou `unknown`.
> - N'ouvre, ne lis et ne marque aucun snap/message comme lu — le but est justement de
>   lister sans consommer, pour ne pas fausser mon propre usage ni déclencher de
>   confirmations de lecture côté expéditeur.
> - Teste sur mon compte actuel, montre-moi le JSON, et vérifie avec moi qu'il
>   correspond à ce qui est marqué non lu à l'écran avant de promouvoir.

**Points à vérifier après génération :**
- Le script confirme-t-il bien qu'il n'ouvre aucun snap (risque de "griller" l'aperçu
  et donc de fausser l'état non-lu, ou de notifier l'expéditeur) ?
- Comportement si la liste est vide ? (cas vu : 3 conversations toutes lues → tableau
  `unread` vide.)
- Le tri chronologique est-il correct ? (non vérifié avec plusieurs non lus : un seul
  non lu à la fois sur le compte de test.)
- Faut-il « seeder » le compte dédié depuis un second compte avant de tester ? Oui :
  sans élément non lu, l'encodage de l'état non lu reste invisible. Envoyer un snap
  photo, un snap vidéo et un message texte, un état à la fois, sans les ouvrir côté
  compte dédié (les ouvrir à la main entre deux pour repasser en « Reçu »).
- Photo et vidéo ont le même libellé et la même icône : seule la couleur les
  distingue. Vérifier que le script les sépare bien.
- La liste de conversations est virtualisée : sur un compte avec beaucoup de
  conversations, seules les lignes rendues sont vues (`conversations_scanned` le rend
  observable). Non testé avec plus de 3 conversations.
- Le chargement de la liste prend près de 10 secondes : vérifier que le script attend
  la stabilisation du nombre de lignes et n'utilise pas un timeout par défaut trop
  court.
- Charger `web.snapchat.com` peut allumer la webcam de la machine (comportement du
  site) : à savoir avant de lancer le script sur un poste partagé.

---

## 3. `snap-friend-requests-triage`

**Prompt à donner à l'agent :**

> Utilise Reduck MCP pour créer un script nommé `snap-friend-requests-triage` sur mon
> compte Snapchat. Le script doit :
> - Aller sur la section des demandes d'ami en attente ("Ajouté(e) par" / "Added Me").
> - Pour chaque demande en attente, extraire : le pseudo, le nom affiché si différent,
>   le nombre d'amis en commun si affiché, et la présence ou non d'une photo de profil/
>   bitmoji personnalisé (signal simple de compte probablement réel vs faux compte).
> - Retourner un tableau JSON : `{ username, display_name, mutual_friends, has_avatar
>   }`.
> - Important : le script ne doit accepter, refuser ou bloquer aucune demande
>   automatiquement. Il liste uniquement, la décision reste manuelle.
> - Teste sur mon compte actuel (même si la liste est vide au début), montre-moi le
>   JSON, avant de promouvoir.

**Points à vérifier après génération :**
- Confirmer noir sur blanc qu'aucune action d'accept/refuse/block n'a été exécutée
  pendant les tests.
- Comportement si aucune demande en attente (cas probable sur un compte tout neuf) —
  prévoir de générer 1-2 fausses demandes avec un second compte test pour valider le
  script avant promotion.

---

## Ordre de construction recommandé

1. `snap-story-analytics` (le plus différenciant, à valider en premier)
2. `snap-unread-digest`
3. `snap-friend-requests-triage` (nécessite d'avoir préparé au moins une demande
   d'ami de test — cf. étape de "seeding" du compte dédié)

Après chaque script promu : le rejouer une deuxième fois à froid (nouvelle session)
pour confirmer la reproductibilité avant de passer au suivant.
