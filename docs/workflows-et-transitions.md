# Workflows et transitions métier — Maintenio

## 1. Objet et portée

Ce document définit les workflows métier du MVP Maintenio :
- le cycle de vie des incidents ;
- le cycle de vie des ordres de travail ;
- le suivi des interventions ;
- les autorisations, préconditions et contrôles applicables aux transitions ;
- les notifications, événements d'audit et tests d'acceptation associés.

Il complète `specifications-fonctionnelles.md` et
`matrice-roles-permissions.md`. En cas de contradiction, une règle de sécurité
plus restrictive prévaut jusqu'à clarification documentée.

Les statuts ne peuvent jamais être modifiés librement par le client web.
L'API constitue la source de vérité et applique les transitions autorisées.

## 2. Principes transversaux

### 2.1. Contrôles obligatoires

Pour chaque commande qui modifie un statut, l'API doit vérifier :

1. l'identité authentifiée de l'utilisateur ;
2. son appartenance à l'organisation concernée ;
3. l'accès au site et à la ressource ;
4. la permission requise pour l'action ;
5. le statut actuel et la transition demandée ;
6. les préconditions métier ;
7. la cohérence des données soumises ;
8. l'enregistrement de l'événement d'audit.

Un identifiant valide ne prouve jamais, à lui seul, qu'un utilisateur peut
accéder à une ressource.

### 2.2. Intégrité des transitions

- Une transition interdite est rejetée sans modifier la ressource.
- Une transition réussie enregistre l'ancien statut, le nouveau statut,
  l'auteur, la date et l'heure, ainsi que le motif lorsqu'il est requis.
- Les horodatages sont produits côté serveur.
- L'historique des transitions n'est pas modifiable par les utilisateurs
  ordinaires.
- Les demandes répétées ne doivent pas produire plusieurs changements
  contradictoires. Les opérations sensibles doivent être conçues pour éviter
  les doubles soumissions.
- Les mises à jour concurrentes doivent détecter un statut devenu obsolète
  plutôt que d'écraser silencieusement une transition récente.

### 2.3. Séparation des objets

Un incident décrit un problème signalé. Un ordre de travail organise une
action de maintenance. Une intervention consigne le travail effectivement
réalisé.

Un ordre de travail peut être lié à un incident, mais il peut également être
créé pour une opération planifiée sans incident.

La clôture d'un incident et la terminaison d'un ordre de travail sont deux
opérations distinctes. L'une ne doit pas entraîner automatiquement l'autre
sans règle métier explicite.

## 3. Workflow des incidents

### 3.1. Statuts

| Statut | Signification |
|---|---|
| Nouveau | Incident déclaré et enregistré, pas encore qualifié. |
| En qualification | Les informations et la nature du problème sont examinées. |
| Pris en charge | Le traitement est organisé ou suivi. |
| En attente | Une information, une pièce ou une action externe est attendue. |
| Résolu | Une solution est déclarée ; une validation ou confirmation peut rester nécessaire. |
| Clos | Le traitement est terminé et la clôture est enregistrée. |
| Rejeté | Le signalement est jugé non recevable, avec un motif. |

### 3.2. Transitions autorisées

| Statut initial | Statut cible | Condition principale |
|---|---|---|
| Nouveau | En qualification | Incident accessible et informations minimales présentes. |
| En qualification | Pris en charge | Qualification suffisante et décision de traitement. |
| En qualification | En attente | Motif et élément attendu renseignés. |
| En qualification | Rejeté | Motif de rejet obligatoire. |
| Pris en charge | En attente | Motif de blocage obligatoire. |
| Pris en charge | Résolu | Description de la résolution renseignée. |
| En attente | En qualification | De nouvelles informations exigent une réévaluation. |
| En attente | Pris en charge | Le blocage est levé et le traitement reprend. |
| Résolu | Clos | Critères de clôture remplis et acteur habilité. |
| Résolu | Pris en charge | Le problème persiste ou la résolution est insuffisante ; motif requis. |
| Clos | Pris en charge | Réouverture exceptionnelle autorisée, justifiée et historisée. |

Toute transition absente de ce tableau est interdite par défaut. Le passage de
Nouveau directement à Pris en charge n'est pas autorisé dans le MVP sans
qualification préalable.

### 3.3. Règles de création et de qualification

- L'auteur est déterminé par l'identité authentifiée, jamais par une valeur
  arbitraire fournie par le client.
