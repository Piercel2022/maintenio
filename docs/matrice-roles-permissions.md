# Maintenio — Matrice des rôles et permissions

## 1. Objet

Ce document définit les rôles fonctionnels, les permissions, les règles
d'accès aux ressources et les contrôles de sécurité de Maintenio.

Il complète :
- `docs/architecture.md`
- `docs/specifications-fonctionnelles.md`

Les permissions sont appliquées côté API. L'interface web peut masquer les
actions interdites, mais ne constitue jamais un mécanisme d'autorisation.

## 2. Principes d'autorisation

Toute opération est autorisée uniquement si les conditions suivantes sont
réunies :

1. L'utilisateur est authentifié et son compte est actif.
2. Il appartient à l'organisation propriétaire de la ressource.
3. Son rôle autorise l'action demandée.
4. Il possède l'accès au site concerné lorsque cette restriction s'applique.
5. Les règles d'affectation et les transitions métier sont respectées.
6. Les éventuelles restrictions propres à la ressource sont satisfaites.

Un identifiant fourni par le navigateur ne constitue jamais une preuve
d'autorisation.

Les contrôles doivent être appliqués à chaque lecture, création, modification,
suppression, validation, téléchargement et changement de statut.

## 3. Rôles fonctionnels

### 3.1. Administrateur d'organisation — ORG_ADMIN

Responsabilités :
- gérer les utilisateurs de son organisation ;
- attribuer les rôles autorisés ;
- gérer les sites et les équipements ;
- consulter les ressources de son organisation ;
- administrer les paramètres accessibles à son rôle.

L'administration ne confère pas automatiquement le droit d'effectuer toutes
les validations opérationnelles. Les permissions d'administration et les
permissions métier doivent rester distinctes.

### 3.2. Responsable de maintenance — MAINTENANCE_MANAGER

Responsabilités :
- consulter les ressources des sites qui lui sont attribués ;
- qualifier et prioriser les incidents de son périmètre ;
- créer et organiser les ordres de travail ;
- affecter ou réaffecter les techniciens autorisés ;
- suivre les échéances et les blocages ;
- examiner les comptes rendus ;
- valider les travaux et clôturer les incidents selon le workflow ;
- consulter les indicateurs de son périmètre.

Le périmètre par défaut est limité aux sites explicitement attribués.

### 3.3. Technicien — TECHNICIAN

Responsabilités :
- déclarer un incident sur un site auquel il a accès ;
- consulter les ordres de travail auxquels il est affecté ;
- consulter les informations nécessaires à l'exécution ;
- signaler un blocage ou un besoin de matériel ;
- enregistrer des interventions et des comptes rendus ;
- joindre des photos autorisées ;
- soumettre le travail à validation.

Le technicien ne peut pas s'attribuer un travail, réaffecter librement un
collègue, modifier une priorité définitive ou valider son propre travail,
sauf règle explicitement autorisée et auditée.

### 3.4. Demandeur — REQUESTER

Responsabilités :
- déclarer un incident sur un site auquel il est autorisé à accéder ;
- joindre des photos autorisées ;
- consulter exclusivement ses propres incidents ;
- ajouter des précisions ou commentaires tant que l'incident n'est pas clos ;
- consulter le statut et la résolution communiquée.

Le demandeur ne peut pas consulter les incidents d'autres demandeurs, les
notes internes de maintenance, les affectations internes ou les comptes
rendus confidentiels.

### 3.5. Prestataire externe — EXTERNAL_PROVIDER

Ce rôle est prévu pour une activation ultérieure ou conditionnelle dans le MVP.

Responsabilités :
- consulter uniquement les ordres de travail explicitement attribués à son
  compte ou à son entreprise ;
- accéder aux informations strictement nécessaires à la mission ;
- enregistrer l'avancement et le compte rendu ;
- joindre les pièces autorisées.

Le prestataire ne peut ni parcourir les autres ordres de travail, ni consulter
les données internes sans rapport avec sa mission, ni valider son propre
travail.

