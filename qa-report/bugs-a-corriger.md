# Bugs à corriger — scripts Reduck Snapchat

Issus de la campagne de QA du 23-24/09/2026. Chaque bug renvoie aux cas de test des fichiers `snap-*.md` de ce dossier.

Gravité :
- **Critique** : le script renvoie un résultat faux présenté comme un succès.
- **Majeure** : le script ne peut pas donner la bonne réponse dans un cas courant.
- **Mineure** : gêne, lenteur, documentation inexacte.

## Récapitulatif

| ID | Gravité | Composant | Titre | Cas QA |
|---|---|---|---|---|
| B-01 | Critique | snap-friend-requests-triage | Faux succès `[]` quand la session est déconnectée | C-02 |
| B-02 | Critique | snap-friend-requests-triage | Faux succès `[]` quand le panneau est fermé avant la lecture | F-04 |
| B-03 | Critique | snap-story-analytics | Stats à `null` avec une interface dans une autre langue | C-06 |
| B-04 | Majeure | snap-story-analytics | Un compte vide est signalé comme « chargement incomplet » | S-01, SP-01 |
| B-05 | Mineure | snap-story-analytics | Session déconnectée : échec au bout de plus de 2 minutes, message trompeur | C-02 |
| B-06 | Mineure | snap-story-analytics | Ordre des Stories contraire à la documentation | S-05 |
| B-07 | Mineure | snap-story-analytics | Fermeture de la modale Spotlight par `Escape` non prouvée | SP-03 |
| B-08 | Mineure | snap-unread-digest | Pas de type dédié pour les notes vocales | U-06 |
| P-01 | Majeure | Reduck | Page HTML Cloudflare 502 brute quand l'extension est désactivée | C-03 |
| P-02 | Majeure | Reduck | Désactiver puis réactiver l'extension supprime l'appairage sans prévenir | C-03 |
| P-03 | Mineure | Reduck | Journal brut du sandbox quand l'onglet est fermé au démarrage | C-05 |
| P-04 | Mineure | Reduck | `builtins.goto` ignore ses options sans erreur | C-01 |
| P-05 | Mineure | Reduck | Message « Please make sure you are logged into Snapchat » affiché une fois sur deux | C-02 |