- La déclaration est limitée aux sites autorisés pour l'utilisateur.
- Les techniciens peuvent déclarer un incident.
- Les demandeurs ne peuvent consulter que leurs propres incidents.
- Les techniciens peuvent consulter leurs propres déclarations et les
  informations d'incidents nécessaires aux travaux qui leur sont affectés.
- Les prestataires externes sont désactivés par défaut dans le MVP.
- La qualification, la priorité et le rejet nécessitent une permission adaptée.
- Un rejet doit contenir un motif compréhensible et traçable.

### 3.4. Conditions de résolution

Le passage à Résolu exige :
- une description de la solution appliquée ou de la cause identifiée ;
- les éléments de preuve nécessaires, si la nature du problème l'exige ;
- l'enregistrement des interventions ou ordres de travail pertinents lorsqu'ils
  existent ;
- une permission autorisant la déclaration de résolution.

Un incident Résolu peut être clos par un responsable habilité après vérification
des informations requises. La clôture par un administrateur d'organisation
n'est pas automatique : une permission opérationnelle explicite est nécessaire.

### 3.5. Réouverture

La réouverture d'un incident Clos est exceptionnelle.

Elle exige :
- une permission spécifique ;
- un motif obligatoire ;
- la conservation de la clôture précédente dans l'historique ;
- une transition vers Pris en charge ;
- une notification aux personnes concernées.

Le demandeur peut fournir des précisions tant que l'incident n'est pas clos,
selon les règles de sa matrice de permissions. La réouverture par le demandeur
n'est pas autorisée dans le MVP sans décision explicite ultérieure.

## 4. Workflow des ordres de travail

### 4.1. Statuts

| Statut | Signification |
|---|---|
| À planifier | Ordre créé, date et organisation du travail à définir. |
| Planifié | Une date ou une échéance est définie. |
| Affecté | Un ou plusieurs intervenants autorisés sont désignés. |
| En cours | L'exécution du travail a commencé. |
| En attente | Le travail est suspendu ou bloqué. |
| À valider | L'exécution est déclarée terminée et attend une vérification. |
| Terminé | Le travail a été validé. |
| Annulé | Le travail ne doit plus être exécuté. |

### 4.2. Transitions autorisées

| Statut initial | Statut cible | Condition principale |
|---|---|---|
| À planifier | Planifié | Date ou échéance renseignée. |
| À planifier | Affecté | Affectation valide ; date renseignée si le type de travail ou la règle de service l'exige. Si aucune date n'est requise, l'absence de planification doit être explicitement justifiée. |
| À planifier | Annulé | Permission d'annulation et motif requis. |
| Planifié | Affecté | Un ou plusieurs intervenants autorisés sont désignés. |
| Planifié | À planifier | Planification retirée avec motif. |
| Planifié | Annulé | Permission d'annulation et motif requis. |
| Affecté | En cours | Au moins un intervenant autorisé commence le travail. |
| Affecté | Planifié | Révision explicite de la planification. |
| Affecté | Annulé | Permission d'annulation et motif requis. |
| En cours | En attente | Motif de suspension obligatoire. |
| En cours | À valider | Compte rendu et informations requises soumis. |
| En attente | En cours | Le blocage est levé et l'exécution reprend. |
| En attente | Affecté | Le travail doit être réorganisé avant reprise. |
| En attente | Annulé | Permission d'annulation et motif requis. |
| À valider | En cours | Validation refusée ; motif et corrections attendues renseignés. |
| À valider | Terminé | Vérification réussie par un acteur habilité. |

Les statuts Terminé et Annulé sont terminaux dans le MVP. Une correction
ultérieure exige une procédure explicite et historisée ; elle ne doit pas
consister à réécrire silencieusement le statut.

### 4.3. Création et planification

La création d'un ordre de travail exige :
- une organisation et un site autorisés ;
- un titre et une description suffisants ;
- une catégorie ou nature de travail lorsque nécessaire ;
- un lien vers l'incident d'origine, s'il existe ;
- un créateur authentifié ;
- un responsable ou une permission permettant de créer cet ordre.

La planification peut comprendre une date prévue, une échéance, une durée
estimée, des consignes et les équipements concernés.

Une échéance modifiée est historisée. Les règles de calcul des SLA et des heures
ouvrées restent à définir séparément.

### 4.4. Affectations multiples

