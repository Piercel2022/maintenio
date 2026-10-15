# Modèle de données — Maintenio

## 1. Objet et périmètre

Ce document définit le modèle relationnel cible du MVP de Maintenio.
Il complète l'architecture technique, les spécifications fonctionnelles,
la matrice des rôles et permissions et les workflows.

Il constitue une référence de conception avant l'implémentation des entités
TypeORM et des migrations PostgreSQL.

Les noms d'entités et de colonnes proposés ci-dessous sont indicatifs.
Le schéma définitif sera matérialisé par des migrations versionnées.

## 2. Principes de conception

- PostgreSQL est la source de vérité des données métier.
- Les identifiants techniques utilisent des UUID.
- Les dates et heures sont stockées avec fuseau horaire (`timestamptz`).
- Les horodatages sont générés côté serveur.
- Les montants et quantités utilisent des types numériques adaptés,
  jamais des nombres flottants pour des valeurs nécessitant une précision exacte.
- Les règles métier et les permissions sont vérifiées par l'API.
- Les contraintes relationnelles protègent également l'intégrité des données.
- Chaque ressource métier est isolée dans le périmètre de son organisation.
- Les changements importants sont historisés.
- Les migrations de base de données sont versionnées.
- La synchronisation automatique du schéma TypeORM est désactivée
  dans les environnements partagés et de production.

## 3. Conventions générales

### 3.1. Identifiants et horodatages

Les entités principales utilisent :
- `id` : UUID, clé primaire ;
- `created_at` : date de création ;
- `updated_at` : date de dernière modification lorsque pertinent.

Les historiques conservent leur date d'événement propre.
Un événement historique ne doit pas être réécrit simplement parce que
l'état courant de la ressource évolue.

### 3.2. Isolation par organisation

Les tables métier concernées portent un `organization_id` explicite.

L'organisation autorisée est déterminée à partir du contexte authentifié
et des appartenances vérifiées côté serveur. Une valeur fournie par le
navigateur ne constitue jamais une preuve d'autorisation.

Les relations entre ressources doivent empêcher les associations
inter-organisations. Les clés étrangères composites doivent être utilisées
lorsque nécessaire pour lier les identifiants des ressources à leur
`organization_id`. Lorsque plusieurs ressources doivent appartenir au même
site, les contraintes doivent également garantir la cohérence de leur
`site_id`. Les colonnes référencées par une clé étrangère composite doivent
être couvertes par une contrainte `UNIQUE` ou une clé primaire compatible.

Ces contraintes doivent protéger notamment les relations entre équipement,
incident, ordre de travail, intervention et pièce jointe. Les contrôles de
l'API restent nécessaires, mais ne remplacent pas les contraintes SQL.

Toute lecture, création, modification ou suppression doit vérifier :
- l'organisation de la ressource ;
- les permissions du rôle ;
- le périmètre de site de l'utilisateur ;
- les relations entre les objets concernés.

Les tests doivent couvrir les tentatives d'accès entre deux organisations.

### 3.3. Suppression et archivage

Les organisations, sites, équipements et utilisateurs peuvent nécessiter
un mécanisme de désactivation ou d'archivage selon leur cycle de vie.

L'archivage d'un site ou d'un équipement ne supprime pas les historiques
de maintenance existants.

Les incidents, ordres de travail, interventions et historiques ne sont pas
supprimés physiquement dans le fonctionnement ordinaire du MVP. Toute
procédure exceptionnelle de suppression doit être autorisée, documentée
et compatible avec les obligations légales applicables.

## 4. Identité et organisations

### 4.1. Organization

Représente une organisation cliente.

Champs principaux :
- `id` ;
- `name` ;
- `slug` ;
- `status` ;
- `created_at` ;
- `updated_at`.

Contraintes :
- `slug` est unique ;
- le statut permet de désactiver l'accès à une organisation ;
- la désactivation ne supprime pas les données métier.

### 4.2. User

Représente un compte utilisateur.

Champs principaux :
- `id` ;
- `first_name` ;
- `last_name` ;
- `email` ;
- `password_hash` ;
- `status` ;
- `created_at` ;
- `updated_at`.

