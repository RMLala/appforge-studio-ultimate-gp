# AppForge Studio Ultimate — état fonctionnel et modèle métier

## Flux cible
Catalogue → devis → commande confirmée → facture(s) → encaissements → licence vérifiée → livraison → support.

Pour les développements sur mesure :
Prospect → brief / cahier des charges → devis → commande + acompte → conception → production → QA → acceptation → livraison → solde → licence → support.

## Entités et identifiants
Chaque entité doit posséder un identifiant stable généré côté application. Les relations définitives devront utiliser des clés étrangères (clientId, quoteId, orderId, invoiceId, projectId), pas des noms libres. Les noms affichés sont des instantanés utiles, mais ne remplacent pas les références.

## Paiements
Un justificatif MVola déposé ou une référence saisie ne constitue pas une confirmation bancaire. Les états doivent distinguer À vérifier, Confirmé et Rejeté. Seuls les paiements confirmés entrent dans les encaissements et les soldes réglés.

## Projets unifiés
Le projet unique doit présenter les vues Conception, Commercial, Livraisons et Licences. Les projets personnels peuvent masquer la vue Commercial. Business Manager ne doit pas être une application séparée, un iframe ou une seconde authentification.

## V3 Cloud/licences — contrat attendu
Worker central : https://appforge-license-api.lalainaadclick.workers.dev

- GET /v1/config
- POST /v1/activate
- POST /v1/status
- POST /v1/cloud/token
- GET/PUT /v1/cloud/snapshot?resource=default

D1 stocke les données structurées et métadonnées; R2 stocke uniquement les pièces jointes binaires. Le Worker est le seul point d'accès autorisé. Synchronisation offline-first par opérations idempotentes, versions et gestion explicite des conflits. Le Worker expose quota, liste, upload et téléchargement R2 authentifiés. Aucun endpoint DELETE ni restauration R2 n'est présent. Upload/download en production restent à tester avec licence dédiée; les tests locaux simulés ne suffisent pas. Voir [CAHIER-DES-CHARGES-V3-R2.md](CAHIER-DES-CHARGES-V3-R2.md) et [AUDIT-WORKER-V3-R2-2026-11-24.md](AUDIT-WORKER-V3-R2-2026-11-24.md).

## Sécurité et limites actuelles
La fondation HTML actuelle utilise localStorage. Elle ne fournit ni authentification réelle, ni isolation multi-utilisateur, ni protection contre un utilisateur ayant accès au navigateur. Aucune clé secrète ne doit être incluse dans le code client. Les appels privilégiés nécessitent une validation côté Worker.

## État au 24 novembre 2026
Implémenté dans la fondation : navigation responsive, référentiels locaux, CRUD générique, recherche simple, export/import JSON, commandes et registre de paiements manuel, KPI locaux.
Non implémenté : relations référentielles strictes, conversion automatisée des documents, facturation et reçus numérotés, workflow complet du projet, Worker V3, D1/R2, migration legacy, builds Android.