Un ordre de travail peut être affecté à plusieurs techniciens. Chaque
affectation possède son propre historique : intervenant, auteur de
l'affectation, date de début, date de fin et raison du changement si nécessaire.

- Seul un acteur habilité peut créer, modifier ou retirer une affectation.
- Un technicien ne peut pas s'attribuer un ordre de travail non autorisé.
- Chaque intervenant ne voit que les informations nécessaires à sa mission.
- Une réaffectation ne supprime pas l'historique précédent.
- Les affectations doivent appartenir à l'organisation concernée et respecter
  le périmètre de site autorisé.

Pour le MVP, un technicien affecté peut enregistrer une intervention et
soumettre un compte rendu pour le travail concerné. Le responsable habilité
reste chargé de la validation finale.

Lorsqu'un ordre comporte plusieurs affectations, chaque intervenant ayant
réalisé une partie du travail doit soumettre son compte rendu avant la
validation finale. Si un intervenant affecté n'a finalement pas participé,
le responsable habilité doit enregistrer l'affectation comme « non réalisée »,
avec un motif et un horodatage, sans supprimer son historique. Aucun compte
rendu d'exécution n'est exigé pour une affectation déclarée non réalisée.
La validation doit vérifier que tous les comptes rendus et éléments
obligatoires correspondant au travail effectivement réalisé sont présents.
Une seule intervention ne suffit pas automatiquement à justifier l'ensemble
de l'exécution.

### 4.5. Mise en attente

La transition vers En attente exige :
- un motif ou une catégorie de blocage ;
- une description du besoin pour reprendre ;
- un auteur et un horodatage.

Exemples de motifs : pièce indisponible, accès au site impossible,
autorisation attendue, intervention d'un tiers ou diagnostic complémentaire.

La reprise est enregistrée comme une nouvelle transition. La durée d'attente
est calculable à partir de l'historique.

Aucune suspension automatique du SLA n'est décidée ici : les règles de pause,
de reprise et de calendrier doivent être définies dans la politique SLA.

### 4.6. Validation et terminaison

Le passage à À valider exige un compte rendu suffisant et les informations
obligatoires pour le type de travail.

Le validateur :
- doit posséder une permission explicite ;
- ne peut pas valider un ordre d'une autre organisation ou hors de son périmètre ;
- vérifie les informations et éléments de preuve requis ;
- peut accepter le travail ou le renvoyer En cours avec un motif ;
- ne peut pas effacer l'historique des interventions.

Le passage à Terminé n'est permis qu'après validation. L'administrateur
d'organisation ne bénéficie pas automatiquement du droit de validation.

### 4.7. Annulation

L'annulation exige une permission spécifique et un motif. Elle doit notifier
les intervenants affectés et préserver les informations déjà enregistrées.

Un ordre déjà Terminé ou Annulé ne peut pas être annulé une seconde fois ni
repris par une simple modification du statut.

## 5. Workflow des interventions

### 5.1. Rôle de l'intervention

Une intervention représente une action réalisée dans le cadre d'un ordre de
travail. Elle peut contenir :
- l'ordre de travail associé ;
- l'intervenant authentifié ;
- la date et les heures de début et de fin, si applicables ;
- la description des actions réalisées ;
- le diagnostic, la cause ou les constatations ;
- les pièces ou consommables utilisés ;
- les photos et autres pièces jointes autorisées ;
- les recommandations et suites à donner.

### 5.2. Enregistrement

- L'auteur est déterminé côté API.
- Un technicien ne peut enregistrer une intervention que pour un travail auquel
  il est affecté, selon les permissions applicables.
- Un prestataire n'est admis que si cette fonctionnalité est activée et son
  accès explicitement autorisé.
- Les données et pièces jointes respectent les limites de taille, de format et
  de visibilité configurées.
- Les heures déclarées doivent être cohérentes ; les règles de correction sont
  tracées.
- Une intervention ne peut pas être rattachée à un ordre d'une autre
  organisation.

### 5.3. Correction et historique

Une intervention soumise ou utilisée pour une validation ne doit pas être
supprimée silencieusement.

Toute correction après soumission doit préserver l'auteur, la date, la nature
du changement et les valeurs précédentes nécessaires à l'audit. Les règles
d'édition avant soumission et après validation seront appliquées selon la
matrice des permissions.

L'intervention et l'ordre de travail ont des historiques distincts mais reliés.

## 6. Matrice de responsabilité des transitions

Les principes ci-dessous complètent la matrice des rôles. Ils ne remplacent
pas le contrôle d'accès à chaque ressource.

