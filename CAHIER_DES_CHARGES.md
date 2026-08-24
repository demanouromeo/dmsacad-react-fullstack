# Cahier des charges — DMS ACAD

**Système de gestion académique multi-établissements (web et mobile)**

---

## Sommaire

1. Présentation générale du projet
2. Acteurs du système
3. Périmètre fonctionnel
4. Exigences non fonctionnelles
5. Architecture technique
6. Contraintes et limites connues
7. Livrables
8. Glossaire

---

## 1. Présentation générale du projet

### 1.1 Contexte

De nombreux établissements scolaires (collèges, lycées, CES, ENIEG, CETIC, ...) gèrent aujourd'hui leurs
données pédagogiques et administratives (élèves, personnel, notes, classes) de façon dispersée, sans outil
commun permettant à chaque établissement de conserver l'autonomie de ses propres données tout en s'appuyant
sur une application unique et mutualisée.

### 1.2 Objectifs du projet

DMS ACAD a pour objectif de fournir une plateforme unique de gestion académique, utilisable par plusieurs
établissements indépendants, permettant de :

- centraliser la gestion administrative et pédagogique de chaque établissement (personnel, élèves, classes,
  matières, notes, discipline) tout en garantissant l'isolation des données entre établissements ;
- offrir un accès web et un accès mobile (Android) à la même application ;
- sécuriser l'accès aux données par un système d'authentification et de rôles ;
- réduire la charge de saisie manuelle (import/export en masse, génération automatique de documents PDF/Excel) ;
- rester utilisable en cas de coupure de connexion ponctuelle (export/import hors-ligne des notes).

### 1.3 Enjeux

- **Isolation des données** : les données d'un établissement ne doivent jamais être visibles par un autre.
- **Sécurité des accès** : chaque utilisateur ne doit accéder qu'aux fonctionnalités correspondant à son rôle.
- **Fiabilité de la saisie des notes**, activité la plus sensible et la plus fréquente du système.
- **Adaptabilité** : le système doit couvrir aussi bien un cursus classique que l'approche par compétences (APC).

---

## 2. Acteurs du système

| Rôle | Description | Périmètre d'accès |
|---|---|---|
| **ADMIN** | Administrateur de l'établissement | Accès complet à tous les modules d'administration |
| **TOP_MANAGEMENT** | Direction générale | Accès de supervision |
| **SG** (Surveillant général) | Discipline et suivi des classes qui lui sont rattachées | Discipline, classes assignées |
| **CENSEUR** (Vice-principal) | Suivi pédagogique des classes qui lui sont rattachées | Classes, matières de classe, élèves, discipline (classes assignées) |
| **TEACHER** (Enseignant) | Saisie des notes des matières qui lui sont attribuées | Saisie des notes, discipline (selon attribution) |
| **BURSAR** (Économe) | Gestion financière (hors périmètre actuel développé) | — |
| **PARENT** | Suivi de la scolarité de son/ses enfant(s) | Consultation (hors périmètre actuel développé) |
| **STUDENT** (Élève) | Suivi de sa propre scolarité | Consultation (hors périmètre actuel développé) |

Chaque compte est rattaché à un établissement (une `connection`) et, pour les comptes non-administratifs,
peut être limité aux classes qui lui sont explicitement assignées (ex. le SG et le CENSEUR ne voient que les
classes dont ils ont la charge).

---

## 3. Périmètre fonctionnel

### 3.1 Authentification et sécurité des accès

- Connexion par identifiant/mot de passe, propre à chaque établissement.
- Émission d'un jeton d'accès de courte durée et d'un jeton de rafraîchissement, permettant de maintenir la
  session active sans nouvelle saisie du mot de passe.
- Déconnexion invalidant les jetons côté serveur.
- Restriction de chaque écran et de chaque action aux rôles autorisés.
- Sélection de l'établissement, de l'année scolaire et de la section (francophone/anglophone) avant la
  connexion.

### 3.2 Gestion multi-établissements

- Chaque établissement dispose de ses propres données (personnel, élèves, classes, notes), totalement isolées
  des autres établissements.
- Un compte peut être associé à un seul établissement à la fois.
- Un basculement entre serveur distant (production) et serveur local est possible pour les besoins de test.

### 3.3 Gestion du personnel

- Création, consultation, modification et suppression des membres du personnel.
- Association d'un compte de connexion (identifiant/mot de passe) à chaque membre du personnel.
- Génération automatique d'identifiants et de mots de passe.
- Recherche et export (CSV/PDF) de la liste du personnel.

### 3.4 Gestion des élèves

