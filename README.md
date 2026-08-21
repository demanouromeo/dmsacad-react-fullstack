# DMS ACAD

DMS ACAD est un système de gestion académique multi-établissements composé de deux projets indépendants
réunis dans ce dépôt racine :

- **[`backend/`](backend/README.md)** — API REST Laravel 11, une base de données par établissement scolaire.
- **[`frontend/`](frontend/)** — application React + TypeScript (Vite), également packagée en application
  Android via Capacitor.

Chaque sous-projet possède son propre dépôt Git, son propre `.gitignore` et (pour le backend) son propre
`README.md` détaillé. Ce fichier racine donne uniquement une vue d'ensemble de l'application complète.

## Vue d'ensemble fonctionnelle

L'application permet à plusieurs écoles indépendantes de gérer, chacune dans sa propre base de données :
personnel, élèves, parents, classes, filières, spécialités, matières (y compris les classes à approche par
compétences/APC), attributions de cours, saisie des notes, discipline, effectifs, et les comptes/rôles associés
(ADMIN, TOP_MANAGEMENT, SG, CENSEUR, TEACHER, PARENT, STUDENT, BURSAR). La génération des bulletins n'est pas
encore développée.

## Architecture générale

```
dmsacad-react/
├── backend/     API REST Laravel — une connexion MySQL par établissement + une base transverse
└── frontend/    Application React/TypeScript (Vite), packagée Android via Capacitor
```

- Le **backend** n'a pas de base de données unique : chaque requête précise une `connection` (l'établissement
  ciblé), le contrôleur bascule alors la connexion MySQL par défaut de Laravel sur cette base. Une base
  générique héberge les données transverses (comptes, personnel, liaison élèves).
- L'**authentification** est un JWT maison (`firebase/php-jwt`), pas Laravel Sanctum : jeton d'accès court
  (1h) + jeton de rafraîchissement en cookie httpOnly (7 jours), avec un middleware de rôle (`role:ADMIN,...`).
- Le **frontend** consomme cette API via des classes `*Reader` dédiées par domaine fonctionnel (élèves,
  classes, matières, personnel, ...), gère la session (token en mémoire, restauration silencieuse via le
  cookie de rafraîchissement) et adapte son URL cible (backend distant ou local via XAMPP) depuis un
  sélecteur en page de connexion.
- Certains traitements (ex. calculs liés aux bulletins, effectifs) sont volontairement effectués côté client
  plutôt que dupliqués côté API, pour éviter une nouvelle route d'agrégation backend à chaque écran.

Pour le détail de chaque partie (schéma multi-tenant, contrat d'authentification, conventions d'erreurs côté
backend ; routage, gestion d'état, écrans CRUD côté frontend), consulter :

- [`backend/README.md`](backend/README.md) et `backend/CLAUDE.md`
- `frontend/CLAUDE.md`

## Prérequis

- PHP 8.2+ avec l'extension `pdo_mysql`
- Composer
- MySQL/MariaDB
- XAMPP (Apache + PHP + MySQL) — le backend est conçu pour tourner sous Apache, pas `php artisan serve`
- Node.js/npm (pour le frontend, et pour les quelques assets Vite du backend)
- Android Studio (optionnel, uniquement pour builder l'application mobile via Capacitor)

## Installation rapide

### Backend

```bash
cd backend
composer install
copy .env.example .env
"/c/xampp/php/php.exe" artisan key:generate
```

Placer (ou lier) le dossier `backend` dans le répertoire servi par Apache (`htdocs`), de sorte qu'il soit
accessible à l'URL `http://localhost/dmsacad_backend_dev`. Voir [`backend/README.md`](backend/README.md) pour
le détail de la configuration multi-établissements (`.env`) et le lancement.

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Le frontend cible par défaut le backend distant (`https://dmsacad.com/...`) ; un sélecteur Distant/Local en
page de connexion permet de basculer vers le backend XAMPP local le temps du développement.

## Dépôts Git

`backend/` et `frontend/` sont deux dépôts Git distincts (chacun avec son propre historique et ses propres
remotes) — ce dossier racine n'est pas lui-même un dépôt Git. Les commandes Git doivent donc s'exécuter
depuis l'un ou l'autre sous-dossier, pas depuis la racine.
