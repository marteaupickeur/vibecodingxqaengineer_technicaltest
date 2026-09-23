# Scope write-up — veille d'un compte Snapchat avec Reduck

## Le cas d'usage

Suivre un compte Snapchat de créateur sans avoir à ouvrir l'application plusieurs fois par jour. Trois questions reviennent chaque jour :

1. **Mon contenu marche-t-il ?** Combien de vues, de spectateurs, de captures d'écran, de likes, de partages pour mes Stories et mes Spotlights ?
2. **Qu'est-ce que je n'ai pas encore lu ?** Quels snaps et quels messages attendent, de qui, depuis quand ?
3. **Qui veut m'ajouter ?** Quelles demandes d'ami sont en attente, et lesquelles ont l'air réelles ?

Le périmètre est volontairement **en lecture seule** : les scripts observent le compte, ils ne publient rien, n'ouvrent aucun snap et ne répondent à aucune demande. Les décisions restent humaines.

## Pourquoi ce périmètre est intéressant

- **Ce sont des écrans que quelqu'un consulte à la main, tous les jours.** Les chiffres de performance sont dans un tableau de bord web, la boîte de réception et les demandes d'ami dans l'interface de chat. Aucun de ces écrans n'expose ses données sous une forme réutilisable : c'est une personne qui les lit.
- **Le compte est connecté dans le navigateur de la personne.** C'est le point fort de Reduck ici : les scripts héritent de la session déjà ouverte, sans identifiants à confier ni à stocker.
- **Il couvre deux surfaces qui se comportent très différemment**, ce qui met les scripts à l'épreuve : un tableau de bord avec des `data-testid` (`profile.snapchat.com`) et une application de chat sans aucun identifiant stable, avec des classes hachées et des listes virtualisées (`web.snapchat.com`).
- **Le risque est faible et maîtrisé** : lecture seule, compte dédié, aucune action irréversible.
- **Chaque script répond à une question qu'un humain se pose vraiment**, et leurs sorties se composent : on peut les appeler à la suite pour un point du matin, ou en boucle avec un seuil pour déclencher un rappel.

## Comment les trois scripts couvrent le cas

| Question | Script | Site | Ce qu'il renvoie |
|---|---|---|---|
| Mon contenu marche-t-il ? | `snap-story-analytics` | `profile.snapchat.com` | Deux tableaux séparés. **Stories** : date de création, spectateurs, vues, réponses, captures d'écran, interactions, statut de modération. **Spotlights** : vues, spectateurs, likes, partages. Chaque élément a un libellé lisible, `date + rang` (ex. `23/09/2026 #1`). |
| Qu'est-ce que je n'ai pas lu ? | `snap-unread-digest` | `web.snapchat.com` | Les conversations non lues, du plus ancien au plus récent : expéditeur, type (`snap_photo`, `snap_video`, `chat`, `mixed`), libellé de statut, date ISO de réception. N'ouvre rien, donc ne marque rien comme lu. |
| Qui veut m'ajouter ? | `snap-friend-requests-triage` | `web.snapchat.com` | Les demandes en attente : pseudo, nom affiché, relation (ex. « Dans Mes Contacts »), présence d'un avatar, intitulé de la section. N'accepte, ne refuse et ne bloque rien. |

**Un point du matin, de bout en bout :** le premier script dit si les derniers contenus ont fonctionné, le deuxième liste ce qui attend une réponse, le troisième liste les personnes à examiner. La personne publie, répond et décide ; les scripts lui évitent d'aller chercher l'information.

## Ce qui a été vérifié

- Les trois scripts sont promus, déclarés en lecture seule (`sideEffects: none`) et privés.
- Chacun a été rejoué **à froid au moins deux fois** avec un résultat identique.
- Ils ont été validés sur un compte dédié, avec des données réelles : 2 Stories et 2 Spotlights, un snap photo, un snap vidéo et un message texte non lus, 2 demandes d'ami en attente.
- Pour le script des demandes d'ami, la trace de chaque run ne contient qu'un seul clic (le bouton qui ouvre le panneau), et les demandes sont toujours en attente après les tests.

## Limites à connaître

- **Cas non observés** : aucune Story ou Spotlight active, aucun élément non lu dans un état « envoyé/ouvert », un panneau de demandes vide, une demande sans avatar, un très grand nombre de conversations (liste virtualisée).
- **Le rang d'un élément n'est pas une clé stable** : il change quand du contenu est ajouté ou retiré.
- **Le type d'un snap non lu est déduit de la forme et de la couleur de l'icône**, pas du texte. Ça peut casser si Snapchat redessine l'interface.
- **Le compte n'est pas à l'abri d'une restriction** : automatiser Snapchat va à l'encontre de l'usage normal du service. D'où le compte dédié, un rythme lent et une utilisation normale du compte entre deux lancements.
- **Snapchat Web n'autorise qu'un onglet à la fois** et peut allumer la webcam au chargement.

## Prolongement possible

Un rappel « pense à poster » : le script des Stories donne l'heure de publication, il suffit qu'un déclencheur externe compare cette date à l'heure actuelle et prévienne au bout d'un certain temps. Ce n'est pas construit : ce serait de la composition autour des scripts, pas un nouveau script.
