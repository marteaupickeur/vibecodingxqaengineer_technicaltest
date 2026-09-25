# QA — snap-story-analytics

Script : `@mde/profile.snapchat.com/snap-story-analytics` (lecture seule, sans argument)

Criticité : même échelle que `bugs-a-corriger.md` (Critique / Majeure / Mineure). « Aucune » si le cas passe, « Non évaluée » si le test n'est pas concluant.

## Cas communs

### C-01 — Compte connecté, cas nominal
- Catégorie : SITE / SCRIPT
- Criticité : Mineure
- Script concerné : snap-story-analytics
- Action tentée : Lancer le script avec 1 Story et 2 Spotlights en ligne.
- Résultat attendu : Le script réussit et renvoie les stats.
- Résultat obtenu : OK, mais seulement après correction (v5). Les 2 snaps et les 2 Spotlights remontent avec les bons chiffres.
  - Avant la v5, ça plantait à chaque fois au bout de 10 s : profile.snapchat.com était très lent ce jour-là, et `builtins.goto` ignore les options de délai qu'on lui passe. La v5 utilise `page.goto` avec un délai plus long et renvoie des `warnings` au lieu d'un vide silencieux.
  - Au passage, Snapchat avait changé une classe CSS pendant la nuit, ce qui aurait mis toutes les stats Spotlight à `null`. Corrigé aussi dans la v5.
  - Bizarrerie côté Snapchat : un Spotlight affiche 8 spectateurs pour 7 vues. J'ai vérifié à l'écran, c'est bien ce que le site affiche, le script n'y est pour rien.
- Reproductible : oui (avant la v5)
  - Étapes de repro (si oui) :
    1. Lancer la version d'avant la v5 quand le site est lent
    2. Le run échoue au bout de 10 s

### C-02 — Session Snapchat déconnectée
- Catégorie : SCRIPT
- Criticité : Mineure
- Script concerné : snap-story-analytics
- Action tentée : Me déconnecter de Snapchat, puis lancer le script.
- Résultat attendu : Une erreur claire, pas de liste vide.
- Résultat obtenu : Pas de faux vide, c'est l'essentiel. Mais le script met plus de 2 minutes à échouer, et l'erreur parle de l'onglet Spotlights au lieu de dire qu'on est déconnecté.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Se déconnecter de Snapchat
    2. Lancer le script : échec au bout de plus de 2 min

### C-03 — Extension Reduck désactivée
- Catégorie : PRODUIT
- Criticité : Majeure
- Script concerné : snap-story-analytics
- Action tentée : Désactiver l'extension dans `chrome://extensions`, puis lancer le script.
- Résultat attendu : Une erreur claire, tout de suite.
- Résultat obtenu : Ça échoue bien tout de suite, mais le message est illisible : `HTTP 502` suivi de toute une page HTML Cloudflare. Rien ne dit que l'extension est désactivée. Et en la réactivant, l'appareil avait disparu : il a fallu refaire l'appairage.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Désactiver l'extension
    2. Lancer le script

### C-04 — Mode sombre
- Catégorie : SCRIPT
- Criticité : Non évaluée
- Script concerné : snap-story-analytics
- Action tentée : Passer Snapchat en thème sombre, puis lancer le script.
- Résultat attendu : Même résultat qu'en thème clair.
- Résultat obtenu : Non concluant. Le script a marché, mais le thème sombre ne s'est pas appliqué dans l'onglet ouvert par Reduck. À refaire.
- Reproductible : non, le thème sombre ne s'est pas appliqué

### C-05 — Fermeture de l'onglet dès le lancement
- Catégorie : PRODUIT
- Criticité : Aucune
- Script concerné : snap-story-analytics
- Action tentée : Lancer le script et fermer tout de suite l'onglet ouvert par Reduck.
- Résultat attendu : Le run échoue vite, sans rester bloqué.
- Résultat obtenu : OK, échec immédiat. Le message (`net::ERR_ABORTED; maybe frame was detached?`) ne dit pas clairement que l'onglet a été fermé, mais bon, on comprend.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Lancer le script
    2. Fermer l'onglet « Reduck is controlling this tab »

### C-06 — Interface Snapchat en anglais
- Catégorie : SCRIPT
- Criticité : Critique
- Script concerné : snap-story-analytics
- Action tentée : Passer l'interface en anglais, puis lancer le script.
- Résultat attendu : Même résultat qu'en français.
- Résultat obtenu : Échec (faux succès). Le script cherche les stats par leurs libellés en français (`Vues`, `Spectateurs·rices`, `Likes`…). En anglais il ne les trouve pas et renvoie tout à `null`, avec un statut de succès.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Passer Chrome en anglais
    2. Lancer le script