Contraintes :
- l'adresse électronique normalisée est unique à l'échelle de la plateforme ;
- le mot de passe haché n'est jamais exposé dans les réponses API ;
- un compte désactivé ne peut pas accéder aux ressources protégées.

### 4.3. OrganizationMembership

Représente l'appartenance d'un utilisateur à une organisation.

Champs principaux :
- `id` ;
- `organization_id` ;
- `user_id` ;
- `role` ;
- `status` ;
- `created_at` ;
- `updated_at`.

Règles :
- une appartenance relie un utilisateur et une organisation ;
- un utilisateur ne peut avoir qu'une appartenance unique par organisation ;
- le rôle appartient à l'ensemble des rôles définis dans la matrice des permissions ;
- une appartenance désactivée ne donne aucun droit opérationnel.

Le MVP peut limiter un utilisateur à une seule organisation active par une
règle applicative, sans imposer cette limitation au modèle relationnel.
La gestion complète de plusieurs organisations par utilisateur reste à confirmer.

Les rôles ne sont pas stockés comme une propriété globale du compte : ils
s'appliquent à une appartenance donnée.

## 5. Sites et équipements

### 5.1. Site

Représente un bâtiment ou un établissement.

Champs principaux :
- `id` ;
- `organization_id` ;
- `name` ;
- `reference` ;
- `address_line_1` ;
- `address_line_2` ;
- `postal_code` ;
- `city` ;
- `country_code` ;
- `site_type` ;
- `status` ;
- `created_at` ;
- `updated_at` ;
- `archived_at`.

Contraintes :
- un site appartient à une seule organisation ;
- une référence de site, lorsqu'elle est renseignée, est unique dans
  son organisation ;
- un site archivé ne reçoit pas de nouvelles opérations ordinaires.

### 5.2. Equipment

Représente un équipement technique maintenable.

Champs principaux :
- `id` ;
- `organization_id` ;
- `site_id` ;
- `name` ;
- `reference` ;
- `equipment_type` ;
- `location_description` ;
- `manufacturer` ;
- `model` ;
- `serial_number` ;
- `commissioned_at` ;
- `status` ;
- `created_at` ;
- `updated_at` ;
- `archived_at`.

Contraintes :
- un équipement appartient à un site et à la même organisation ;
- une référence renseignée est unique dans le périmètre retenu ;
- un équipement archivé conserve son historique ;
- les nouvelles opérations ordinaires ne peuvent pas utiliser un équipement
  archivé.

La gestion avancée des contrats, garanties, compteurs et plans de maintenance
préventive est hors du premier périmètre de données, sauf décision ultérieure.

## 6. Incidents

### 6.1. Incident

Représente un problème ou une demande signalée.

Champs principaux :
- `id` ;
- `organization_id` ;
- `site_id` ;
- `equipment_id`, nullable ;
- `reference` ;
- `title` ;
- `description` ;
- `priority` ;
- `status` ;
- `reporter_user_id` ;
- `assigned_manager_user_id`, nullable ;
- `resolution_summary`, nullable ;
- `resolved_at`, nullable ;
- `closed_at`, nullable ;
- `created_at` ;
- `updated_at`.

Contraintes :
- le site appartient à l'organisation de l'incident ;
- si un équipement est renseigné, il appartient au site sélectionné ;
- le déclarant est autorisé dans l'organisation et sur le site ;
- la référence lisible est unique par organisation ;
- le statut courant respecte les transitions définies ;
- une résolution conserve son auteur, sa date et les informations requises ;
- la clôture est distincte de la résolution.

L'incident peut exister sans ordre de travail immédiat.
Un ordre de travail peut aussi être créé sans incident d'origine.

### 6.2. IncidentComment

Représente une précision ou un commentaire lié à un incident.

Champs principaux :
- `id` ;
- `organization_id` ;
- `incident_id` ;
- `author_user_id`, nullable pour un événement système ;
- `content` ;
- `visibility` ;
- `created_at` ;
- `updated_at`, uniquement si les modifications sont permises.

La visibilité distingue au minimum :
- `REQUESTER_VISIBLE` ;
- `INTERNAL`.

