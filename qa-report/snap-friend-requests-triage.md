# QA — snap-friend-requests-triage

Script : `@mde/web.snapchat.com/snap-friend-requests-triage` (lecture seule, sans argument)

Criticité : même échelle que `bugs-a-corriger.md` (Critique / Majeure / Mineure). « Aucune » si le cas passe, « Non évaluée » si le test n'est pas concluant.

## Cas communs

### C-01 — Compte connecté, cas nominal
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-friend-requests-triage
- Action tentée : Lancer le script sans demande d'ami en attente.
- Résultat attendu : `requests: []`, rien n'est accepté ni refusé.
- Résultat obtenu : OK sur 2 lancements. J'ai vérifié que la session était bien active, parce que ce script renvoie aussi `[]` quand on est déconnecté (voir C-02).
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Aucune demande en attente
    2. Lancer le script

### C-02 — Session Snapchat déconnectée
- Catégorie : SCRIPT
- Criticité : Critique
- Script concerné : snap-friend-requests-triage
- Action tentée : Me déconnecter de Snapchat, puis lancer le script.
- Résultat attendu : Une erreur claire, pas de liste vide.
- Résultat obtenu : **Échec : faux succès**, 2 fois sur 2. Le script renvoie `requests: []` comme si tout allait bien. Sur la page de connexion, il clique sur le premier bouton qu'il trouve en haut (le menu des applis Snapchat), n'y voit aucune demande et conclut qu'il n'y en a pas. Impossible de faire la diffèrence avec « 0 demande ».
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Se déconnecter de Snapchat
    2. Lancer le script : succès avec `requests: []`

### C-03 — Extension Reduck désactivée
- Catégorie : PRODUIT
- Criticité : Majeure
- Script concerné : snap-friend-requests-triage
- Action tentée : Désactiver l'extension, puis lancer le script.
- Résultat attendu : Une erreur claire, tout de suite.
- Résultat obtenu : OK, message clair : « No device available ». Bizarrement, les deux autres scripts lancés quelques secondes avant avaient eu une page HTML `502` à la place. Autre souci : après réactivation, l'appareil avait disparu et il a fallu refaire l'appairage.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Désactiver l'extension
    2. Lancer le script

### C-04 — Mode sombre
- Catégorie : SCRIPT
- Criticité : Non évaluée
- Script concerné : snap-friend-requests-triage
- Action tentée : Passer Snapchat en sombre, puis lancer le script.
- Résultat attendu : Même résultat qu'en thème clair.
- Résultat obtenu : Non concluant, le thème sombre ne s'est pas appliqué dans l'onglet de Reduck. À refaire, idéalement avec une demande en attente.
- Reproductible : non le thème sombre ne s'est pas appliqué

### C-05 — Fermeture de l'onglet dès le lancement
- Catégorie : PRODUIT
- Criticité : Aucune
- Script concerné : snap-friend-requests-triage
- Action tentée : Lancer le script et fermer tout de suite l'onglet ouvert par Reduck.
- Résultat attendu : Le run échoue vite, sans rester bloqué.
- Résultat obtenu : OK, échec immédiat avec un message lisible (`Target page, context or browser has been closed`).
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Lancer le script
    2. Fermer l'onglet dès qu'il apparaît

### C-06 — Interface Snapchat en anglais
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-friend-requests-triage
- Action tentée : Passer l'interface en anglais, puis lancer le script avec une demande en attente.
- Résultat attendu : Les libellés passent en anglais, le reste ne bouge pas.
- Résultat obtenu : OK. Le script repère les lignes par leur structure et le bouton par sa position, pas par le texte.
- Reproductible : à confirmer par un test live
  - Étapes de repro (si oui) :
    1. Passer Chrome en anglais
    2. Lancer le script

## Cas propres au script

### F-01 — Aucune demande en attente
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-friend-requests-triage
- Action tentée : Lancer le script sans demande en attente.
- Résultat attendu : `requests: []` au bout de 8 s environ.
- Résultat obtenu : OK sur 2 lancements. Mais attention, on aurait exactement le même résultat si le panneau ne s'était pas ouvert (voir F-04).
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Traiter toutes les demandes à la main
    2. Lancer le script

### F-02 — Demandes en attente
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-friend-requests-triage
- Action tentée : Lancer le script avec 2 demandes en attente.
- Résultat attendu : Une entrée par demande.
- Résultat obtenu : OK, les 2 demandes remontent avec le pseudo, le nom, la relation (« Dans Mes Contacts ») et l'avatar. Même résultat sur 3 lancements, et les demandes sont toujours en attente après.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Recevoir 2 demandes d'ami
    2. Lancer le script

### F-03 — Beaucoup de demandes (30 ou plus)
- Catégorie : SCRIPT
- Criticité : Non évaluée
- Script concerné : snap-friend-requests-triage
- Action tentée : Lancer le script avec une longue liste de demandes.
- Résultat attendu : Toutes les demandes remontent.
- Résultat obtenu : Non testé. Il faudrait 30 comptes Snapchat différents, pas faisable ici.
- Reproductible : non (jeu de données impossible à constituer)

### F-04 — Fermeture du panneau des demandes
- Catégorie : SCRIPT
- Criticité : Critique
- Script concerné : snap-friend-requests-triage
- Action tentée : Fermer à la main le panneau des demandes juste après que le script l'a ouvert, avec 1 demande en attente.
- Résultat attendu : Une erreur, ou la bonne demande. Pas de liste vide.
- Résultat obtenu : **Échec : faux succès.** Le script a bien cliqué sur le bon bouton, mais une fois le panneau fermé il renvoie `requests: []` comme si de rien n'était. La demande était pourtant toujours en attente. Même problème qu'en C-02 : le script ne vérifie pas que le panneau est encore ouvert avant de lire.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Avoir au moins 1 demande en attente
    2. Lancer le script
    3. Fermer le panneau dès qu'il s'ouvre
