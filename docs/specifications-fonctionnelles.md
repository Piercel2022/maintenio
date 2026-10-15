# Spécifications fonctionnelles de Maintenio

## 1. Objet et objectifs

Maintenio est une plateforme SaaS destinée à centraliser la maintenance
technique des bâtiments et des équipements pour les entreprises multisites,
les hôtels, les restaurants, les commerces et les syndics.

Le produit doit permettre de signaler les problèmes, d'organiser leur
résolution, de suivre les techniciens et de conserver un historique fiable.

Les objectifs fonctionnels sont les suivants :

- Centraliser les demandes de maintenance.
- Réduire les délais de prise en charge et de résolution.
- Clarifier les responsabilités et les échéances.
- Tracer les interventions réalisées.
- Donner aux responsables une vue consolidée de l'activité.
- Garantir la confidentialité des données entre organisations clientes.

## 2. Périmètre fonctionnel du MVP

Le MVP comprend :

- L'authentification et la gestion des utilisateurs.
- La gestion des organisations et des rôles.
- La gestion des sites et des équipements.
- La déclaration et la qualification des incidents.
- La création et le suivi des ordres de travail.
- L'affectation des interventions aux techniciens.
- Les comptes rendus et les pièces jointes.
- Un tableau de bord opérationnel.
- L'historique des changements importants.
- Les contrôles d'accès et l'isolation des organisations.

La facturation, les achats, la gestion avancée des stocks, l'IoT et les
intégrations complexes sont hors périmètre initial.

## 3. Principes métier fondamentaux

### 3.1. Incident

Un incident représente un problème ou une demande signalée : fuite d'eau,
panne électrique, climatisation défaillante, porte endommagée ou autre anomalie.

Un incident possède son propre identifiant, sa description, son site, sa
priorité, son statut, son déclarant et son historique.

Exemple d'identifiant : INC-2026-000124.

### 3.2. Ordre de travail

Un ordre de travail formalise les actions organisées pour résoudre un incident
ou exécuter une opération de maintenance.

Il possède son propre identifiant, son responsable, son échéance, son statut,
ses consignes et ses affectations.

Exemple d'identifiant : OT-2026-000087.

Un incident peut être enregistré sans ordre de travail immédiat. Un ordre de
travail peut également être créé pour une opération planifiée sans incident
initial.

### 3.3. Intervention

Une intervention représente le travail réellement exécuté. Elle contient
notamment le technicien, les dates, le diagnostic, les actions réalisées et
le compte rendu.

Un ordre de travail peut nécessiter plusieurs interventions.

### 3.4. Traçabilité

Les changements significatifs doivent être historisés : changement de statut,
affectation, modification de priorité, échéance, commentaire de résolution
et clôture.

Chaque événement conserve, lorsque cela est pertinent, l'auteur, la date
et les valeurs nécessaires à la compréhension du changement.

## 4. Utilisateurs et rôles

Les rôles ci-dessous constituent la proposition initiale. Les droits précis
seront appliqués côté API.

### 4.1. Administrateur d'organisation

- Gérer les utilisateurs et leurs rôles.
- Gérer les sites et les équipements.
- Consulter les incidents, ordres de travail et interventions de son organisation.
- Configurer les paramètres accessibles à son organisation.
- Consulter les tableaux de bord et l'historique autorisé.

### 4.2. Responsable de maintenance

- Consulter les incidents des sites auxquels il a accès.
- Qualifier les incidents et définir leur priorité.
- Créer les ordres de travail.
- Affecter les techniciens ou prestataires autorisés.
- Définir les échéances et suivre les retards.
- Vérifier les comptes rendus et clôturer les travaux selon ses permissions.
- Consulter les indicateurs opérationnels.

### 4.3. Technicien

- Consulter les ordres de travail qui lui sont affectés.
- Accepter ou prendre en charge un travail selon le processus retenu.
- Mettre à jour son avancement.
- Ajouter un diagnostic, un compte rendu et des photos.
- Signaler un blocage ou un besoin de matériel.
- Soumettre le travail à validation.

### 4.4. Demandeur

