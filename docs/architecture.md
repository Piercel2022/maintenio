# Architecture technique de Maintenio

## 1. Objectif

Maintenio est une plateforme SaaS de gestion de la maintenance des bâtiments.
Elle centralise les sites, les équipements, les incidents techniques, les ordres
de travail, les interventions et leur historique.

L'architecture doit permettre de gérer plusieurs organisations clientes,
plusieurs sites par organisation et différents profils d'utilisateurs.

## 2. Principes d'architecture

- Séparer l'interface web, l'API et la persistance des données.
- Appliquer les règles métier et les autorisations côté API.
- Isoler strictement les données de chaque organisation.
- Privilégier une architecture modulaire avant d'envisager des microservices.
- Conserver un historique exploitable des incidents et interventions.
- Ne jamais stocker de secrets dans le dépôt Git.
- Tester les règles métier, les permissions et l'isolation des organisations.
- Concevoir les interfaces pour un usage sur ordinateur, tablette et mobile.

## 3. Composants techniques

### 3.1. Application web

Technologies prévues :
- React
- TypeScript
- Vite
- Tailwind CSS
- React Router
- Client HTTP pour communiquer avec l'API

Responsabilités :
- Authentification et navigation.
- Consultation des sites et équipements.
- Déclaration et suivi des incidents.
- Création et suivi des ordres de travail.
- Affectation des techniciens.
- Consultation des historiques et tableaux de bord.

L'interface ne constitue jamais la seule barrière de sécurité.
Toute opération sensible doit être autorisée par l'API.

### 3.2. API

Technologies prévues :
- Node.js
- TypeScript
- NestJS
- API REST versionnée
- TypeORM

Responsabilités :
- Authentification et contrôle d'accès.
- Validation des entrées.
- Application des règles métier.
- Isolation des données par organisation.
- Gestion des sites, équipements, incidents et ordres de travail.
- Historisation des changements importants.
- Déclenchement des notifications et traitements planifiés.

Les routes publiques et protégées seront définies explicitement.
Les erreurs ne devront pas exposer de secrets, de traces internes ou de données
appartenant à une autre organisation.

### 3.3. Base de données

Technologie prévue : PostgreSQL.

La base stockera notamment :
- Les organisations et les utilisateurs.
- Les rôles et les appartenances aux organisations.
- Les sites et les équipements.
- Les incidents et leurs pièces jointes référencées.
- Les ordres de travail et leurs affectations.
- Les interventions et les comptes rendus.
- Les historiques et journaux d'audit nécessaires.

Les relations, contraintes, index et règles de suppression seront définis dans
un modèle de données dédié avant la mise en œuvre complète.

Les migrations de schéma seront versionnées. La synchronisation automatique
du schéma ne sera pas activée en production.

### 3.4. Photos et documents

Les photos d'incidents et les documents seront conservés dans un stockage
d'objets privé, distinct de la base relationnelle.

La base conservera les métadonnées et la référence du fichier.
L'accès aux fichiers devra être autorisé et limité à la bonne organisation.

Les fichiers envoyés devront faire l'objet de contrôles de type, de taille et
d'accès. Les URL temporaires signées pourront être utilisées pour leur consultation.

Le fournisseur de stockage reste à sélectionner.

### 3.5. Traitements automatiques

Des traitements planifiés pourront gérer :
- Les rappels d'échéance.
- Les retards d'ordres de travail.
- Les opérations de maintenance préventive.
- Les notifications liées aux changements de statut.

Le mécanisme d'exécution, la file de tâches et le fournisseur de notifications
seront choisis selon les besoins réels du MVP.

## 4. Modules métier

### 4.1. Organisations et utilisateurs

Gérer les organisations clientes, les comptes utilisateurs, les rôles et les
droits d'accès.

### 4.2. Sites

Représenter les bâtiments ou établissements d'une organisation : hôtels,
restaurants, commerces, immeubles ou autres sites professionnels.

### 4.3. Équipements

Identifier les équipements à maintenir, leur localisation, leurs caractéristiques
utiles et leur historique de maintenance.

### 4.4. Incidents

Permettre de signaler un problème, de le décrire, de le localiser, de définir
sa priorité et d'ajouter des photos.

Un incident est une demande ou un problème à qualifier. Il ne doit pas être
confondu avec l'ordre de travail utilisé pour organiser sa résolution.

