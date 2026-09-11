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

## Reste à faire

- La maquette d'impression de NASSRA n'est pas portée : les deux comptoirs impriment pour l'instant la feuille ZADNA.
- La charte visuelle est encore dans le code (`CHARTES`) plutôt que dans les réglages. La déplacer permettrait de créer un comptoir sans toucher au fichier.