- Déclarer un incident sur un site autorisé.
- Ajouter une description et des photos.
- Consulter les incidents qu'il est autorisé à suivre.
- Ajouter des précisions ou commentaires lorsque le processus le permet.
- Consulter le statut et la résolution communiquée.

### 4.5. Prestataire externe

Le rôle prestataire pourra être activé dans le MVP si nécessaire.

- Accéder uniquement aux ordres de travail qui lui sont explicitement attribués.
- Consulter les informations nécessaires à son intervention.
- Ajouter l'avancement et un compte rendu.
- Ne pas consulter les données internes non nécessaires à sa mission.

## 5. Gestion des organisations et des accès

- Chaque organisation cliente dispose d'un espace de données isolé.
- Un utilisateur authentifié ne peut accéder qu'aux organisations auxquelles
  il appartient et aux ressources autorisées.
- Les permissions sont contrôlées côté API à chaque lecture et écriture.
- Une organisation ou un identifiant transmis par le navigateur ne constitue
  jamais, à lui seul, une preuve d'autorisation.
- Les actions d'administration sont réservées aux rôles autorisés.
- Les comptes désactivés ne peuvent plus accéder à l'application.
- Les erreurs d'accès ne doivent pas révéler de données d'une autre organisation.

Les appartenances multiples d'un même utilisateur à plusieurs organisations
restent une décision à confirmer avant la conception définitive des données.

## 6. Gestion des sites

Un site représente un bâtiment ou un établissement professionnel.

Informations envisagées :

- Nom du site.
- Code interne facultatif.
- Adresse et informations de localisation.
- Type de site.
- Contact opérationnel facultatif.
- Statut actif ou archivé.
- Organisation propriétaire.

Règles :

- Un site appartient à une seule organisation.
- Les opérations sur un site nécessitent une autorisation explicite.
- Un site archivé ne doit pas recevoir de nouvelles opérations ordinaires.
- L'historique des opérations existantes doit rester consultable selon les droits.

## 7. Gestion des équipements

Un équipement représente un élément technique maintenable : chaudière,
climatisation, ascenseur, tableau électrique, pompe ou équipement similaire.

Informations envisagées :

- Nom et référence interne.
- Catégorie.
- Site et localisation précise dans le bâtiment.
- Marque et modèle facultatifs.
- Numéro de série facultatif.
- Date d'installation facultative.
- Statut en service, hors service ou archivé.
- Instructions de sécurité et de maintenance, si nécessaires.

Règles :

- Un équipement appartient à un site et à son organisation.
- Un incident peut référencer un équipement ou concerner une zone générale.
- L'historique des incidents et travaux liés à l'équipement est consultable
  par les utilisateurs autorisés.
- L'archivage ne doit pas effacer l'historique de maintenance.

## 8. Déclaration d'un incident

### 8.1. Champs requis

- Site concerné.
- Titre explicite.
- Description du problème.
- Priorité initiale ou règle permettant de la déterminer.
- Déclarant.

L'équipement est facultatif lorsque le problème concerne une zone générale.

### 8.2. Champs complémentaires

- Zone ou localisation précise.
- Photos.
- Date et heure de constat, si différentes de la déclaration.
- Catégorie du problème.
- Commentaires complémentaires.

### 8.3. Règles de validation

- Le site doit être accessible au déclarant.
- L'équipement, s'il est renseigné, doit appartenir au site sélectionné.
- Les champs requis sont validés côté API.
- Les fichiers sont contrôlés selon leur taille, leur type et les règles de sécurité.
- L'auteur et la date de création sont enregistrés par le serveur.
- Le client ne peut pas imposer un auteur ou une organisation non autorisés.

## 9. Cycle de vie des incidents

Statuts proposés :

1. **Nouveau** : incident enregistré, pas encore qualifié.
2. **En qualification** : un responsable analyse le problème.
3. **Pris en charge** : la résolution est organisée ou suivie.
4. **En attente** : une information, une pièce ou un intervenant est attendu.
5. **Résolu** : la solution est déclarée, sous réserve de validation éventuelle.
6. **Clos** : le traitement est terminé et validé.
7. **Rejeté** : la demande est considérée comme invalide ou hors périmètre.

Transitions proposées :