Le demandeur ne consulte que les commentaires qui lui sont accessibles.
Un commentaire interne n'est jamais exposé par erreur à un demandeur.
Les corrections ou suppressions de commentaires doivent respecter la politique
de traçabilité retenue.

### 6.3. IncidentStatusHistory

Conserve les transitions de statut des incidents.

Champs principaux :
- `id` ;
- `organization_id` ;
- `incident_id` ;
- `from_status`, nullable lors de la création ;
- `to_status` ;
- `actor_user_id`, nullable pour certains événements système ;
- `reason`, nullable lorsque non obligatoire ;
- `created_at`.

Chaque transition autorisée produit un enregistrement dans la même
transaction que la mise à jour de l'incident.

L'historique est append-only dans le fonctionnement normal.

## 7. Ordres de travail

### 7.1. WorkOrder

Représente le travail organisé à effectuer.

Champs principaux :
- `id` ;
- `organization_id` ;
- `site_id` ;
- `equipment_id`, nullable ;
- `incident_id`, nullable ;
- `reference` ;
- `title` ;
- `description` ;
- `priority` ;
- `status` ;
- `created_by_user_id` ;
- `scheduled_start_at`, nullable ;
- `scheduled_end_at`, nullable ;
- `due_at`, nullable ;
- `completed_at`, nullable ;
- `cancelled_at`, nullable ;
- `cancellation_reason`, nullable ;
- `created_at` ;
- `updated_at`.

Contraintes :
- le site et l'équipement appartiennent à la même organisation ;
- l'incident éventuel appartient à la même organisation ;
- les dates de planification sont cohérentes ;
- les transitions suivent le workflow défini ;
- la validation et la terminaison exigent les permissions prévues ;
- les ordres terminés ou annulés sont terminaux dans le fonctionnement
  ordinaire du MVP ;
- la référence lisible est unique par organisation.

### 7.2. WorkOrderAssignment

Représente une affectation à un technicien.

Champs principaux :
- `id` ;
- `organization_id` ;
- `work_order_id` ;
- `technician_user_id` ;
- `assigned_by_user_id` ;
- `status` ;
- `assigned_at` ;
- `ended_at`, nullable ;
- `end_reason`, nullable ;
- `created_at` ;
- `updated_at`.

Le statut distingue au minimum une affectation active d'une affectation
terminée ou déclarée non réalisée.

Règles :
- un ordre peut avoir plusieurs affectations ;
- seul un acteur habilité peut créer ou modifier une affectation ;
- une affectation terminée n'est pas supprimée ;
- une affectation non réalisée comporte un motif et un horodatage ;
- une réaffectation conserve l'historique ;
- les techniciens ne consultent que les ordres et informations autorisés ;
- les affectations doivent respecter l'organisation et le périmètre de site.

Une contrainte doit empêcher plusieurs affectations actives identiques pour
le même technicien et le même ordre. Son implémentation exacte sera définie
dans la migration PostgreSQL.

### 7.3. WorkOrderStatusHistory

Conserve les transitions de statut d'un ordre de travail.

Champs principaux :
- `id` ;
- `organization_id` ;
- `work_order_id` ;
- `from_status`, nullable lors de la création ;
- `to_status` ;
- `actor_user_id`, nullable pour certains événements système ;
- `reason`, nullable lorsque non obligatoire ;
- `created_at`.

La transition et son historique sont enregistrés atomiquement.

## 8. Interventions

### 8.1. Intervention

Représente le travail réellement exécuté.

Champs principaux :
- `id` ;
- `organization_id` ;
- `work_order_id` ;
- `assignment_id`, nullable uniquement lorsque la règle métier l'autorise ;
- `performed_by_user_id` ;
- `description` ;
- `started_at`, nullable ;
- `ended_at`, nullable ;
- `status` ;
- `submitted_at`, nullable ;
- `created_at` ;
- `updated_at`.

Règles :
- une intervention appartient à un ordre de travail de la même organisation ;
- l'auteur est autorisé à intervenir sur cet ordre ;
- une intervention distingue brouillon, compte rendu soumis et éventuelle
  validation selon le workflow retenu ;
- un technicien affecté documente le travail qu'il a réellement réalisé ;
- une intervention soumise ne doit pas être réécrite sans trace ;
- la validation finale de l'ordre vérifie tous les comptes rendus et éléments
  obligatoires correspondant au travail effectué.