## 4. Matrice générale des permissions

Légende :
- Oui : autorisé dans le périmètre défini.
- Limité : autorisé sous les conditions décrites dans ce document.
- Non : interdit.

| Action | ORG_ADMIN | MAINTENANCE_MANAGER | TECHNICIAN | REQUESTER | EXTERNAL_PROVIDER |
|---|---|---|---|---|---|
| Gérer les utilisateurs | Oui | Non | Non | Non | Non |
| Attribuer les rôles | Oui, rôles autorisés | Non | Non | Non | Non |
| Gérer les sites | Oui | Limité | Non | Non | Non |
| Gérer les équipements | Oui | Limité | Non | Non | Non |
| Déclarer un incident | Oui | Oui | Oui | Oui | Non par défaut |
| Consulter tous les incidents autorisés | Oui | Oui, sites attribués | Non | Non | Non |
| Consulter ses propres incidents | Oui | Oui | Oui | Oui | Non |
| Qualifier un incident | Oui, si habilité | Oui | Non | Non | Non |
| Définir la priorité | Oui, si habilité | Oui | Non | Non | Non |
| Modifier le statut d'un incident | Selon permission | Oui | Limité | Non | Non |
| Créer un ordre de travail | Selon permission | Oui | Non | Non | Non |
| Affecter un technicien | Selon permission | Oui | Non | Non | Non |
| Consulter les ordres de travail | Oui | Oui, sites attribués | Affectés | Non | Attribués |
| Enregistrer une intervention | Selon permission | Oui | Oui, travaux affectés | Non | Travaux attribués |
| Soumettre un compte rendu | Selon permission | Oui | Oui | Non | Oui |
| Valider un travail | Selon permission | Oui | Non | Non | Non |
| Clôturer un incident | Selon permission | Oui | Non | Non | Non |
| Consulter le tableau de bord global | Oui | Non | Non | Non | Non |
| Consulter les indicateurs de son périmètre | Oui | Oui | Limité | Limité | Limité |
| Consulter l'historique autorisé | Oui | Oui | Limité | Limité | Limité |

La matrice constitue une politique de référence. Chaque permission marquée
« selon permission » doit être déterminée par une règle explicite côté API,
et non par une supposition fondée sur le seul rôle.

Les droits d'administration de l'organisation ne confèrent pas
automatiquement les droits opérationnels de validation d'un travail
ou de clôture d'un incident. Ces actions nécessitent une permission
explicite côté API.

## 5. Périmètre des données

### 5.1. Isolation par organisation

- Chaque ressource métier appartient à une organisation.
- L'organisation autorisée est déterminée à partir du contexte authentifié.
- Une valeur `organizationId` envoyée par le client ne peut pas remplacer ce
  contexte.
- Les relations entre ressources doivent appartenir à la même organisation.
- Toute tentative d'accès transversal est refusée sans divulguer de données.
- Les fichiers héritent des restrictions de la ressource à laquelle ils sont
  associés.

### 5.2. Accès aux sites

- L'administrateur accède aux sites de son organisation.
- Le responsable accède aux sites qui lui sont explicitement attribués.
- Le technicien déclare et exécute des travaux uniquement dans son périmètre
  autorisé.
- Le demandeur déclare des incidents uniquement sur les sites auxquels il a
  accès.
- Le prestataire accède aux informations de site strictement nécessaires à
  ses ordres de travail.

Toute modification de périmètre doit être contrôlée et historisée.

### 5.3. Propriété des incidents

- L'auteur d'un incident est enregistré par l'API à partir de l'utilisateur
  authentifié.
- Un demandeur ne peut pas changer l'auteur d'un incident.
- Un demandeur ne peut lire que ses propres incidents.
- Un technicien peut lire les incidents qu'il a déclarés et ceux dont
  les informations sont nécessaires à un ordre de travail qui lui est
  affecté. Cela ne lui donne pas accès à tous les incidents de
  l'organisation.