- Nouveau vers En qualification.
- En qualification vers Pris en charge, En attente ou Rejeté.
- Pris en charge vers En attente ou Résolu.
- En attente vers En qualification ou Pris en charge.
- Résolu vers Clos, ou retour à Pris en charge si le problème persiste.
- Clos vers une réouverture uniquement selon une règle autorisée et tracée.

Le système doit empêcher les transitions non autorisées et conserver leur historique.
Les rôles autorisés à effectuer chaque transition seront précisés dans la matrice
de permissions.

## 10. Priorités et délais

Niveaux de priorité proposés :

- **Basse** : problème sans conséquence opérationnelle immédiate.
- **Normale** : problème à traiter dans le flux courant.
- **Haute** : gêne importante ou risque de dégradation rapide.
- **Critique** : risque pour la sécurité, dommage important ou interruption majeure.

La priorité décrit l'urgence de traitement ; elle ne remplace pas une évaluation
des risques de sécurité.

Les délais cibles de première prise en charge et de résolution doivent être
configurables selon le contrat, le site et la priorité.

Les durées exactes et le calcul des heures ouvrées restent à confirmer.
En présence d'un danger immédiat, l'application doit rappeler de suivre les
procédures de sécurité et de contacter les services compétents si nécessaire.

## 11. Création et gestion des ordres de travail

### 11.1. Informations requises

- Titre ou objet du travail.
- Site concerné.
- Description ou consignes.
- Priorité.
- Statut.
- Créateur autorisé.

L'incident d'origine, l'équipement, le responsable, le technicien et l'échéance
sont renseignés selon la nature du travail.

### 11.2. Cycle de vie proposé

1. **À planifier** : travail créé mais pas encore planifié.
2. **Planifié** : une date ou une échéance est définie.
3. **Affecté** : un technicien ou prestataire est désigné.
4. **En cours** : le travail a commencé.
5. **En attente** : le travail est bloqué ou suspendu.
6. **À valider** : l'exécution est déclarée terminée et attend une vérification.
7. **Terminé** : le travail est validé.
8. **Annulé** : le travail ne doit plus être exécuté.

Les transitions doivent être contrôlées côté API. Un travail annulé ou terminé
ne peut être modifié que selon des règles spécifiques et historisées.

### 11.3. Règles d'affectation

- Le responsable et le technicien doivent être autorisés pour l'organisation.
- Les affectations doivent être historisées.
- Un changement d'affectation doit être explicite.
- Le technicien ne peut pas accéder aux ordres qui ne lui sont pas autorisés.
- Une échéance dépassée doit pouvoir être identifiée sans modifier artificiellement
  le statut du travail.

## 12. Exécution d'une intervention

Le compte rendu doit permettre de comprendre le travail réalisé.

Informations envisagées :

- Ordre de travail concerné.
- Intervenant.
- Date et heure de début et de fin, si applicables.
- Diagnostic.
- Actions effectuées.
- Résultat.
- Blocages ou travaux restant à réaliser.
- Photos avant et après, si pertinentes.
- Recommandations de suivi.

Règles :

- Une intervention ne peut être rattachée qu'à un ordre de travail autorisé.
- Les dates et durées incohérentes doivent être rejetées ou soumises à validation.
- Une intervention soumise à validation ne doit pas être considérée automatiquement
  comme un travail terminé.
- Les corrections importantes après soumission doivent être tracées.

## 13. Photos et pièces jointes

- Les photos sont associées à un incident, une intervention ou un ordre de travail.
- Les fichiers sont conservés dans un stockage privé.
- La base de données conserve les métadonnées nécessaires.
- Les téléchargements et consultations nécessitent une autorisation.
- Les types de fichiers autorisés et la taille maximale sont configurés.
- Les fichiers ne doivent pas être rendus publics par défaut.
- Les suppressions et durées de conservation suivent les règles définies par le produit.

La limite de taille, les formats autorisés et le fournisseur de stockage restent
à confirmer avant l'implémentation.

## 14. Tableau de bord

Le tableau de bord initial doit présenter, selon les permissions :

- Nombre d'incidents ouverts.
- Incidents par priorité.
- Incidents en attente.
- Ordres de travail en cours.
- Ordres de travail en retard.
- Travaux à valider.
- Répartition de l'activité par site.
- Activité récente et interventions récemment terminées.

