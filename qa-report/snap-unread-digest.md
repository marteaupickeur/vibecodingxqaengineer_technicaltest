# QA — snap-unread-digest

Script : `@mde/web.snapchat.com/snap-unread-digest` (lecture seule, sans argument)

Criticité : même échelle que `bugs-a-corriger.md` (Critique / Majeure / Mineure). « Aucune » si le cas passe, « Non évaluée » si le test n'est pas concluant.

## Cas communs

### C-01 — Compte connecté, cas nominal
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-unread-digest
- Action tentée : Lancer le script avec 1 message texte non lu.
- Résultat attendu : Le message remonte et reste non lu.
- Résultat obtenu : OK sur 2 lancements, le message remonte en `chat`. J'ai vérifié sur le téléphone : il est toujours non lu après le run.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Recevoir un message sans l'ouvrir
    2. Lancer le script

### C-02 — Session Snapchat déconnectée
- Catégorie : SCRIPT / PRODUIT
- Criticité : Mineure
- Script concerné : snap-unread-digest
- Action tentée : Me déconnecter de Snapchat, puis lancer le script.
- Résultat attendu : Une erreur claire, pas de liste vide.
- Résultat obtenu : OK, le script échoue au bout de 20 s sans renvoyer de liste vide. La 1ʳᵉ fois Reduck a affiché un message clair (« Please make sure you are logged into Snapchat… »), la 2ᵉ fois non, juste l'erreur technique. Pas tres constant.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Se déconnecter de Snapchat
    2. Lancer le script

### C-03 — Extension Reduck désactivée
- Catégorie : PRODUIT
- Criticité : Majeure
- Script concerné : snap-unread-digest
- Action tentée : Désactiver l'extension, puis lancer le script.
- Résultat attendu : Une erreur claire, tout de suite.
- Résultat obtenu : Échec immédiat, mais avec une page HTML Cloudflare `502` en guise de message. Rien ne dit que l'extension est désactivée.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Désactiver l'extension
    2. Lancer le script

### C-04 — Mode sombre
- Catégorie : SCRIPT
- Criticité : Non évaluée
- Script concerné : snap-unread-digest
- Action tentée : Passer Snapchat en sombre, avec un snap photo et un snap vidéo non lus.
- Résultat attendu : Photo et vidéo bien reconnues (le script les distingue par la couleur).
- Résultat obtenu : Non concluant. Le thème sombre ne s'est pas appliqué dans l'onglet de Reduck, et je n'avais pas de snap vidéo non lu. À refaire, avec la photo et la vidéo dans deux conversations différentes.
- Reproductible : non le thème sombre ne s'est pas appliqué

### C-05 — Fermeture de l'onglet dès le lancement
- Catégorie : PRODUIT
- Criticité : Mineure
- Script concerné : snap-unread-digest
- Action tentée : Lancer le script et fermer tout de suite l'onglet ouvert par Reduck.
- Résultat attendu : Le run échoue vite, sans rester bloqué.
- Résultat obtenu : OK sur le fond, mais le message d'erreur est un vrai pavé technique : logs du sandbox, codes couleur, URL websocket, et même un conseil Deno qui n'a rien a voir.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Lancer le script
    2. Fermer l'onglet avant que Snapchat commence à charger

### C-06 — Interface Snapchat en anglais
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-unread-digest
- Action tentée : Passer l'interface en anglais, puis lancer le script.
- Résultat attendu : Seul `status_text` change de langue.
- Résultat obtenu : OK. Le type de message est déduit de l'icône et de la couleur.
- Reproductible : ou
  - Étapes de repro (si oui) :
    1. Passer Chrome en anglais
    2. Lancer le script

## Cas propres au script

### U-01 — Aucun message non lu
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-unread-digest
- Action tentée : Tout lire sur le téléphone, puis lancer le script.
- Résultat attendu : `unread: []`, sans erreur.
- Résultat obtenu : OK, `unread: []` avec 6 conversations lues. Le script fait bien la différence entre « rien de non lu » (succès) et « rien de chargé » (erreur, voir C-02).
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Tout lire sur le téléphone
    2. Lancer le script

### U-02 — 1 snap photo non lu
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-unread-digest
- Action tentée : Recevoir un snap photo sans l'ouvrir, puis lancer le script.
- Résultat attendu : `snap_photo`.
- Résultat obtenu : OK sur 3 lancements. La première version renvoyait juste `snap`, parce que photo et vidéo ont la même icône. Maintenant on les distingue par la couleur.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Recevoir un snap photo
    2. Lancer le script sans l'ouvrir

### U-03 — 1 snap vidéo non lu
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-unread-digest
- Action tentée : Recevoir un snap vidéo sans l'ouvrir, puis lancer le script.
- Résultat attendu : `snap_video`.
- Résultat obtenu : OK sur plusieurs lancements, et le snap reste non lu après.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Recevoir un snap vidéo
    2. Lancer le script sans l'ouvrir

### U-04 — 1 chat non lu
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-unread-digest
- Action tentée : Recevoir un message texte, puis lancer le script.
- Résultat attendu : `chat`.
- Résultat obtenu : OK. Le nom avec emoji (« Mme Bonheur ♥️ ») passe sans probleme, et le tri du plus ancien au plus récent est respecté.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Recevoir un message texte
    2. Lancer le script

### U-05 — Snap + chat dans la même conversation
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-unread-digest
- Action tentée : Recevoir un snap puis un message dans la même conversation.
- Résultat attendu : `mixed`.
- Résultat obtenu : OK, `mixed`. Testé une seule fois.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Recevoir un snap puis un message du même contact
    2. Lancer le script

### U-06 — Note vocale, sticker, lien, pièce jointe
- Catégorie : SCRIPT
- Criticité : Mineure
- Script concerné : snap-unread-digest
- Action tentée : Recevoir ces contenus, puis lancer le script.
- Résultat attendu : Pas de plantage, un type cohérent.
- Résultat obtenu : Partiel. La note vocale remonte en `chat` avec « Nouveau message vocal » : ça marche, mais seul le texte (en français) indique que c'est un vocal. Sticker, lien et pièce jointe pas encore testés.
- Reproductible : oui (note vocale)
  - Étapes de repro (si oui) :
    1. Recevoir une note vocale sans l'écouter
    2. Lancer le script