| Action | Responsable habilité | Technicien affecté | Demandeur | Administrateur |
|---|---|---|---|---|
| Qualifier un incident | Oui | Non | Non | Permission explicite |
| Modifier la priorité | Oui | Non | Non | Permission explicite |
| Déclarer un incident | Oui | Oui | Oui | Oui |
| Ajouter une précision à son incident | Selon périmètre | Selon périmètre | Oui, avant clôture | Selon permission |
| Déclarer une résolution | Oui, si permission accordée | Oui, si permission explicite et travail affecté | Non | Uniquement avec permission opérationnelle explicite |
| Créer et affecter un ordre | Oui | Non | Non | Permission explicite |
| Démarrer un ordre | Selon permission | Oui, si affecté | Non | Permission explicite |
| Suspendre ou reprendre un ordre | Selon permission | Oui, si affecté | Non | Permission explicite |
| Soumettre un compte rendu | Oui | Oui, pour son travail | Non | Permission explicite |
| Valider et terminer un ordre | Oui, si habilité | Non | Non | Permission explicite |
| Clôturer ou rouvrir un incident | Oui, si habilité | Non par défaut | Non | Permission explicite |

Toute permission plus restrictive définie dans la matrice des rôles reste
applicable. Les droits ne sont jamais accordés uniquement parce qu'un utilisateur
porte le titre d'administrateur.

## 7. Notifications

Les événements suivants peuvent déclencher une notification selon les
préférences et paramètres de l'organisation :

- création d'un incident ;
- qualification, changement de priorité ou rejet ;
- prise en charge, mise en attente et reprise ;
- résolution, clôture ou réouverture d'un incident ;
- création, planification, affectation ou réaffectation d'un ordre ;
- démarrage, mise en attente, soumission, refus de validation, terminaison ou
  annulation d'un ordre ;
- ajout d'un compte rendu ou d'une pièce jointe pertinente.

Chaque notification doit respecter les droits d'accès : aucun destinataire ne
reçoit de notes internes ou de données qu'il n'est pas autorisé à consulter.

Une notification ne remplace jamais l'événement métier ni son enregistrement
d'audit. L'échec d'envoi d'une notification ne doit pas annuler une transition
métier déjà validée.

Les canaux, préférences, tentatives, reprises et règles de déduplication seront
définis dans la spécification technique des notifications.

## 8. Audit et traçabilité

Les événements suivants sont historisés au minimum :
- création d'un incident ou d'un ordre de travail ;
- toute transition de statut ;
- modification de priorité, échéance ou consigne importante ;
- affectation, réaffectation et retrait d'un intervenant ;
- soumission, validation, refus de validation et annulation ;
- résolution, clôture et réouverture ;
- ajout ou retrait logique d'une pièce jointe ;
- modification d'une permission affectant l'accès à la ressource.

Un événement d'audit contient au minimum :
- l'organisation concernée ;
- le type et l'identifiant de la ressource ;
- l'action effectuée ;
- l'identité de l'acteur authentifié ;
- la date et l'heure côté serveur ;
- l'état précédent et le nouvel état, si pertinents ;
- le motif ou les métadonnées métier utiles, sans secret inutile.

Les journaux ne doivent pas enregistrer de mots de passe, de jetons
d'authentification ou de données sensibles inutiles.

## 9. Exigences transactionnelles et API

- La validation des transitions doit être centralisée dans la couche métier,
  et non dispersée uniquement dans les contrôleurs HTTP.
- Une transition et son événement d'audit doivent être enregistrés de manière
  cohérente. Une opération ne doit pas réussir si son audit obligatoire échoue.
- Les contrôles de permission et de périmètre sont effectués côté serveur à
  chaque lecture ou écriture.
- Les identifiants de ressources et les statuts fournis par le client sont
  validés ; le client ne peut pas imposer un auteur ou un validateur.
- Les transitions concurrentes doivent être protégées par une stratégie de
  cohérence adaptée, par exemple une transaction avec verrouillage ou contrôle
  de version.
- Les erreurs doivent distinguer une ressource inaccessible, une transition
  invalide et des données métier incomplètes, sans divulguer de données
  appartenant à une autre organisation.
- Les opérations de changement de statut doivent disposer de réponses et
  contrats API documentés.

## 10. Tests d'acceptation obligatoires

### 10.1. Incidents