Chaque indicateur doit être calculé à partir de données autorisées pour
l'organisation et l'utilisateur concernés.

Les filtres par période, site, priorité et statut seront étudiés pour le MVP.
Les objectifs de performance et les règles précises de calcul seront documentés
avant les tests de recette.

## 15. Notifications

Notifications envisagées :

- Nouvel incident.
- Incident critique ou de priorité haute.
- Ordre de travail affecté.
- Changement d'échéance ou d'affectation.
- Travail en retard.
- Travail soumis à validation.
- Résolution ou clôture.

Le canal initial, probablement le courrier électronique et les notifications
dans l'application, reste à confirmer.

Les notifications ne doivent contenir que les informations nécessaires et ne
doivent jamais contourner les contrôles d'accès.

## 16. Critères d'acceptation du MVP

### Authentification et autorisations

- Un utilisateur autorisé peut se connecter.
- Un utilisateur désactivé ne peut plus accéder aux ressources protégées.
- Un utilisateur ne peut pas consulter les données d'une organisation étrangère.
- Une opération interdite est rejetée côté API, même si elle est appelée directement.

### Sites et équipements

- Un utilisateur autorisé peut créer et consulter un site.
- Un équipement ne peut pas être associé à un site d'une autre organisation.
- L'archivage préserve l'historique existant.
- Un site ou équipement non autorisé ne peut pas être utilisé pour créer un incident.

### Incidents

- Un demandeur autorisé peut déclarer un incident valide.
- Les champs requis et les relations sont validés côté serveur.
- Une photo autorisée peut être jointe et consultée par les utilisateurs habilités.
- Une transition de statut interdite est rejetée.
- L'auteur, la date de création et les changements significatifs sont traçables.

### Ordres de travail et interventions

- Un responsable autorisé peut créer et affecter un ordre de travail.
- Un technicien peut consulter uniquement les travaux auxquels il a accès.
- Le technicien peut enregistrer un compte rendu autorisé.
- Un ordre de travail ne peut être terminé que selon le processus de validation retenu.
- Les changements d'affectation, de statut et d'échéance sont historisés.

### Tableau de bord

- Les indicateurs correspondent aux données accessibles à l'utilisateur.
- Les filtres ne permettent pas d'extraire des données d'une autre organisation.
- Les ordres en retard sont calculés selon les échéances et règles définies.

## 17. Stratégie de tests fonctionnels

Les tests doivent couvrir :

- Les scénarios nominaux de chaque module.
- Les champs manquants et les valeurs invalides.
- Les transitions de statut autorisées et interdites.
- Les rôles et permissions.
- L'isolation entre au moins deux organisations.
- Les affectations de techniciens.
- Les fichiers non autorisés, trop volumineux ou invalides.
- Les échéances et le calcul des retards.
- La cohérence entre incident, ordre de travail et intervention.
- Les erreurs attendues sans fuite d'informations sensibles.

Les tests automatisés de l'API complètent les tests manuels de recette.

## 18. Décisions à confirmer avant le développement

- Les rôles définitifs et leur matrice de permissions.
- L'accès d'un utilisateur à plusieurs organisations.
- Les transitions exactes des incidents et ordres de travail.
- Les règles de validation et de réouverture.
- Les délais cibles par priorité et les heures ouvrées.
- La possibilité d'affecter plusieurs techniciens à un ordre de travail.
- Les formats et limites des pièces jointes.
- Le canal de notification initial.
- Les règles d'archivage, de suppression et de conservation.
- Les indicateurs et filtres obligatoires du tableau de bord.

## 19. Ordre de mise en œuvre recommandé

1. Finaliser les rôles, permissions et règles de statut.
2. Définir le modèle de données.
3. Construire l'authentification et l'isolation des organisations.
4. Implémenter les sites et les équipements.
5. Implémenter les incidents et les pièces jointes.
6. Implémenter les ordres de travail et les affectations.
7. Implémenter les interventions et leur validation.
8. Construire le tableau de bord.
9. Ajouter les notifications essentielles.
10. Exécuter les tests de sécurité, de recette et de non-régression.
