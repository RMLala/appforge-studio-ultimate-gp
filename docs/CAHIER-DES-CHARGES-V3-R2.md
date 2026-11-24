# AppForge Studio Ultimate — Cahier des charges consolidé V1.0 (V3 + D1 + R2)

**Statut :** spécification cible à implémenter; ne constitue pas une preuve de développement ou de test.  
**Application :** AppForge Studio Ultimate, application unique, sans Business Manager séparé, iframe ou seconde authentification.  
**Standard :** `RMLala/android-template/APPFORGE-STANDARD-V3.md`  
**Worker central :** `https://appforge-license-api.lalainaadclick.workers.dev`

## 1. Objectif et règles fondamentales
Construire une application professionnelle, offline-first, regroupant CRM, commerce, projets, production, licences, support et synchronisation Cloud.
- HTML autonome en première étape; Android ensuite selon les formats explicitement sélectionnés avant livraison.
- Devise métier par défaut : MGA (Ariary).
- Une seule navigation et un seul verrouillage local.
- Données locales disponibles hors connexion; aucune perte en cas d'erreur Cloud.
- D1 pour données structurées et métadonnées; R2 uniquement pour pièces jointes binaires nécessaires.
- Aucun compte Cloud, e-mail, mot de passe ou organisation Cloud demandé au client.
- Aucun secret serveur ou accès privilégié dans le client.
- Distinguer prévu, développé, testé, validé et livré.

## 2. Périmètre fonctionnel

### 2.1 Pilotage et administration
Dashboard calculé depuis les données sources : clients, opportunités, devis, commandes, factures, encaissements confirmés, projets, livraisons, licences, support et synchronisation. Filtres par période/statut/client/produit/responsable. Chaque KPI ouvre son détail source. Administration des référentiels, tarifs, rôles internes, modèles, paramètres et journal d'audit. Aucune donnée fictive mélangée aux données réelles.

### 2.2 CRM et commerce
Prospects, clients, contacts, historique, catalogue, éditions, tarifs et droits Cloud; opportunités/pipeline; devis versionnés et lignes détaillées; commandes issues d'un devis accepté; factures, acomptes, échéances, avoirs, reçus numérotés; paiements partiels et rapprochement.
MVola : un justificatif ou une référence crée un paiement « À vérifier ». Seule une validation explicite permet « Confirmé » ou « Rejeté ». Un justificatif n'est jamais une confirmation bancaire. Seuls les paiements confirmés alimentent encaissements et soldes réglés.

### 2.3 Projets unifiés
Chaque projet comporte les onglets :
- **Conception** : brief, cahier des charges, exigences, décisions, maquettes et versions.
- **Commercial** : client, devis, commande, factures et paiements (masqué pour projet personnel).
- **Livraisons** : tâches, jalons, Kanban, calendrier, QA, anomalies, recette et fichiers livrés.
- **Licences** : produit/édition, activation, appareils, droits Cloud, échéances et statut serveur.
Workflow sur mesure : prospect → brief → cahier des charges → devis → commande/acompte → conception → développement → QA → acceptation → livraison → solde → licence → support.

### 2.4 Licences et support
Essai 72 h non renouvelable abusivement; offres mensuelle, annuelle, à vie et propriétaire selon réponse du Worker. Activation liée à l'application, la licence et l'appareil. Afficher état réel, type, édition, dates, temps restant, appareils, droits Cloud et dernier contrôle. Expiration/révocation bloque les fonctions payantes et l'accès distant selon politique serveur, sans supprimer les données locales.

## 3. Architecture locale et modèle de données
IndexedDB recommandé pour données structurées et file d'opérations; localStorage réservé aux préférences non sensibles. UUID stable pour chaque entité et opération, transactions locales atomiques, migrations de schéma versionnées et reprise après fermeture forcée.
Entités minimales : Client, Contact, Product, PricePlan, Opportunity, Quote, QuoteLine, Order, OrderLine, Invoice, InvoiceLine, CreditNote, Payment, Receipt, Project, ProjectRequirement, Task, Milestone, Delivery, QAItem, License, Device, SupportTicket, Attachment, SyncOperation, AuditEvent.
Relations par IDs stables : clientId, quoteId, orderId, invoiceId, projectId, licenseId, attachmentId. Les noms affichés sont des instantanés, jamais des clés. Détecter doublons, références orphelines et incohérences. Documents financiers annulés par contre-écriture plutôt que suppression destructive. Numérotation résistante aux doubles clics et reprises réseau.

