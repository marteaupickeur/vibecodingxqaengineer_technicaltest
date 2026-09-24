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
> - Aller sur la page qui liste mes propres Stories/Spotlight actifs (postés récemment
>   et encore visibles).
> - Pour chaque Story/Spotlight actif, extraire : le titre/légende s'il y en a une, la
>   date/heure de publication, le nombre de vues, le nombre de captures d'écran
>   (screenshots), le nombre de réponses (replies) reçues.
> - Retourner un tableau JSON, un objet par Story, avec ces champs :
>   `{ story_id, caption, posted_at, views, screenshots, replies }`.
> - Ne fais aucune action d'écriture (pas de like, pas de réponse, pas de partage) —
>   uniquement de la lecture de données déjà visibles pour moi en tant que propriétaire
>   du compte.
> - Teste le script sur mon compte actuel avant de le promouvoir, et montre-moi le
>   JSON obtenu pour que je vérifie qu'il correspond à ce que je vois à l'écran.

**Points à vérifier après génération (avant promotion) :**
- Les chiffres du JSON correspondent-ils exactement à ce qui est affiché dans l'appli/
  le web pour chaque Story ?
- Que se passe-t-il si je n'ai aucune Story active (tableau vide propre, ou erreur) ?
- Le script gère-t-il le cas Story vs Spotlight séparément si le champ diffère ?

---

## 2. `snap-unread-digest`

**Prompt à donner à l'agent :**

> Utilise Reduck MCP pour créer un script nommé `snap-unread-digest` sur mon compte
> Snapchat. Le script doit :
> - Aller sur mon écran de discussions/chat (liste des conversations et snaps reçus).
> - Repérer les conversations marquées comme non lues (snap non ouvert, message non
>   lu).
> - Pour chacune, extraire : le pseudo de l'expéditeur, le type de contenu (snap
>   photo/vidéo, message texte, snap vocal si visible), et l'horodatage de réception
>   si affiché.
> - Retourner un tableau JSON trié du plus ancien au plus récent :
>   `{ sender, content_type, received_at }`.
> - N'ouvre, ne lis et ne marque aucun snap/message comme lu — le but est justement de
>   lister sans consommer, pour ne pas fausser mon propre usage ni déclencher de
>   confirmations de lecture côté expéditeur.
> - Teste sur mon compte actuel, montre-moi le JSON, et vérifie avec moi qu'il
>   correspond à ce qui est marqué non lu à l'écran avant de promouvoir.

**Points à vérifier après génération :**
- Le script confirme-t-il bien qu'il n'ouvre aucun snap (risque de "griller" l'aperçu
  et donc de fausser l'état non-lu, ou de notifier l'expéditeur) ?
- Comportement si la liste est vide ?
- Le tri chronologique est-il correct ?

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