1. Un utilisateur autorisé crée un incident et devient son auteur enregistré.
2. Un demandeur ne peut lire ni modifier l'incident d'un autre demandeur.
3. Un technicien peut déclarer un incident.
4. Un technicien ne peut pas consulter tous les incidents de l'organisation
   par simple accès à la liste.
5. Une transition non autorisée est rejetée sans modifier le statut.
6. Un incident ne peut être résolu sans les informations obligatoires.
7. Un incident ne peut être clos sans permission et conditions requises.
8. Une réouverture autorisée conserve l'historique de la clôture et exige un motif.
9. Un rejet exige un motif.
10. Un utilisateur d'une autre organisation ne peut accéder à l'incident.

### 10.2. Ordres de travail

11. Un responsable habilité peut créer, planifier et affecter un ordre.
12. Un technicien ne peut démarrer qu'un ordre auquel il est affecté.
13. Un technicien ne peut pas s'affecter lui-même un ordre non autorisé.
14. La mise en attente exige un motif.
15. Un ordre ne peut passer à Terminé sans validation autorisée.
16. Un administrateur sans permission opérationnelle ne peut pas valider un ordre.
17. Un ordre Terminé ou Annulé ne peut pas reprendre par une transition ordinaire.
18. Une réaffectation conserve l'historique précédent.
19. Une transition concurrente fondée sur un ancien statut est rejetée ou
    résolue sans écrasement silencieux.
20. Les contrôles d'organisation et de site sont appliqués à chaque opération.

### 10.3. Interventions et notifications

21. Un technicien affecté peut soumettre un compte rendu.
22. Un technicien non affecté ne peut pas enregistrer une intervention sur
    l'ordre concerné.
23. Une correction après soumission préserve les éléments d'historique requis.
24. Une pièce jointe suit les permissions de la ressource parente.
25. Une notification ne divulgue aucune note interne au demandeur.
26. L'échec d'envoi d'une notification ne supprime pas une transition validée.
27. Chaque transition réussie possède un événement d'audit correspondant.
28. Les tests d'isolation couvrent au moins deux organisations distinctes.

## 11. Décisions retenues pour le MVP

Les règles de conception suivantes sont retenues :

- Les transitions non explicitement autorisées sont refusées.
- La qualification précède la prise en charge d'un incident.
- La résolution et la clôture d'un incident restent distinctes.
- La soumission d'un ordre à validation et sa terminaison restent distinctes.
- Les ordres peuvent avoir plusieurs techniciens affectés.
- Les interventions conservent une trace de leurs auteurs et modifications.
- Les prestataires externes sont désactivés par défaut.
- Les permissions sont vérifiées côté API, pour chaque opération.
- Les transitions, affectations et décisions importantes sont historisées.
- Les SLA ne sont pas suspendus automatiquement lors d'une mise en attente.

## 12. Décisions restant à confirmer

Avant l'implémentation définitive, il faudra confirmer :

1. Les délais SLA par priorité, le calendrier ouvré et les jours fériés.
2. L'effet d'une mise en attente sur les délais contractuels.
3. Les délais et permissions de réouverture d'un incident clos.
4. Les critères précis de clôture et l'éventuelle confirmation du demandeur.
5. La règle de validation lorsque plusieurs techniciens interviennent sur un ordre.
6. Les champs obligatoires par catégorie d'intervention.
7. Les formats, tailles maximales et durées de conservation des pièces jointes.
8. Les canaux de notification et les préférences par rôle.
9. Les permissions exceptionnelles de l'administrateur d'organisation.
10. Les règles de correction d'un ordre terminé ou d'une intervention validée.

Ces décisions doivent être documentées avant d'être traduites en règles
techniques. En attendant, une action non couverte par une permission ou une
transition explicite doit être refusée.

## 13. Ordre de mise en œuvre

1. Modéliser les statuts et transitions dans la couche métier.
2. Implémenter les contrôles d'organisation, de ressource et de rôle.
3. Mettre en place l'historique des statuts et l'audit transactionnel.
4. Développer les transitions d'incident et leurs tests.
5. Développer les ordres de travail et les affectations multiples.
6. Développer les interventions et la validation des comptes rendus.
7. Ajouter les notifications sans couplage transactionnel fragile.
8. Exécuter les tests unitaires, d'intégration, d'autorisation et d'isolation.
9. Documenter les contrats API et les erreurs métier.
10. Valider les décisions ouvertes avant la mise en production.
