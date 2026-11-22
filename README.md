# AppForge Studio Ultimate

Application métier unifiée — fondation HTML standalone, version 0.1.1 en cours.

## Démarrage
Ouvrir `index.html` dans Chrome ou Edge. Aucun build n'est requis.

## Modules disponibles
- Tableau de bord avec KPI dérivés des données locales
- Clients / CRM, catalogue, devis
- Commandes, factures, paiements et encaissements
- Projets, tâches de production, livraisons
- Suivi manuel des licences et support
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
- Worker central V3 configuré en référence mais pas encore intégré : aucun appel /v1/config, /v1/activate, /v1/status, /v1/cloud/token ni /v1/cloud/snapshot.
- Pas de synchronisation D1/R2, pas de cloud partagé, pas de gestion de fichiers distants.
- Les identifiants client/devis/facture sont actuellement saisis en texte : les relations strictes et listes de sélection restent à construire.
- Les flux de conversion devis→commande→facture et la génération de reçus numérotés restent à implémenter.
- Le suivi licence local ne vaut pas une activation vérifiée par le serveur.
- Aucune authentification sécurisée : localStorage n'est pas une frontière de sécurité.
- Migration des données des anciennes applications non exécutée.

## Développement
Travailler sur une branche dédiée et proposer les modifications par Pull Request. Ne pas pousser directement sur `main`.