Une intervention ne remplace ni l'ordre de travail ni l'historique de statut.

### 8.2. InterventionPart

Représente une pièce ou un consommable utilisé lors d'une intervention.

Champs principaux :
- `id` ;
- `organization_id` ;
- `intervention_id` ;
- `name` ;
- `reference`, nullable ;
- `quantity` ;
- `unit` ;
- `created_at`.

La quantité doit être strictement positive. La gestion de stock complète
n'est pas incluse dans le MVP.

## 9. Pièces jointes

### 9.1. Attachment

Représente les métadonnées d'un fichier stocké hors de PostgreSQL.

Champs principaux :
- `id` ;
- `organization_id` ;
- `incident_id`, nullable ;
- `work_order_id`, nullable ;
- `intervention_id`, nullable ;
- `uploaded_by_user_id` ;
- `storage_key` ;
- `original_filename` ;
- `media_type` ;
- `size_bytes` ;
- `checksum`, nullable ;
- `created_at` ;
- `deleted_at`, nullable si une suppression logique est retenue.

Contraintes :
- un fichier est associé à exactement un propriétaire parmi incident,
  ordre de travail ou intervention ;
- une contrainte SQL `CHECK` impose exactement une valeur non nulle parmi
  `incident_id`, `work_order_id` et `intervention_id` ;
- une clé étrangère composite, ou un mécanisme d'intégrité équivalent,
  garantit que le propriétaire appartient à la même organisation ;
- lorsque le propriétaire est rattaché à un site, la cohérence du site
  est également garantie ;
- le stockage est privé ;
- la clé de stockage ne constitue pas une URL publique ;
- type, taille et contenu du fichier sont contrôlés côté serveur ;
- chaque consultation ou téléchargement fait l'objet d'une autorisation ;
- les chemins et clés de stockage ne sont pas modifiables par le client.

Les contraintes SQL, notamment celle imposant exactement un propriétaire,
seront définies dans la migration.

## 10. Audit et notifications

### 10.1. AuditLog

Conserve les événements importants à des fins de traçabilité.

Champs principaux :
- `id` ;
- `organization_id`, nullable uniquement pour les événements explicitement
  globaux ;
- `actor_user_id`, nullable pour les événements système ;
- `action` ;
- `resource_type` ;
- `resource_id` ;
- `metadata` ;
- `created_at`.

Les événements couvrent notamment les changements de priorité, de statut,
d'affectation, d'échéance, les résolutions, les clôtures et les actions
administratives sensibles.

Les métadonnées ne doivent pas contenir de mot de passe, de jeton
d'authentification ou d'autres secrets. Les journaux ne sont pas modifiables
par les utilisateurs ordinaires.

### 10.2. Notification

Représente une notification destinée à un utilisateur.

Champs principaux :
- `id` ;
- `organization_id` ;
- `recipient_user_id` ;
- `type` ;
- `resource_type`, nullable ;
- `resource_id`, nullable ;
- `payload` ;
- `read_at`, nullable ;
- `created_at`.

Les notifications ne contiennent que les informations nécessaires.
L'accès à une notification ne donne jamais accès à une ressource sans
vérification indépendante des permissions.

Le canal initial et le suivi des tentatives d'envoi seront précisés avant
l'implémentation du service de notifications.

## 11. Relations principales

- Organization 1 — N OrganizationMembership.
- User 1 — N OrganizationMembership.
- Organization 1 — N Site.
- Site 1 — N Equipment.
- Site 1 — N Incident.
- Un incident référence zéro ou un équipement ; un équipement peut
  être associé à zéro ou plusieurs incidents.
- Un incident possède zéro ou plusieurs commentaires et entrées
  d'historique de statut.
- Un ordre de travail référence zéro ou un incident ; un incident peut
  être associé à zéro ou plusieurs ordres de travail.
- Site 1 — N WorkOrder.
- WorkOrder 1 — N WorkOrderAssignment.
- WorkOrder 1 — N WorkOrderStatusHistory.
- WorkOrder 1 — N Intervention.
- Intervention 1 — N InterventionPart.
- Incident, WorkOrder et Intervention peuvent chacun posséder des pièces
  jointes selon la règle d'association de Attachment.
