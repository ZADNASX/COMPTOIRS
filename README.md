# Comptoirs

Application de gestion commerciale — devis, pro forma, factures, avoirs, bons de livraison et reçus — pour les maisons de ZADNA SOLUTIONS ET SERVICES.

Une seule application sert plusieurs comptoirs. On se connecte, on choisit sa maison, et chacune garde ses mentions légales, sa numérotation et ses documents. Rien ne circule de l'une à l'autre.

## Déploiement

Site statique, un seul fichier. Aucune étape de construction.

| Réglage Cloudflare Pages | Valeur |
|---|---|
| Framework preset | None |
| Build command | *(vide)* |
| Build output directory | `/` |

`_headers` empêche le navigateur de garder une version périmée de `index.html` en cache.

## Données

Supabase, projet `pfofqxhrtzydarlipcqy`.

- `comptoirs` — une ligne par maison
- `reglages`, `clients`, `articles`, `documents` — cloisonnés par comptoir, corps en `jsonb`

La clé publiable présente dans `index.html` est publique par construction : c'est la politique RLS qui protège, et elle s'appuie sur la table `autorises`. **Un compte créé mais absent de cette liste ne voit rien.**

Chaque appareil garde un miroir local des données du comptoir ouvert : on continue à facturer hors ligne, et tout repart à la reconnexion.

## Après le premier déploiement

Dans Supabase → Authentication → URL Configuration, renseigner l'URL du site comme **Site URL** et l'ajouter aux **Redirect URLs**. Sans ça, le lien de confirmation reçu à l'inscription pointe vers `localhost` et la première connexion échoue.

## Rédaction assistée des courriers

L'éditeur de courrier porte une carte « Rédiger avec l'IA » : on décrit ce que la lettre doit dire, ou l'on joint la photo ou le PDF d'un courrier reçu, et l'IA propose l'objet, la formule d'appel, le corps, la formule de politesse et les pièces jointes. Pour une réponse, elle remplit aussi le destinataire, sa qualité, son adresse et sa référence, et affiche ce qu'elle a compris du courrier reçu. Rien n'est enregistré sans « Enregistrer » ; « Revenir au texte précédent » annule une proposition.

La clé de l'API Claude ne peut pas vivre dans une page publique. L'appel passe par la fonction Supabase **`rediger-courrier`** (source : `ZADNA S.X/app/fonctions/rediger-courrier/index.ts`), qui :

- exige un compte connecté **et** présent dans `autorises` pour le comptoir demandé — un compte non listé ne déclenche aucun appel payant ;
- lit la clé dans le secret **`ANTHROPIC_API_KEY`** (Supabase → Edge Functions → Secrets) ;
- interroge `claude-opus-5` et renvoie le texte au fil de l'eau, une ligne JSON par événement.

Les pièces jointes ne sont conservées nulle part : les photos sont réduites à 2 000 px dans le navigateur, envoyées, puis oubliées.

## Envoi des pièces par WhatsApp

Le bouton « Envoyer par WhatsApp » de l'éditeur enregistre la pièce si besoin, puis en fabrique le PDF dans le navigateur (html2canvas + jsPDF, chargés au premier envoi). Le PDF reprend la feuille imprimée ; les pages se coupent entre deux lignes ou deux paragraphes, jamais au milieu d'un total ou d'une signature, et l'en-tête du tableau se répète.

WhatsApp n'accepte pas de fichier par un lien, d'où deux voies :

- **Partager le PDF** — le partage du téléphone (ou de Windows) : on choisit WhatsApp puis le contact, le fichier part en pièce jointe ;
- **Envoyer un lien** — WhatsApp s'ouvre sur le numéro du client (`wa.me`), avec le message et un lien de téléchargement valable 30 jours. Le PDF est déposé dans le stockage privé **`pieces`** (`<comptoir>/<document>/<horodatage>.pdf`), soumis à la même liste blanche que les tables.

Une pièce envoyée passe de « brouillon » à « envoyé », et garde la trace de ses envois (`envois`).

## Reste à faire

- La marge : prix de revient par article, affichée contre le plafond du Répertoire.
- Les impayés : tableau des factures non soldées et lettre de relance.