## Stories

### S-01 — Aucune Story
- Catégorie : SCRIPT
- Criticité : Majeure
- Script concerné : snap-story-analytics
- Action tentée : Supprimer toutes les Stories, puis lancer le script.
- Résultat attendu : `stories: []`, sans erreur.
- Résultat obtenu : `stories: []`, mais avec un avertissement « le panneau n'a pas fini de charger ». Le script attend un bouton qui n'apparaît que s'il y a au moins une Story, donc sur un compte vide il attend 60 s pour rien. Résultat : impossible de savoir si le compte est vide ou si la page était juste lente.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Supprimer toutes les Stories
    2. Lancer le script

### S-02 — 1 Story, 1 snap
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-story-analytics
- Action tentée : Lancer le script avec une seule Story.
- Résultat attendu : 1 entrée complète.
- Résultat obtenu : OK, 1 entrée avec la date, les vues, les spectateurs et le reste.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Publier une Story
    2. Lancer le script

### S-03 — 1 Story avec plusieurs snaps
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-story-analytics
- Action tentée : Lancer le script avec 2 snaps dans la même Story.
- Résultat attendu : Une entrée par snap.
- Résultat obtenu : OK, les 2 snaps remontent (`#1` et `#2`).
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Publier 2 snaps en Story
    2. Lancer le script

### S-04 — Plusieurs Stories
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-story-analytics
- Action tentée : Lancer le script avec 2 Stories.
- Résultat attendu : Les 2 sont là, numérotées dans l'ordre.
- Résultat obtenu : OK, même résultat sur 3 lancements d'affilée.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Publier 2 Stories
    2. Lancer le script

### S-05 — Story publiée pendant que le script tourne
- Catégorie : SCRIPT
- Criticité : Mineure
- Script concerné : snap-story-analytics
- Action tentée : Publier une Story depuis le téléphone pendant le run.
- Résultat attendu : Pas de plantage, pas de doublon.
- Résultat obtenu : OK. La nouvelle Story n'apparait pas dans ce run (le script lit la liste une seule fois au début), mais elle est bien là au run suivant. Petit écart : elle arrive en premier, alors que la doc du script dit que les plus anciennes viennent d'abord.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Lancer le script
    2. Publier une Story pendant le chargement
    3. Relancer : la nouvelle Story est en `#1`

## Spotlight

### SP-01 — Aucun Spotlight
- Catégorie : SCRIPT
- Criticité : Majeure
- Script concerné : snap-story-analytics
- Action tentée : Supprimer tous les Spotlights, puis lancer le script.
- Résultat attendu : `spotlights: []`, sans erreur.
- Résultat obtenu : Même souci que S-01 : `spotlights: []` avec un avertissement « pas fini de charger », alors que la page affichait clairement « Créer un contenu Spotlight ». Le compte vide n'est pas reconnu.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Supprimer tous les Spotlights
    2. Lancer le script

### SP-02 — 1 puis plusieurs Spotlights
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-story-analytics
- Action tentée : Lancer le script avec 1 Spotlight, puis avec 2.
- Résultat attendu : Une entrée par Spotlight.
- Résultat obtenu : OK dans les 2 cas, les likes et partages sont à `0` et pas à `null`.
- Reproductible : oui
  - Étapes de repro (si oui) :
    1. Publier 1 Spotlight, lancer le script
    2. En publier un 2ᵉ, relancer

### SP-03 — Fermeture de la modale Spotlight
- Catégorie : SCRIPT
- Criticité : Mineure
- Script concerné : snap-story-analytics
- Action tentée : Fermer à la main la modale Spotlight pendant que le script la lit.
- Résultat attendu : Le script lit les stats ou échoue proprement.
- Résultat obtenu : Non concluant. Le script lit chaque modale en moins de 0,3 s, impossible de la fermer à temps à la main. Par contre j'ai remarqué que la dernière modale restait ouverte à la fin, donc pas sûr que la fermeture par `Escape` marche vraiment.
- Reproductible : non (trop rapide pour un humain)

### SP-04 — Spotlight dont la vidéo n'est pas encore chargée
- Catégorie : SCRIPT
- Criticité : Aucune
- Script concerné : snap-story-analytics
- Action tentée : Lire un Spotlight dont la vidéo charge encore.
- Résultat attendu : Les stats sont lues quand même.
- Résultat obtenu : OK le script attend que la vidéo soit là. Je n'ai vu que des Spotlights vidéo, donc un Spotlight sans vidéo reste à voir.
- Reproductible : oui (avant la correction)
  - Étapes de repro (si oui) :
    1. Ancienne version, session froide
    2. Lancer le script
