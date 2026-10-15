# Maintenio

Maintenio est une plateforme SaaS de gestion de la maintenance des bâtiments.

Elle vise à centraliser les incidents techniques, les équipements, les ordres de
travail, les affectations, les délais et l'historique des interventions.

## Architecture

- `apps/api` : API REST et logique métier.
- `apps/web` : application web.
- `docs` : spécifications fonctionnelles, architecture et décisions techniques.

## Socle technique prévu

- Node.js et TypeScript
- pnpm workspaces
- NestJS pour l'API
- React, Vite et Tailwind CSS pour le web
- PostgreSQL et TypeORM pour les données

## Principes de développement

- Isolation des données entre organisations.
- Contrôles d'accès appliqués côté API.
- Tests avant validation des fonctionnalités.
- Aucun secret dans Git.
- Vérification de la qualité avant chaque commit.
- Aucun push sans validation explicite.

## Statut

Projet initialisé. Les applications et leurs dépendances restent à créer.
