# Audit technique Worker AppForge V3 + R2 — 24 novembre 2026

## Périmètre et preuve source
- Dépôt Worker : [RMLala/appforg](https://github.com/RMLala/appforg), répertoire `cloud/license-worker`, branche `main`.
- Configuration : `wrangler.toml`; Worker `appforge-license-api`; D1 `appforge-licenses`; binding R2 privé `FILE_BUCKET` vers `appforge-cloud-files`.
- Source auditée : `src/index.js`, `src/r2-files.js`, migrations `0001` à `0008`, tests Worker et workflows Actions.
- Le Worker déployé le 24/11 correspond au commit `bce1407d7ee165587e83ec2de62b8d8447e8619ff` (run de déploiement R2 `45380687909`).
- Aucun secret, code de licence, donnée client, snapshot D1 ou fichier R2 existant n'a été lu ou modifié pendant cet audit.

## 1. Contrat API constaté

| Méthode | Route | Contrôle / résultat |
|---|---|---|
| GET | `/health` | Santé Worker |
| GET | `/v1/config` | Clé publique P-256 et protocole |
| POST | `/v1/trial` | Essai 72 h lié à l'installation |
| POST | `/v1/activate` | Activation licence/appareil |
| POST | `/v1/status` | Vérification statut, appareil, expiration/révocation |
| POST | `/v1/cloud/token` | Émission/renouvellement jeton Cloud depuis la preuve licence/essai |
| GET/PUT | `/v1/cloud/snapshot?resource=...` | Snapshot D1; écriture conditionnelle par révision |
| GET | `/v1/cloud/quota` | Usage D1 + R2 et quotas réels par accès |
| GET | `/v1/cloud/files` | Liste des fichiers binaires de l'accès courant |
| GET/PUT | `/v1/cloud/file?fileId=...` | Téléchargement / upload binaire |

### Authentification Cloud
- Routes snapshot, quota et fichiers : `Authorization: Bearer <jeton Cloud signé>`.
- Le jeton porte le produit, l'accès (licence ou essai), l'installation et une expiration. Durée maximale constatée : 6 heures, bornée par l'expiration de licence/essai.
- Chaque opération recontrôle dans D1 l'état du produit, l'état/expiration de la licence ou de l'essai, les droits Cloud et l'appareil autorisé.
- Une révocation ou désactivation bloque les nouvelles opérations distantes au prochain contact Worker. Les données locales doivent rester conservées.
- Aucun compte Cloud client, mot de passe ou organisation n'est nécessaire.

### Snapshot D1
- GET retourne `resource`, `revision`, `snapshot`, `updatedAt` (ou revision 0 et snapshot null).
- PUT attend `{ expectedRevision, snapshot }`; révision incorrecte : HTTP 409 avec état courant.
- Les données snapshot sont stockées en clair côté D1 dans le code audité; le Worker ne fournit pas de chiffrement de bout en bout.

## 2. Contrat R2 constaté

### Upload
`PUT /v1/cloud/file?fileId=<id>`

- Authentification Bearer obligatoire.
- `fileId` : 1–80 caractères parmi A–Z, a–z, 0–9, underscore et tiret.
- En-tête obligatoire : `X-AppForge-Expected-Revision` (0 pour création, révision courante pour remplacement).
- Corps binaire; taille vide refusée (400), dépassement refusé (413).
- Réponse création : 201; remplacement : 200; conflit de révision : 409.
- Réponse contient ID, révision, taille, SHA-256 et date.

### Download et liste
- `GET /v1/cloud/file?fileId=...` retourne le binaire `application/octet-stream`, avec `X-AppForge-Revision`, `X-AppForge-SHA256` et Content-Length.
- `GET /v1/cloud/files` retourne jusqu'à 100 métadonnées de fichiers, triées par mise à jour.
- Fichier non trouvé : 404; objet R2 absent malgré métadonnée D1 : 502.
- La clé R2 est opaque et préfixée par un hash produit+accès; elle ne contient pas directement le code licence ou l'ID installation.

### Quotas configurés
| Paramètre | Valeur de configuration |
|---|---:|
| Taille maximale par fichier | 10 Mio |
| Nombre maximal de fichiers | 100 |
| Quota R2 essai | 20 Mio |
| Quota R2 licence payée | 100 Mio |
| Quota D1 essai (snapshot) | 5 Mio |
| Quota requêtes Cloud/jour essai | 1 000 |
| Quota requêtes Cloud/jour licence | 10 000 par défaut de migration |

Ces valeurs sont techniques et proviennent de `wrangler.toml`, de la fonction `r2Limits` et des migrations. L'API authentifiée reste la source de vérité pour l'usage/quotas effectifs. Les offres commerciales ne doivent pas être déduites automatiquement de ces constantes.

### Limites R2 bloquantes
- Aucune route DELETE de fichier n'est présente.
- Aucune route de restauration complète/sélective D1+R2 n'est présente.
- Aucun mécanisme applicatif de reprise/résumé d'upload n'est présent dans ce module.
- Les fichiers sont stockés en octets génériques `application/octet-stream`; pas de validation MIME/type réel.
- Le Worker n'applique pas de chiffrement de bout en bout; les fichiers sensibles doivent être chiffrés côté client.
- Pas de politique de rétention/GC générale documentée pour les objets orphelins.
- Le remplacement supprime l'ancien objet après mise à jour du pointeur D1; ce mécanisme n'est pas une fonction de suppression utilisateur.
- Les tests locaux couvrent upload, list, download, conflit, isolation, quota, corps vide/trop grand et nettoyage d'un remplacement. Ils n'attestent pas un transfert R2 authentifié sur l'environnement de production.

## 3. D1 et migrations
Migrations présentes :
- `0001_init.sql` : schéma initial licences/produits/appareils.
- `0002_status_requests.sql` : compteurs de statut.
- `0003_trials.sql` : essais.
- `0004_license_types.sql` : types de licence.
- `0005_admin_crud.sql` : suppression logique/archivage administration.
- `0006_cloud_snapshots.sql` : snapshots Cloud.
- `0007_cloud_entitlements_quotas.sql` : droits Cloud et quotas par licence.
- `0008_cloud_files_r2.sql` : métadonnées fichiers R2.

La table `cloud_files` utilise une clé primaire composite `(access_id, product_id, file_id)`, une clé objet unique, une révision, taille, SHA-256 et dates. Les requêtes R2 filtrent par `access_id` et `product_id`; le jeton est contrôlé avant accès. Les tests synthétiques testent l'isolation entre deux accès.

## 4. Tests et déploiement — état prouvé

### Tests CI
- Run R2 CI sur la branche PR #47 : [45385589895](https://github.com/RMLala/appforg/actions/runs/45385589895), succès le 24/11/2026.
- Étapes confirmées : syntaxe Worker/R2, tests automatisés R2 (isolation, quotas, conflits, remplacement), binding privé et migration non destructive.
- Ce résultat est un test local/simulé du module, pas un test live R2.

### Activation/déploiement R2 en production
- Run manuel : [45380687909](https://github.com/RMLala/appforg/actions/runs/45380687909), succès le 24/11/2026.
- Étapes terminées avec succès : contrôle secrets sans affichage; bucket privé existant/créé; garde-fou migrations n'autorisant que `0008_cloud_files_r2.sql`; application migration; contrôle table `cloud_files`; déploiement Worker; `/health`; `/v1/config`; refus HTTP 401 sur lecture R2 sans jeton.
- Cela confirme le déploiement et le contrôle d'accès anonyme. Cela ne prouve pas que le transfert authentifié R2 fonctionne de bout en bout.

### Test de production proposé mais non exécuté
- PR [#47](https://github.com/RMLala/appforg/pull/47), ouverte, non fusionnée au moment de l'audit.
- Elle ajoute un workflow manuel qui crée deux identités d'essai temporaires, renouvelle des jetons, teste quota, upload/download, SHA-256, conflit, liste, isolation inter-essais, accès anonyme 401 et nettoyage D1/R2.
- Les contrôles CI de cette PR sont verts.
- Ce workflow utilise des essais temporaires sur le D1 de production, pas une licence de test dédiée. Conformément à la règle projet de ne tester les écritures Cloud qu'avec une licence de test dédiée, ne pas lancer ce workflow avant adaptation/validation de la stratégie de test.
- Le test n'a pas été exécuté; aucun fichier temporaire de test n'a été créé par cet audit.

### D1
- Aucun nouveau run read-only D1 n'a été trouvé parmi les exécutions récentes consultées.
- Le dernier inventaire D1 documenté reste celui du 20/11/2026. Il est historique et ne permet pas de conclure sur les Product IDs/licences actuels au 24/11.

## 5. Verdict

| Domaine | Verdict |
|---|---|
| Contrat Worker V3 | Présent dans le code |
| Jeton Cloud et contrôle licence/appareil | Présent dans le code et tests synthétiques |
| Snapshot D1 avec révision | Présent dans le code et tests synthétiques |
| Schéma D1 R2 | Migration 0008 appliquée en production |
| Bucket R2 privé / binding | Déploiement workflow réussi; binding authentifié pas encore prouvé par round-trip |
| Routes R2 upload/list/download | Présentes dans le code; tests simulés verts |
| Upload/download R2 en production | Non testé |
| Suppression/restauration R2 | Non implémentées comme routes |
| Chiffrement E2E | Non implémenté par Worker |
| Synchronisation multi-appareils réelle | Non certifiée |
| Inventaire D1 à jour | Non disponible après le 20/11 |

## 6. Actions prioritaires
1. Adapter le test live R2 pour utiliser une licence de test dédiée (et une procédure de nettoyage vérifiable), puis exécuter uniquement après revue.
2. Ajouter les routes DELETE, restauration/manifeste, nettoyage contrôlé et validation MIME/type; documenter idempotence/reprise et chiffrement client.
3. Exécuter un audit D1 read-only actualisé, sans exposer codes, clients ou identifiants d'installation.
4. Intégrer au client AppForge Studio les routes existantes sans prétendre que delete/restore sont disponibles; afficher quota réel et état de transfert.
5. Tester la synchronisation D1 entre deux appareils physiques et la restauration complète avant validation produit.

**Conclusion :** R2 est déployé côté Worker et les contrôles anonymes sont confirmés. L'implémentation de base upload/list/download existe et ses tests simulés passent. Mais l'upload/download authentifié en production, la suppression, la restauration, le chiffrement client et la synchronisation multi-appareils restent à valider ou développer. Ne pas annoncer « Cloud R2 complet » à ce stade.