- Les commentaires et pièces jointes d'un incident suivent les permissions
  de cet incident.
- Les données internes de maintenance ne sont pas exposées au demandeur par
  défaut.

### 5.4. Affectations

- Un ordre de travail peut être affecté à plusieurs techniciens.
- Chaque affectation possède son propre historique.
- Le responsable autorisé crée, modifie ou retire une affectation.
- Un technicien ne consulte que les ordres auxquels il est affecté, sauf
  permission supplémentaire explicitement définie.
- Une réaffectation conserve l'historique de l'affectation précédente.
- Une affectation ne peut relier que des utilisateurs autorisés de la même
  organisation et du périmètre de site concerné.

## 6. Commentaires et informations internes

Les commentaires doivent être classés au minimum en deux catégories :

- Commentaire visible par les utilisateurs autorisés à suivre l'incident.
- Note interne réservée aux profils de maintenance habilités.

Le demandeur peut ajouter des précisions à ses propres incidents tant que
ceux-ci ne sont pas clos.

Le demandeur ne peut ni modifier ni supprimer l'historique des commentaires
déjà enregistrés. Les corrections éventuelles doivent être tracées.

Les notes internes, les données de diagnostic réservées et les informations
confidentielles ne sont jamais incluses dans une réponse destinée au demandeur
sans règle explicite de publication.

## 7. Règles d'action et de transition

### 7.1. Incidents

- Un incident nouveau peut être qualifié par un responsable autorisé.
- La priorité définitive est modifiée par un profil habilité.
- Le technicien peut ajouter un diagnostic ou signaler un blocage, mais ne
  décide pas seul de la clôture.
- Le demandeur ne modifie pas directement le statut opérationnel.
- Une demande de réouverture par le demandeur déclenche une demande de
  réexamen ; elle ne rouvre pas automatiquement l'incident.
- Toute transition est vérifiée côté API et ajoutée à l'historique.

### 7.2. Ordres de travail

- Seuls les profils habilités créent et affectent un ordre de travail.
- Un technicien peut démarrer ou mettre à jour un travail qui lui est affecté,
  selon les transitions permises.
- Un technicien ne valide pas son propre travail.
- Un travail soumis à validation ne devient terminé qu'après validation
  par un responsable habilité.
- Une annulation, une réaffectation ou une modification importante est tracée.

### 7.3. Interventions

- Une intervention est rattachée à un ordre de travail autorisé.
- L'intervenant est déterminé à partir de l'identité authentifiée.
- Les dates, résultats et durées sont validés côté API.
- La soumission d'un compte rendu ne vaut pas validation finale du travail.
- Toute correction importante après soumission est historisée.

Les transitions définitives et leurs conditions seront détaillées dans une
spécification de workflow avant l'implémentation.

## 8. Administration et cycle de vie des comptes

- Un compte désactivé ne peut plus accéder aux routes protégées.
- Un utilisateur ne peut pas s'octroyer un rôle supérieur.
- Les changements de rôle et de périmètre sont réservés aux profils autorisés.
- Les opérations d'administration sont journalisées.
- Les règles de suppression ou de désactivation ne doivent pas détruire
  l'historique nécessaire à la traçabilité.
- Les appartenances multiples à plusieurs organisations ne sont pas incluses
  par défaut dans le MVP ; elles nécessiteraient un modèle d'appartenance
  explicite.

## 9. Fichiers et pièces jointes

- Les fichiers sont conservés dans un stockage privé.
- L'accès à un fichier exige une autorisation sur la ressource associée.
- Les types de fichiers, tailles maximales et règles de conservation sont
  configurés côté serveur.
- Les liens de téléchargement, même temporaires, ne doivent pas permettre
  un accès au-delà du périmètre autorisé.
- Les fichiers rejetés ou non autorisés ne sont pas publiés.
- Les suppressions et opérations sensibles sont tracées selon la politique
  de conservation retenue.

## 10. Journal d'audit

Les événements suivants doivent être historisés lorsque applicables :

