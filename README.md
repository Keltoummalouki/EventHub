# EventHub

**La plateforme centralisée pour découvrir et réserver des événements au Maroc.**

EventHub est une plateforme web destinée aux résidents et aux touristes qui souhaitent **découvrir, réserver et gérer leurs événements au Maroc** : concerts, festivals, événements sportifs, conférences, expositions et bien plus encore.

Le projet repose sur une architecture **full-stack TypeScript** avec **Next.js**, **NestJS** et **PostgreSQL**. Il comprend un système d'authentification sécurisé, une gestion des billets, une validation des entrées par QR code ainsi que des outils dédiés aux organisateurs et à leur personnel.

> **Statut :** en développement actif — dernière version documentée : `v0.3.0` (mars 2026).

## Sommaire

- [Présentation](#présentation)
- [Fonctionnalités](#fonctionnalités)
- [Stack technique](#stack-technique)
- [Architecture du projet](#architecture-du-projet)
- [Modèle de données](#modèle-de-données)
- [Installation locale](#installation-locale)
- [API REST — authentification](#api-rest--authentification)
- [Sécurité](#sécurité)
- [Tests](#tests)
- [CI/CD](#cicd)
- [Déploiement](#déploiement)
- [Historique des versions](#historique-des-versions)
- [Contribution](#contribution)
- [Notes sur le changement de nom](#notes-sur-le-changement-de-nom)

## Présentation

### Objectif

Simplifier l'accès aux événements au Maroc et proposer une expérience fluide et sécurisée, de la découverte d'un événement jusqu'au contrôle des billets à l'entrée.

### Public cible

- **Résidents :** trouver des événements près de chez eux.
- **Touristes internationaux :** découvrir les activités culturelles, musicales et sportives du Maroc.
- **Passionnés d'événements :** suivre concerts, festivals, conférences, salons, expositions et compétitions sportives.
- **Organisateurs et personnel :** gérer les événements, les participants et les contrôles d'accès.

## Fonctionnalités

### 1. Authentification et comptes

- Inscription et connexion par adresse e-mail et mot de passe.
- Authentification par **JWT** avec access token (15 minutes) et refresh token (7 jours).
- Rotation des refresh tokens pour invalider les anciens tokens après renouvellement.
- Hachage des mots de passe et des refresh tokens avec **bcrypt** (cost factor 12).
- Limitation du nombre de requêtes sur les routes sensibles.
- Gestion des rôles : `ADMIN`, `ORGANIZER`, `STAFF` et `USER`.

### 2. Découverte d'événements et billetterie

- Consultation des événements avec filtres par type, date, ville et prix.
- Réservation de plusieurs billets.
- Billets individuels associés à un QR code.
- Consultation de l'historique des réservations.
- Catégories de billets par événement, comme Standard et VIP.

### 3. Paiements et remboursements

Périmètre prévu pour la billetterie :

- Intégration de moyens de paiement, notamment **Stripe**, **PayPal** et cartes bancaires locales.
- Reçus envoyés par e-mail et disponibles au téléchargement.
- Gestion des remboursements.

> La présence de ces fonctionnalités dans le périmètre du projet ne signifie pas que tous les prestataires de paiement sont déjà intégrés.

### 4. Espace utilisateur

- Gestion des informations du profil et des préférences.
- Historique des réservations et des billets.
- Paramètres de sécurité du compte.

### 5. Scan et validation des QR codes

- QR codes signés avec **HMAC-SHA256** pour limiter la contrefaçon.
- Validation des billets par le personnel autorisé (`STAFF`, `ORGANIZER`).
- Verrouillage pessimiste en base de données pour éviter les doubles scans concurrents.
- Historique d'audit des scans (`QRScanLog`).
- Interface de scan adaptée au mobile.
- Limitation à **30 scans par minute**.
- Tableau de bord organisateur avec statistiques en temps réel et export CSV des participants.

## Stack technique

| Couche | Technologies |
| --- | --- |
| Frontend | Next.js 14+, React, TypeScript |
| Backend | Node.js, NestJS 11, TypeScript |
| Base de données | PostgreSQL 17 (Supabase ou Docker en local) |
| ORM | TypeORM 0.3 |
| Authentification | JWT, Passport.js, bcrypt |
| Validation | `class-validator`, Joi |
| Documentation API | Swagger |
| Conteneurisation | Docker, Docker Compose |
| CI/CD | GitHub Actions |
| Hébergement envisagé | Vercel (frontend), Render (backend), Supabase (base de données) |

## Architecture du projet

```text
EventHub/
├── api/                          # Backend NestJS
│   ├── src/
│   │   ├── auth/                 # Authentification et stratégies Passport
│   │   │   ├── dto/
│   │   │   ├── guards/
│   │   │   ├── strategies/
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   └── auth.module.ts
│   │   ├── users/                # Utilisateurs et rôles
│   │   │   ├── entities/
│   │   │   ├── users.service.ts
│   │   │   └── users.module.ts
│   │   ├── events/               # Événements et catégories de billets
│   │   │   └── entities/
│   │   ├── orders/               # Commandes et billets
│   │   │   └── entities/
│   │   ├── payments/             # Paiements
│   │   │   └── entities/
│   │   ├── database/
│   │   │   └── seeds/             # Initialisation des rôles et comptes
│   │   └── app.module.ts
│   ├── test/                     # Tests
│   ├── .env.example
│   └── package.json
├── web/                          # Frontend Next.js
├── docs/
│   ├── conception/               # Diagrammes UML (draw.io)
│   └── SUPABASE_DEPLOYMENT.md    # Guide de déploiement
├── docker-compose.yml            # PostgreSQL local
└── .github/
    └── workflows/
        └── ci.yml                # Pipeline CI/CD
```

## Modèle de données

| Entité | Description |
| --- | --- |
| `User` | Utilisateur et rôle (`ADMIN`, `ORGANIZER`, `STAFF`, `USER`) |
| `Role` | Rôle et droits associés |
| `Event` | Événement, catégorie, dates, capacité et lieu |
| `TicketCategory` | Catégorie de billet (VIP, Standard, etc.) |
| `Order` | Réservation / commande d'un utilisateur |
| `Ticket` | Billet individuel et QR code signé |
| `Payment` | Transaction de paiement |
| `EventStaff` | Affectation du personnel à un événement |
| `QRScanLog` | Traçabilité et audit des validations de billets |

### Relations principales

- `User` **N:1** `Role`
- `User` **1:N** `Order`
- `User` (organisateur) **1:N** `Event`
- `Event` **1:N** `TicketCategory`
- `Order` **N:1** `User` et **N:1** `Event`
- `Order` **1:N** `Ticket`
- `Order` **1:1** `Payment`

Les diagrammes de conception UML sont disponibles dans [`docs/conception/`](docs/conception/).

## Installation locale

### Prérequis

- **Node.js 18+**
- **Git**
- **Docker** et **Docker Compose** (facultatifs si vous utilisez une base PostgreSQL distante)

### 1. Cloner le dépôt

```bash
git clone https://github.com/Keltoummalouki/EventHub.git
cd EventHub
```

### 2. Préparer la base de données

Pour utiliser PostgreSQL en local via Docker :

```bash
cp api/.env.example api/.env
docker compose up -d
docker compose ps
```

Renseigner les variables dans `api/.env` :

```dotenv
# Configuration PostgreSQL locale d'origine
DB_HOST=localhost
DB_PORT=5432
DB_USER=morocco360_user
DB_PASS=morocco360_pass
DB_NAME=morocco360

NODE_ENV=development

# À remplacer par des secrets forts et différents
JWT_ACCESS_SECRET=change_this_to_a_long_random_secret
JWT_REFRESH_SECRET=change_this_to_another_long_random_secret
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
```

> **Important :** `DB_USER`, `DB_PASS` et `DB_NAME` conservent ici leurs valeurs historiques. Renommer le dépôt en EventHub ne modifie pas automatiquement les utilisateurs et les bases PostgreSQL définis dans `docker-compose.yml`. Ne les changez que si la configuration Docker, les données et les variables d'environnement sont migrées ensemble.

### 3. Démarrer le backend

```bash
cd api
npm install
npm run start:dev
```

API locale : **http://localhost:4001**

### 4. Initialiser les rôles et les utilisateurs de développement

Depuis le répertoire `api/` :

```bash
npm run seed
```

Comptes de démonstration décrits dans la configuration initiale :

| E-mail | Mot de passe de démonstration | Rôle |
| --- | --- | --- |
| `admin@morocco360.ma` | `Admin1234` | `ADMIN` |
| `organizer@morocco360.ma` | `Organizer1234` | `ORGANIZER` |
| `user@morocco360.ma` | `User1234` | `USER` |

> **Sécurité :** ces comptes et mots de passe sont uniquement destinés au développement local. Ne les exposez jamais dans un environnement de production. Leurs adresses conservent également le nom historique jusqu'à la mise à jour des scripts de seed.

### 5. Démarrer le frontend

Dans un **second terminal**, depuis la racine du projet :

```bash
cd web
npm install
npm run dev
```

L'URL exacte du frontend dépend de la configuration de `web/`.

## API REST — authentification

| Méthode | Route | Description | Authentification |
| --- | --- | --- | --- |
| `POST` | `/auth/register` | Créer un compte | Publique |
| `POST` | `/auth/login` | Se connecter | Publique |
| `POST` | `/auth/logout` | Se déconnecter | Bearer (access token) |
| `POST` | `/auth/refresh` | Renouveler les tokens | Bearer (refresh token) |

### Inscription

**Requête — `POST /auth/register`**

```json
{
  "username": "johndoe",
  "email": "john@example.com",
  "password": "Password1",
  "full_name": "John Doe",
  "phone_number": "+212600000000"
}
```

**Exemple de réponse — `201 Created`**

```json
{
  "accessToken": "<jwt-access-token>",
  "refreshToken": "<jwt-refresh-token>"
}
```

**Règles de validation des mots de passe :**

- Entre 8 et 100 caractères.
- Au moins une majuscule.
- Au moins une minuscule.
- Au moins un chiffre.

### Connexion

**Requête — `POST /auth/login`**

```json
{
  "email": "john@example.com",
  "password": "Password1"
}
```

## Sécurité

La sécurité fait partie des objectifs fondamentaux de l'architecture EventHub :

- **JWT à durée limitée :** access token de 15 minutes et refresh token de 7 jours.
- **Rotation des refresh tokens :** chaque renouvellement invalide l'ancien refresh token et génère un nouveau token.
- **Stockage des refresh tokens :** conservation exclusive d'un hash bcrypt, jamais du token brut.
- **Mots de passe :** hachage bcrypt avec un cost factor de 12.
- **Rate limiting :** 5 requêtes/minute pour `/auth/register`, 10 pour `/auth/login`, 30 scans/minute.
- **Validation des données :** validation stricte des DTO avec `class-validator` et propriétés non autorisées filtrées.
- **Autorisations :** contrôle d'accès selon les rôles et les affectations aux événements.
- **Billets :** QR codes signés avec HMAC-SHA256 et protection contre le double scan grâce au verrouillage pessimiste.
- **Traçabilité :** journal d'audit des scans.
- **Configuration :** secrets stockés dans les variables d'environnement, jamais dans Git.

## Tests

Depuis `api/` :

```bash
npm run test       # Tests unitaires
npm run test:cov   # Tests avec couverture
npm run test:e2e   # Tests end-to-end
```

## CI/CD

Le pipeline **GitHub Actions** est défini dans [`.github/workflows/ci.yml`](.github/workflows/ci.yml).

Il effectue automatiquement :

1. L'analyse du code avec **ESLint**.
2. La compilation / vérification du build TypeScript.
3. Les **tests unitaires**.
4. Les **tests E2E**, avec un service PostgreSQL.

## Déploiement

Une architecture d'hébergement envisagée pour EventHub :

| Service | Plateforme |
| --- | --- |
| Frontend Next.js | Vercel |
| API NestJS | Render |
| Base PostgreSQL | Supabase |

Consultez le [guide de déploiement Supabase](docs/SUPABASE_DEPLOYMENT.md) pour les étapes de configuration et de mise en ligne.

## Historique des versions

### `v0.3.0` — Mars 2026

**Système de contrôle d'accès et scanning QR**

- QR codes signés avec HMAC-SHA256.
- Prévention du double scan via verrouillage pessimiste.
- Nouveau rôle `STAFF` pour les agents de contrôle.
- Affectation du personnel par événement (`EventStaff`).
- Journal d'audit des scans (`QRScanLog`).
- Interface de scanning optimisée pour mobile.
- Dashboard organisateur avec statistiques en temps réel.
- Export CSV des participants.
- Rate limiting à 30 scans/minute.
- Health checks pour l'orchestration.
- Documentation Swagger de l'API.
- Validation de la configuration au démarrage avec Joi.
- Filtres d'exception TypeORM personnalisés.

### `v0.2.0` — Mars 2026

**Authentification et sécurisation de l'API**

- Authentification JWT (access et refresh tokens avec rotation).
- Stratégies Passport : `local`, `jwt`, `jwt-refresh`.
- Rate limiting sur les routes d'authentification.
- Seeders pour les rôles et utilisateurs.
- Validation globale des DTOs.

### `v0.1.0` — Février 2026

**Fondations du projet**

- Monorepo `api/` et `web/`.
- Configuration NestJS, TypeORM et PostgreSQL avec Docker.
- Modélisation initiale de 7 entités : `User`, `Role`, `Event`, `TicketCategory`, `Order`, `Ticket`, `Payment`.
- Pipeline CI/CD GitHub Actions.

## Contribution

Les contributions et propositions d'amélioration sont les bienvenues :

1. Forker le dépôt.
2. Créer une branche dédiée : `git checkout -b feature/ma-fonctionnalite`.
3. Développer et tester les modifications.
4. Commiter avec un message clair.
5. Ouvrir une Pull Request vers `main`.

## Notes sur le changement de nom

Le projet s'appelle désormais **EventHub** (anciennement **Morocco360**).

- Le **nom public**, le **titre de la documentation**, l'**URL GitHub** et les commandes de clonage utilisent désormais `EventHub`.
- Les **identifiants internes historiques** (base PostgreSQL, utilisateur Docker, comptes créés par les seeders) restent inchangés dans les exemples ci-dessus pour correspondre à la configuration décrite initialement.
- Une migration de ces identifiants ne doit être effectuée qu'après vérification des fichiers de configuration et des données existantes.

---

**EventHub** — Découvrez les événements du Maroc, réservez vos billets et vivez chaque expérience.