- User peut être l'auteur d'un incident, d'un commentaire, d'une affectation,
  d'une intervention ou d'un événement d'audit.
- User peut recevoir plusieurs notifications.

Les cardinalités doivent être matérialisées par des clés étrangères.
Toute relation traversant plusieurs ressources doit vérifier leur cohérence
d'organisation et, lorsque nécessaire, de site.

## 12. Contraintes, index et transactions

### 12.1. Contraintes d'intégrité

Prévoir notamment :
- clés primaires UUID ;
- clés étrangères explicites ;
- unicité de l'email normalisé ;
- unicité des références par organisation ;
- unicité de l'appartenance utilisateur-organisation ;
- cohérence organisation-site-équipement-incident-ordre ;
- quantités positives ;
- dates de début et de fin cohérentes ;
- valeurs de statut contrôlées ;
- un seul propriétaire pour une pièce jointe ;
- absence de doublons d'affectations actives.

### 12.2. Index

Prévoir des index adaptés aux recherches :
- par `organization_id` ;
- par organisation, site, statut et priorité ;
- par organisation et date de création ;
- par ordre de travail et statut d'affectation ;
- par ressource et date dans les historiques ;
- par destinataire et date de création pour les notifications.

Les index définitifs seront validés à partir des requêtes réelles et des plans
d'exécution. Éviter les index redondants.

### 12.3. Transactions

Les opérations suivantes doivent être atomiques :
- création d'un incident et de son historique initial ;
- transition de statut et enregistrement de l'historique ;
- création ou changement d'affectation et journalisation associée ;
- validation et terminaison d'un ordre de travail ;
- création d'une intervention et de ses pièces utilisées.

Les contrôles métier sensibles doivent tenir compte des accès concurrents.
Une vérification préalable côté API ne remplace pas une contrainte SQL.

## 13. Stratégie de références lisibles

Les identifiants UUID servent aux relations et aux API.
Des références lisibles peuvent être présentées aux utilisateurs, par exemple :
- `INC-2026-000124` ;
- `OT-2026-000087`.

Le format, la portée de l'unicité et le mécanisme de génération concurrente
doivent être définis avant implémentation. Deux requêtes concurrentes ne
doivent jamais générer la même référence dans une organisation.

## 14. Hors périmètre initial

Sauf décision contraire, le modèle MVP ne couvre pas encore :
- la planification préventive avancée et les récurrences ;
- les contrats et garanties détaillés ;
- la gestion de stock complète ;
- la facturation ;
- la gestion avancée des fournisseurs externes ;
- les appartenances multi-organisations pleinement activées ;
- les règles SLA et escalades configurables ;
- la conservation documentaire avancée.

Ces évolutions ne doivent pas compromettre l'intégrité du modèle initial.

## 15. Décisions restant à confirmer

Avant l'implémentation des migrations :
1. La politique définitive d'appartenance des utilisateurs à plusieurs organisations.
2. Le format et la génération transactionnelle des références lisibles.
3. Les statuts précis d'une affectation et d'une intervention.
4. Les champs obligatoires pour valider et terminer un ordre de travail.
5. Les règles de conservation et de suppression des données personnelles.
6. Le stockage objet retenu, les limites de fichiers et les formats autorisés.
7. Le mécanisme de contrôle des permissions au niveau des requêtes PostgreSQL
   et/ou de l'API.
8. Le canal de notification du MVP.

## 16. Ordre d'implémentation proposé

1. Valider le modèle et ses décisions ouvertes.
2. Préparer la configuration PostgreSQL et TypeORM.
3. Créer les migrations des organisations, utilisateurs et appartenances.
4. Implémenter l'authentification et l'isolation des organisations.
5. Créer les migrations des sites et équipements.
6. Implémenter les incidents et leurs historiques.
7. Implémenter les ordres de travail et les affectations.
8. Implémenter les interventions et les pièces jointes.
9. Ajouter l'audit et les notifications essentielles.
10. Tester les transitions, les contraintes d'intégrité et l'isolation entre
    au moins deux organisations.