- création d'un incident ou d'un ordre de travail ;
- changement de statut ou de priorité ;
- affectation et réaffectation ;
- modification d'une échéance ;
- soumission, validation, clôture et réouverture ;
- changement de rôle ou de périmètre d'accès ;
- désactivation d'un compte ;
- opérations sensibles sur les pièces jointes.

Chaque événement conserve au minimum l'auteur, la date, la ressource et la
nature de l'action. Les valeurs précédentes et nouvelles sont enregistrées
lorsqu'elles sont nécessaires à la compréhension du changement.

Un utilisateur ne peut pas modifier ou supprimer librement les événements
d'audit.

## 11. Tests d'autorisation obligatoires

Les tests automatisés de l'API doivent couvrir au minimum :

1. Un utilisateur ne peut accéder aux ressources d'une autre organisation.
2. Un demandeur ne peut consulter l'incident d'un autre demandeur.
3. Un demandeur ne peut modifier un statut opérationnel.
4. Un demandeur ne peut lire les notes internes de maintenance.
5. Un technicien ne peut consulter un ordre de travail non attribué.
6. Un technicien peut déclarer un incident sur un site autorisé.
7. Un technicien ne peut déclarer un incident sur un site hors périmètre.
8. Un technicien ne peut s'affecter lui-même un travail non autorisé.
9. Un technicien ne peut valider son propre travail.
10. Un prestataire ne peut accéder qu'aux ordres qui lui sont attribués.
11. Un utilisateur ne peut s'attribuer un rôle supérieur.
12. Un compte désactivé ne peut accéder aux routes protégées.
13. Une pièce jointe ne peut être consultée sans autorisation sur sa ressource.
14. Une relation entre ressources de deux organisations est rejetée.
15. Une transition de statut interdite est rejetée et n'altère pas les données.
16. Les changements d'affectation et de permissions sont historisés.

Chaque scénario sensible doit être testé avec au moins deux organisations
distinctes et, lorsque pertinent, plusieurs utilisateurs du même rôle.

## 12. Décisions retenues pour le MVP

Les règles initiales retenues sont :

- Les techniciens peuvent déclarer des incidents.
- Les demandeurs ne consultent que leurs propres incidents.
- Les responsables sont limités aux sites qui leur sont attribués.
- Un ordre de travail peut être affecté à plusieurs techniciens.
- Les demandeurs peuvent ajouter des précisions à leurs incidents tant qu'ils
  ne sont pas clos.
- Les commentaires visibles et les notes internes sont séparés.
- Les prestataires externes sont désactivés par défaut dans le MVP. Leur
  activation nécessite une décision explicite et la définition de leur
  périmètre d'accès.
- Les appartenances d'un utilisateur à plusieurs organisations sont exclues
  par défaut du MVP.
- Les permissions sont vérifiées côté API à chaque opération.
- Les transitions définitives seront précisées dans une spécification dédiée.

## 13. Conséquences sur la conception technique

Le modèle de données devra notamment permettre de représenter :

- les organisations et les utilisateurs ;
- les rôles et, si nécessaire, les permissions explicites ;
- les accès des utilisateurs aux sites ;
- les incidents et leurs auteurs ;
- les ordres de travail et leurs affectations multiples ;
- les interventions et les comptes rendus ;
- les commentaires visibles et les notes internes ;
- les pièces jointes et leurs ressources parentes ;
- les événements d'audit et les transitions de statut.

La structure exacte des tables, les clés étrangères, les index et les
contraintes seront définis dans `docs/modele-de-donnees.md`.

## 14. Ordre de mise en œuvre

1. Valider cette matrice.
2. Définir les transitions et conditions métier.
3. Concevoir le modèle de données.
4. Implémenter l'authentification et les contrôles d'autorisation.
5. Ajouter les tests d'isolation et de permissions avant les modules métier.
6. Protéger les sites, incidents, ordres de travail, interventions et fichiers.
7. Vérifier les permissions dans les tests d'intégration et de recette.