- Création, consultation, modification et suppression des élèves, par classe.
- Génération automatique de matricule selon la convention de numérotation de l'établissement.
- Import en masse depuis un fichier Excel et export (CSV/PDF).
- Gestion de la photo de l'élève (ajout, recadrage, rotation).
- Suivi des indicateurs par classe (effectifs filles/garçons, redoublants, nouveaux, cas sociaux).
- Suivi des statistiques d'effectifs à l'échelle de l'établissement (par cycle et par section).

### 3.5 Gestion pédagogique

- **Filières** et **spécialités** : création, modification, suppression.
- **Classes** : création, import, gestion par niveau, activation de l'approche par compétences (APC) par
  niveau.
- **Matières** et **groupes de matières** : création, modification, suppression, import.
- **Attribution des matières aux classes** (coefficient, groupe), avec possibilité de dupliquer une
  attribution vers d'autres classes du même niveau.
- **Compétences** (classes APC) : gestion des compétences par matière, par classe et par trimestre.

### 3.6 Attribution des cours

- Attribution d'un enseignant à une matière pour une ou plusieurs classes.
- Consultation des attributions par enseignant et par classe.
- Suppression individuelle ou groupée des attributions.
- Impression de la liste complète des attributions de l'établissement.

### 3.7 Saisie des notes

- Saisie des notes selon deux modes, déterminés par le niveau de la classe :
  - **mode classique** : deux séquences par trimestre ;
  - **mode par compétences (APC)** : une évaluation par compétence.
- Verrouillage global d'une séquence/période, empêchant toute saisie une fois les notes validées.
- Visualisation du taux de remplissage des notes par matière (tableau et graphique).
- Effacement groupé des notes d'une classe/matière/période.
- Export et import hors-ligne des notes (CSV, ou classeur Excel multi-feuilles pour l'ensemble des matières),
  afin de sécuriser la saisie en cas de coupure réseau.
- Édition d'un rapport PDF consolidé des notes de toutes les classes d'une section, pour un trimestre donné.

### 3.8 Discipline

- Suivi des faits de discipline par classe, accessible au SG et au CENSEUR selon les classes qui leur sont
  assignées.

### 3.9 Gestion des comptes utilisateurs

- Gestion centralisée des comptes (identifiants, mots de passe, rôles) de l'ensemble des utilisateurs de
  l'établissement.
- Auto-gestion des identifiants par chaque utilisateur connecté (changement de son propre mot de passe).

### 3.10 Paramétrage

- Paramétrage, par année scolaire, du mode de classification des élèves (« classifié » / « non classifié »)
  utilisé lors de la génération future des bulletins, avec un seuil de taux de participation configurable.

### 3.11 Export et édition de documents

- Export CSV et PDF pour l'ensemble des listes (personnel, élèves, classes, matières, filières, spécialités,
  attributions, effectifs, notes).
- Documents PDF avec en-tête institutionnel (logo, coordonnées de l'établissement), bloc de signature et
  filigrane, générés automatiquement à partir des informations de l'établissement.

### 3.12 Informations générales de l'établissement

- Renseignement des informations de base de l'établissement (nom, type, adresse, contact, logo, lieu et date
  de signature des documents officiels) par année scolaire.

### 3.13 Application mobile

- Empaquetage de l'application dans une application Android, offrant le même périmètre fonctionnel que la
  version web.

### 3.14 Fonctionnalités hors périmètre actuel

Les fonctionnalités suivantes sont identifiées comme besoins futurs, non couvertes par la version actuelle :

- génération des bulletins de notes ;
- envoi de SMS aux parents ;
- édition des livrets scolaires ;
- gestion des promotions et du basculement d'année scolaire ;
- gestion des bourses et des élèves insolvables ;
- espace dédié aux parents et aux élèves ;
- gestion financière (économat).

---

## 4. Exigences non fonctionnelles

### 4.1 Sécurité

- Authentification obligatoire pour l'ensemble des fonctionnalités d'administration.
- Contrôle d'accès basé sur les rôles, appliqué à la fois côté interface et côté serveur.
- Jeton d'accès à durée de vie limitée, renouvelé via un jeton de rafraîchissement stocké de façon protégée
  (cookie non accessible en JavaScript).

### 4.2 Performance et disponibilité

- Les écrans de liste doivent permettre la recherche et le filtrage instantanés côté client.
- Un mécanisme de détection de perte de connexion doit avertir l'utilisateur en temps réel.
- La saisie des notes doit pouvoir être sauvegardée et restaurée localement (export/import) en cas
  d'indisponibilité du réseau.

### 4.3 Compatibilité et portabilité

- L'application doit être accessible depuis un navigateur web standard.
- L'application doit être disponible sous forme d'application Android installable.

### 4.4 Ergonomie

- Retour visuel systématique après chaque action (confirmation, succès, erreur) via un système de
  notifications non bloquantes.
