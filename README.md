# AppForge Studio Ultimate

Application métier unifiée — fondation HTML standalone, version 0.1.1 en cours.

## Démarrage
Ouvrir `index.html` dans Chrome ou Edge. Aucun build n'est requis.

## Modules disponibles
- Tableau de bord avec KPI dérivés des données locales
- Clients / CRM, catalogue, devis
- Commandes, factures, paiements et encaissements
- Projets, tâches de production, livraisons
- Suivi manuel des licences commerciales et support
- Écran « Licence & Worker V3 » : URL officielle préconfigurée, validation HTTPS/domaine, mémorisation locale et test public GET /v1/config avec affichage de la réponse
- Studio / brief, sauvegarde locale et paramètres

## Données et contrôles
- Devise MGA (Ariary).
- Persistance navigateur via localStorage.
- Export JSON et import-fusion par identifiant, sans suppression automatique.
- Import JSON limité à 10 Mo.
- Montants validés comme nombres positifs.
- Un paiement MVola ne devient confirmé qu'après validation manuelle et saisie d'une référence/preuve.
- Les encaissements du tableau de bord ne comptent que les paiements au statut « Confirmé ».

## Limites non levées
- Worker V3 : écran de configuration ajouté; le test public GET /v1/config est déclenché manuellement depuis le navigateur. L’activation /v1/activate, le statut /v1/status, le jeton /v1/cloud/token, les snapshots D1 et les fichiers/quota R2 ne sont pas encore raccordés au workflow métier.
- Pas de synchronisation D1/R2, pas de cloud partagé, pas de gestion de fichiers distants.
- Les identifiants client/devis/facture sont actuellement saisis en texte : les relations strictes et listes de sélection restent à construire.
- Les flux de conversion devis→commande→facture et la génération de reçus numérotés restent à implémenter.
- Le suivi licence local ne vaut pas une activation vérifiée par le serveur.
- Aucune authentification sécurisée : localStorage n'est pas une frontière de sécurité.
- Migration des données des anciennes applications non exécutée.

## Développement
Travailler sur une branche dédiée et proposer les modifications par Pull Request. Ne pas pousser directement sur `main`.


## Cahier des charges V3 + D1 + R2
La spécification consolidée est dans [docs/CAHIER-DES-CHARGES-V3-R2.md](docs/CAHIER-DES-CHARGES-V3-R2.md). Elle distingue les exigences cibles des fonctions effectivement intégrées. À ce stade, Worker V3, D1, R2 et la synchronisation offline-first ne sont pas implémentés dans cette fondation.