## 4. Contrat V3 — Worker et licence
Base unique : `https://appforge-license-api.lalainaadclick.workers.dev`.
- `GET /v1/config` : configuration publique/protocole.
- `POST /v1/activate` : activation licence/appareil.
- `POST /v1/status` : vérification statut, droits et appareil.
- `POST /v1/cloud/token` : jeton Cloud temporaire.
- `GET /v1/cloud/snapshot?resource=default` : lecture snapshot.
- `PUT /v1/cloud/snapshot?resource=default` : écriture snapshot.
Les schémas JSON, codes d'erreur et versions doivent être repris du Worker/SDK existant avant implémentation. Ne pas inventer de champs ni modifier le protocole par supposition.
Parcours : paiement MVola validé par opérateur → licence activée → vérification Worker → jeton temporaire lié à licence/appareil → synchronisation automatique si droits et quotas le permettent. Aucun compte Cloud client.
Appeler `POST /v1/status` au lancement, au retour premier plan et après reconnexion. Contrôler côté serveur licence, application, appareil, expiration, révocation, droits et quotas. Renouveler le jeton après vérification. Une révocation bloque l'accès distant au prochain contrôle; les données locales restent exportables. Hors ligne, afficher que le statut serveur ne peut être confirmé.

## 5. D1 — données structurées et synchronisation
D1 contient données métier, métadonnées, versions et références de fichiers. Le client n'accède jamais directement à D1; le Worker est le seul point d'autorisation.
- Isolation stricte par licence/tenant et application à chaque lecture/écriture.
- UUID, version, horodatages et identifiant d'opération idempotent.
- Synchronisation incrémentale, pagination, déduplication et reprise après timeout.
- Contrôle de concurrence optimiste et détection des modifications concurrentes.
- Audit des opérations sensibles; sauvegarde, rétention et restauration D1 documentées.
Conflits : événements indépendants fusionnables après contrôle d'idempotence; référentiels concurrents signalés; finances/factures/paiements jamais fusionnés par addition aveugle; utiliser événements et contre-écritures. Aucun écrasement silencieux. Suppressions par tombstones versionnés.

## 6. R2 — pièces jointes
R2 stocke uniquement les binaires (images, PDF, Word/Excel, audio/vidéo si réellement utilisés). D1 conserve les métadonnées : ID, tenant/licence, application, entité liée, clé objet, nom assaini, MIME détecté, taille, empreinte, auteur/appareil, dates, état transfert et suppression.
- Bucket privé; pas d'accès public par défaut.
- Le Worker autorise chaque opération après contrôle jeton/licence/appareil/tenant/droits.
- Accès par flux Worker ou URL signée à courte durée; stratégie à confirmer après audit du Worker.
- Validation serveur taille, type réel, extension, empreinte et quotas; noms utilisateur jamais utilisés directement comme clé.
- Quotas par licence/application : capacité, utilisé, restant, nombre de fichiers et erreurs visibles.
- Transfert réessayable/reprenable; empreinte pour corruption/doublons; suppression logique, rétention et suppression physique contrôlée.
- Définir versionnement, sauvegarde et restauration R2 avant production.
**Contrat R2 non établi par les routes V3 listées :** les routes/payloads d'upload, download, delete, quota et restore doivent être découvertes dans le Worker existant ou spécifiées dans une évolution backend. Ne pas inventer d'URL ni de payload. Tant que non vérifié, conserver les fichiers localement/en attente et ne jamais afficher « synchronisé R2 ».

## 7. Offline-first
Toute écriture métier est validée localement avant réseau. File durable avec dépendances, idempotence, temporisation progressive et reprise automatique. États visibles : local, en attente, synchronisé, conflit, erreur, hors ligne. D1 et R2 ont des états de transfert distincts. Une panne Worker ne bloque pas les opérations locales autorisées. La synchronisation manuelle est disponible. Pas de remplacement global de snapshot sans fusion et confirmation.