- Confirmation explicite requise avant toute action destructrice (suppression).
- Indicateur de chargement affiché pendant les opérations d'écriture.

### 4.5 Internationalisation

- L'interface doit être disponible en français et en anglais, avec bascule à la volée.

### 4.6 Maintenabilité

- Le code doit suivre une organisation modulaire par domaine fonctionnel, documentée, afin de permettre
  l'ajout de nouveaux modules sans remettre en cause l'architecture existante.

---

## 5. Architecture technique

### 5.1 Vue d'ensemble

```
                         ┌────────────────────────┐
                         │   Application mobile    │
                         │   (Android / Capacitor) │
                         └───────────┬─────────────┘
                                     │
        ┌────────────────────┐      │      ┌───────────────────────────┐
        │  Application web    │◄─────┴─────►│   API REST (Laravel 11)   │
        │  (React / TS / Vite)│    HTTPS     │                            │
        └────────────────────┘   + JWT       └───────────┬───────────────┘
                                                            │
                                     ┌──────────────────────┼──────────────────────┐
                                     │                      │                      │
                              ┌──────▼──────┐        ┌──────▼──────┐       ┌───────▼─────┐
                              │ Base école A│        │ Base école B│  ...  │ Base commune│
                              │  (MySQL)    │        │  (MySQL)    │       │  (comptes)  │
                              └─────────────┘        └─────────────┘       └─────────────┘
```

### 5.2 Backend

- Framework **Laravel 11** (PHP 8.2+), exposant une API REST sous le préfixe `/api`.
- **Architecture multi-tenant à la connexion** : chaque requête précise l'établissement ciblé ; le backend
  bascule dynamiquement la connexion à la base de données correspondante avant d'exécuter la requête. Chaque
  établissement dispose d'un schéma de base de données identique.
- **Authentification par JWT** (jeton d'accès + jeton de rafraîchissement), avec middleware dédié pour le
  décodage du jeton et pour le contrôle des rôles.

### 5.3 Frontend web

- **React 19** avec **TypeScript**, construit avec **Vite**.
- Consommation de l'API via des modules dédiés par domaine fonctionnel.
- Gestion de session en mémoire côté client, avec restauration automatique via le jeton de rafraîchissement.
- Mise en forme avec **TailwindCSS**/**DaisyUI**.
- Génération de documents : **jsPDF** (+ `jspdf-autotable`) pour les PDF, **ExcelJS** pour les classeurs
  Excel multi-feuilles.

### 5.4 Application mobile

- Empaquetage de l'application web via **Capacitor**, ciblant Android.

### 5.5 Modèle de données

- Une base de données par établissement, répliquant le même schéma (années scolaires, classes, élèves,
  personnel, matières, notes, ...).
- Une base transverse pour les données communes (comptes utilisateurs, personnel, liaison des comptes élèves).

---

## 6. Contraintes et limites connues

- L'application backend est actuellement hébergée sous **XAMPP/Apache**, et non via un serveur applicatif
  autonome.
- Le durcissement de la sécurité (authentification et contrôle des rôles) est en cours de généralisation :
  certains points d'accès historiques restent, à ce stade, non protégés.
- Le stockage des mots de passe en clair sur certains comptes est une dette technique connue et identifiée,
  à corriger dans une prochaine itération (passage à un hachage des mots de passe).
- La documentation d'API (Swagger) est installée mais non encore renseignée.
- Aucune contrainte d'intégrité déclarative (clés étrangères en cascade) n'existe au niveau de la base de
  données ; les suppressions en cascade sont gérées applicativement.

---

## 7. Livrables

- Code source du backend (API REST Laravel).
- Code source du frontend web (React/TypeScript).
- Projet Android généré (Capacitor).
- Documentation technique (README, documentation d'architecture).
- Présent cahier des charges.

---

## 8. Glossaire

| Terme | Définition |
|---|---|
| **Multi-tenant** | Architecture permettant à plusieurs établissements d'utiliser la même application tout en gardant des données isolées, ici via une base de données dédiée par établissement. |
| **JWT** (JSON Web Token) | Format de jeton utilisé pour authentifier un utilisateur sans nécessiter de session serveur classique. |
| **APC** | Approche Par Compétences — mode pédagogique où l'évaluation se fait par compétence plutôt que par séquence classique. |
| **NC** | Non Classifié — statut d'un élève n'ayant pas atteint le taux de participation minimal aux évaluations, selon le paramétrage de l'établissement. |
| **RBAC** | Contrôle d'accès basé sur les rôles (Role-Based Access Control). |
| **Connection** | Terme technique désignant, dans ce système, l'identifiant de l'établissement/de la base de données ciblée par une requête. |