### 4.5. Ordres de travail

Formaliser les actions nécessaires à la résolution d'un incident ou à une
opération de maintenance planifiée.

Un ordre de travail peut comporter une priorité, un responsable, une échéance,
un statut, des consignes et un historique.

### 4.6. Interventions

Enregistrer le travail réellement effectué : technicien, dates, diagnostic,
actions réalisées, pièces utilisées si nécessaire et compte rendu.

### 4.7. Tableau de bord

Présenter des indicateurs utiles : incidents ouverts, ordres en retard,
interventions en cours et activité par site.

Les indicateurs devront être calculés à partir de données autorisées pour
l'organisation et l'utilisateur concernés.

## 5. Flux métier principal

1. Un utilisateur déclare un incident sur un site.
2. L'API valide la demande et enregistre l'incident.
3. Un responsable qualifie l'incident et définit sa priorité.
4. Un ordre de travail est créé lorsque la résolution nécessite une action organisée.
5. Le responsable affecte l'ordre à un technicien interne ou à un intervenant autorisé.
6. Le technicien prend en charge le travail et renseigne son avancement.
7. Le compte rendu est enregistré.
8. La résolution est vérifiée, puis l'ordre de travail et l'incident sont clôturés
   selon les règles métier retenues.
9. L'historique reste consultable par les utilisateurs autorisés.

Les statuts définitifs et les transitions autorisées seront précisés dans les
spécifications fonctionnelles.

## 6. Sécurité et isolation des organisations

- Chaque ressource métier doit être rattachée à une organisation lorsque cela
  est pertinent.
- L'organisation autorisée doit être déterminée depuis le contexte
  d'authentification vérifié, et non acceptée aveuglément depuis le client.
- Chaque lecture et écriture doit contrôler l'appartenance et les permissions.
- Les identifiants difficiles à deviner ne remplacent jamais les contrôles d'accès.
- Les mots de passe seront stockés sous forme de condensats sécurisés.
- Les jetons, secrets et mots de passe ne seront jamais consignés dans les logs.
- Les opérations sensibles feront l'objet d'une journalisation adaptée.
- Les sauvegardes et procédures de restauration devront être définies avant
  la mise en production.
- La durée de conservation des données et les procédures de suppression seront
  précisées selon les exigences légales et contractuelles applicables.

## 7. Qualité et tests

Les contrôles prévus comprennent :
- Tests unitaires des règles métier.
- Tests d'intégration de l'API et de la persistance.
- Tests d'authentification et d'autorisation.
- Tests d'isolation entre deux organisations distinctes.
- Tests des transitions de statuts.
- Tests des validations de fichiers.
- Vérifications de lint, de compilation et de format du patch.

Une fonctionnalité ne sera considérée comme terminée qu'après validation de ses
cas nominaux, de ses erreurs attendues et de ses contrôles de sécurité.

## 8. Environnements et configuration

Les environnements de développement, de test et de production devront être
distincts.

La configuration sera fournie par variables d'environnement ou par un gestionnaire
de secrets adapté. Un fichier d'exemple pourra documenter les variables nécessaires,
sans contenir de valeurs confidentielles.

Les migrations de base de données, les sauvegardes, les journaux et la procédure
de déploiement seront documentés avant la mise en production.

## 9. Périmètre du MVP

Le MVP vise à fournir :
- La gestion des organisations et des utilisateurs.
- La gestion des sites et des équipements.
- La déclaration et la qualification des incidents.
- La création, l'affectation et le suivi des ordres de travail.
- Les comptes rendus d'intervention.
- Un tableau de bord opérationnel.
- Les photos d'incidents avec accès sécurisé.
- Les contrôles d'accès et les tests d'isolation.

La maintenance préventive avancée, les intégrations externes, l'IoT, la facturation
et les fonctionnalités analytiques avancées seront étudiés après validation du
socle MVP.

## 10. Décisions restant à confirmer

- Modèle précis des rôles et permissions.
- États des incidents et des ordres de travail.
- Modèle de données détaillé et règles de suppression.
- Fournisseur de stockage des photos.
- Service d'envoi des courriels et notifications.
- Hébergement de l'API, de la base et de l'application web.
- Politique de sauvegarde, de conservation et de restauration.