## 8. Sauvegarde et restauration
Séparer Cloud automatique Worker/D1/R2 et sauvegarde manuelle Google Drive. Pas d'OAuth Google : partage natif Android pour exporter, sélecteur de fichiers pour restaurer; HTML/Windows : import/export local. Export versionné, chiffrement facultatif, intégrité contrôlée. Prévisualisation, rapport de conflits, fusion sans écrasement et confirmation. Restauration complète/sélective selon capacités; vérifier références R2. Ne jamais supprimer les données existantes avant validation de l'import.

## 9. Sécurité
TLS, autorisation Worker à chaque requête, stockage sécurisé des jetons sur Android. PIN obligatoire selon Standard V3, biométrie prioritaire et PIN de secours; limitation des tentatives. Le verrouillage local ne remplace pas une sécurité serveur. Pour HTML, documenter les limites du navigateur. Aucun secret serveur/clé privée dans dépôt ou artefacts. Minimisation des données, consentement diagnostic, journal d'audit, protection XSS, validation import/fichiers. Toute erreur Cloud conserve les données locales.

## 10. UX et exploitation
Interface responsive PC/tablette/mobile, FR/MG selon Standard V3. Navigation unique, aucun iframe ni second login. Écran Cloud/licence : statut réel, type/édition, échéance, droits, appareils, quotas R2, dernière synchronisation, opérations en attente. À propos & Support conforme V3 sans historique des versions. Aucune PWA. Demander les formats de livraison avant génération : HTML, EXE si retenu, APK Debug, Release API23, Release API29, Cloud/licences.

## 11. Migration des données legacy
Inventaire des sources, sauvegarde immuable et empreinte, cartographie ancien→nouveau, prévisualisation des doublons/références orphelines/erreurs, migration en environnement de test, rapprochement des totaux et pièces jointes. Fusion par IDs fiables; arbitrage manuel si ambigu. Aucune suppression/écrasement production. Journal et reprise contrôlée.

## 12. Tests et critères d'acceptation
**Métier :** client→devis→commande→facture→paiement partiel→reçu; calculs et soldes cohérents; MVola en attente exclu des encaissements; confirmation comptée une seule fois; projet avec quatre onglets; KPI reliés aux sources.
**V3 :** activation licence de test dédiée sans compte Cloud; appareil non autorisé refusé; renouvellement/expiration/révocation; jeton expiré refusé; vérification lancement/retour/reconnexion; aucun secret dans HTML/APK/logs/exports.
**D1/offline :** deux appareils sous même licence; deux licences isolées; coupure réseau/redémarrage/reprise sans perte ni doublon; conflits référentiels et financiers arbitrés; restauration avec rapprochement.
**R2 :** upload/read/download/delete/restore; refus inter-licences; fichier trop volumineux, MIME trompeur, corruption, quota dépassé; reprise et empreinte; bucket privé et expiration des accès temporaires.
**Qualité :** tests unitaires, navigateur responsive, navigation, formulaires, import/export, CI; tests Worker séparés. Chaque preuve consigne commit, environnement, données de test, résultat et traces expurgées.

## 13. État initial et écarts
La branche `work/initial-foundation` contient une fondation HTML locale : navigation, CRUD générique, stockage navigateur, import/export JSON, commandes/paiements manuels et KPI locaux.
Non démontrés comme implémentés : intégration Worker V3, jetons Cloud, D1, API/transferts R2, file offline-first, conflits, relations complètes, conversions commerciales, projets unifiés, migration legacy, tests multi-appareils et builds Android.
Le smoke test Worker mentionné dans le Standard V3 (22/11/2026) est limité à /health, /v1/config, migration SQL R2 et refus d'un snapshot non authentifié. Il ne certifie pas le parcours licence/Cloud/R2.

## 14. Ordre de réalisation
1. Modèle de données et relations métier.
2. CRM→devis→commande→facture→paiement.
3. Schémas API V3 vérifiés depuis Worker/SDK.
4. Activation, statut et jetons sur licence de test dédiée.
5. D1, sync incrémentale, idempotence et conflits.
6. Contrat R2 puis upload/download/delete/quota/restore.
7. File offline-first et états visibles.
8. Migration legacy en environnement test.
9. Tests sécurité, intégration deux appareils et preuves.
10. Livrables sélectionnés et validation avant fusion.

**Gouvernance :** branche et PR uniquement; aucun push direct sur `main`. PR en brouillon tant que les exigences bloquantes et preuves ne sont pas satisfaites.
