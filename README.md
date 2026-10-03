# Spring-Boot-Angular-IPM-paiements-mobiles-GED-RAG
SEN-PAY  SEN-PAY — Plateforme intelligente de gestion des cotisations IPM et paiements mobiles
# 🇸🇳 SEN-PAY

### Plateforme intelligente de gestion des cotisations IPM et des paiements mobiles

**SEN-PAY** est une plateforme FinTech / GovTech conçue pour digitaliser la gestion des **cotisations IPM**, des **bordereaux de paiement**, des **transactions mobiles** et des **documents justificatifs**.

L'objectif est de proposer une architecture moderne, sécurisée et évolutive permettant aux entreprises de gérer leurs déclarations et paiements IPM depuis une interface web.

---

##  Objectif

SEN-PAY permet à un employeur de :

* gérer son compte entreprise ;
* gérer ses salariés affiliés à une IPM ;
* consulter ses cotisations ;
* générer et consulter ses bordereaux ;
* initier un paiement ;
* choisir un opérateur de paiement mobile ;
* suivre l'état d'une transaction ;
* recevoir un justificatif de paiement ;
* consulter l'historique des opérations ;
* archiver les documents associés.

La plateforme est conçue pour pouvoir intégrer différents fournisseurs de paiement grâce à une architecture basée sur une **abstraction des moyens de paiement**.

---

## 🚀 Fonctionnalités

### 🏢 Gestion des entreprises

* Création et gestion des employeurs
* Informations administratives
* Numéro de compte IPM
* Identification fiscale
* Gestion des contacts
* Association à une organisation / tenant

### 👥 Gestion des salariés

* Ajout des employés
* Informations personnelles et professionnelles
* Numéro d'affiliation
* Historique des cotisations
* Statut d'affiliation

### 📄 Gestion des cotisations

* Création des périodes de cotisation
* Calcul des montants
* Génération des bordereaux
* Consultation des échéances
* Suivi du statut du bordereau

### 💳 Paiement mobile

SEN-PAY est conçu pour supporter plusieurs opérateurs :

* Wave
* Orange Money
* autres fournisseurs à travers une architecture extensible

Le système utilise une abstraction :

```text
PaymentProvider
       │
       ├── WavePaymentProvider
       │
       ├── OrangeMoneyPaymentProvider
       │
       └── MockPaymentProvider
```

Cette approche permet de développer et tester toute la plateforme localement avant l'intégration des API réelles.

### 🔄 Suivi des transactions

Chaque paiement possède :

* une référence interne ;
* une référence externe ;
* un opérateur ;
* un montant ;
* un statut ;
* une date d'initiation ;
* une date de confirmation ;
* les informations de réponse du fournisseur.

Les statuts peuvent notamment être :

```text
PENDING
PROCESSING
SUCCESS
FAILED
CANCELLED
```

### 🔔 Webhooks

Les fournisseurs de paiement peuvent notifier SEN-PAY lorsqu'une transaction évolue.

Le système prévoit :

```text
Payment Provider
       │
       │ Webhook
       ▼
Webhook Controller
       │
       ▼
Payment Service
       │
       ├── Vérification
       ├── Idempotence
       ├── Mise à jour transaction
       └── Mise à jour bordereau
```

### 🧾 Justificatifs

Après confirmation du paiement :

* génération du reçu ;
* archivage du justificatif ;
* association au bordereau ;
* consultation depuis l'espace entreprise.

---

# 🏗️ Architecture

SEN-PAY adopte une architecture modulaire inspirée de l'**Architecture Hexagonale / Clean Architecture**.

```text
sen-pay
│
├── backend
│   └── src/main/java
│       └── com.senpay
│           │
│           ├── identity
│           │
│           ├── organization
│           │
│           ├── employee
│           │
│           ├── contribution
│           │
│           ├── billing
│           │   ├── domain
│           │   ├── application
│           │   ├── infrastructure
│           │   └── presentation
│           │
│           ├── payment
│           │   ├── domain
│           │   ├── application
│           │   ├── infrastructure
│           │   └── presentation
│           │
│           ├── document
│           │
│           ├── audit
│           │
│           └── shared
│
├── frontend
│   └── Angular
│
├── infrastructure
│   └── docker
│
└── docs
```

---

# 🧩 Architecture du paiement

Le contrôleur REST ne communique pas directement avec Wave ou Orange Money.

```text
Angular
   │
   ▼
PaymentController
   │
   ▼
InitiatePaymentService
   │
   ▼
PaymentProvider
   │
   ├───────────────┐
   ▼               ▼
Wave              Orange Money
Provider          Provider
```

Interface principale :

```java
public interface PaymentProvider {

    PaymentInitiationResult initiate(
        PaymentRequest request
    );

    PaymentStatusResult checkStatus(
        String transactionReference
    );
}
```

Cette abstraction permet d'ajouter ultérieurement d'autres moyens de paiement sans modifier le cœur métier.

---

# 🛠️ Technologies

## Backend

* Java 17
* Spring Boot 3
* Spring Web
* Spring Data JPA
* Hibernate
* Spring Security
* Bean Validation
* Maven
* PostgreSQL
* Flyway
* JUnit 5
* Mockito
* OpenAPI / Swagger

## Frontend

* Angular
* TypeScript
* RxJS
* Angular Material / Bootstrap
* Reactive Forms
* HTTP Client

## Infrastructure

* Docker
* Docker Compose
* PostgreSQL
* Git
* GitHub
* CI/CD

---

# 🔐 Sécurité

La plateforme prévoit plusieurs mécanismes de sécurité :

* authentification ;
* autorisation basée sur les rôles ;
* JWT ;
* validation des données entrantes ;
* protection des endpoints ;
* gestion sécurisée des secrets ;
* contrôle des webhooks ;
* idempotence des transactions ;
* audit des opérations sensibles.

Les clés et secrets des fournisseurs de paiement ne sont jamais stockés dans le repository.

---

# 🔄 Exemple de parcours de paiement

```text
1. L'employeur se connecte
             │
             ▼
2. Consultation du bordereau
             │
             ▼
3. Choix du moyen de paiement
             │
             ▼
4. Création de la transaction
             │
             ▼
5. Appel du PaymentProvider
             │
             ▼
6. Paiement mobile
             │
             ▼
7. Webhook du fournisseur
             │
             ▼
8. Vérification de la transaction
             │
             ▼
9. Transaction SUCCESS
             │
             ▼
10. Bordereau PAYÉ
             │
             ▼
11. Génération du reçu
             │
             ▼
12. Archivage du justificatif
```

---

# 🗄️ Modèle de données simplifié

```text
EMPLOYER
   │
   ├── EMPLOYEES
   │
   └── CONTRIBUTION_STATEMENTS
             │
             ├── PAYMENT_TRANSACTIONS
             │        │
             │        └── PAYMENT_ATTEMPTS
             │
             └── PAYMENT_RECEIPTS
```

Principales entités :

```text
IpmEmployer
Employee
ContributionStatement
PaymentTransaction
PaymentAttempt
PaymentReceipt
```

---

# 🧪 Environnement de développement

Le projet peut fonctionner entièrement en local.

```text
┌───────────────────────┐
│       Angular         │
│      localhost        │
└───────────┬───────────┘
            │ REST
            ▼
┌───────────────────────┐
│     Spring Boot       │
│      localhost        │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│      PostgreSQL       │
│        Docker         │
└───────────────────────┘
```

Les paiements peuvent être simulés avec des **Mock Payment Providers** afin de tester le workflow sans utiliser de véritables transactions financières.

---

# ⚙️ Installation

## Prérequis

```text
Java 17+
Node.js
npm
Angular CLI
Maven
Docker
Docker Compose
Git
```

## Cloner le projet

```bash
git clone https://github.com/malado04/sen-pay.git

cd sen-pay
```

## Démarrer PostgreSQL

```bash
docker compose up -d postgres
```

## Lancer le backend

```bash
cd backend

./mvnw spring-boot:run
```

Sous Windows :

```bash
mvnw.cmd spring-boot:run
```

## Lancer Angular

```bash
cd frontend

npm install

ng serve
```

Application :

```text
http://localhost:4200
```

API :

```text
http://localhost:8080
```

Swagger :

```text
http://localhost:8080/swagger-ui/index.html
```

---

# 🧪 Tests

Le projet utilise :

```text
JUnit 5
Mockito
Spring Boot Test
MockMvc
```

Exemple :

```bash
./mvnw test
```

Les tests couvrent notamment :

* création d'un bordereau ;
* validation d'un paiement ;
* changement de statut ;
* gestion des erreurs ;
* idempotence des webhooks ;
* validation du montant ;
* intégration avec les fournisseurs simulés.

---

# 📌 Roadmap

## Phase 1 — Core

* [x] Initialisation du projet
* [ ] Architecture Spring Boot
* [ ] PostgreSQL
* [ ] Gestion des employeurs
* [ ] Gestion des employés
* [ ] Gestion des cotisations
* [ ] Bordereaux

## Phase 2 — Paiement

* [ ] PaymentProvider
* [ ] Mock Payment Provider
* [ ] Initiation paiement
* [ ] Suivi transaction
* [ ] Webhooks
* [ ] Idempotence
* [ ] Historique des paiements

## Phase 3 — Angular

* [ ] Authentification
* [ ] Dashboard employeur
* [ ] Liste des bordereaux
* [ ] Paiement
* [ ] Historique
* [ ] Reçus

## Phase 4 — Sécurité

* [ ] Spring Security
* [ ] JWT
* [ ] RBAC
* [ ] Audit
* [ ] Gestion des secrets

## Phase 5 — Documents

* [ ] Génération PDF
* [ ] Archivage
* [ ] GED
* [ ] Recherche documentaire

## Phase 6 — Intelligence

* [ ] Indexation documentaire
* [ ] Recherche sémantique
* [ ] RAG
* [ ] Assistant réglementaire
* [ ] Réponses avec sources documentaires

## Phase 7 — Production

* [ ] Dockerisation
* [ ] CI/CD
* [ ] Monitoring
* [ ] Logs
* [ ] Déploiement cloud
* [ ] Intégration des fournisseurs de paiement réels

---

# 🎯 Objectifs techniques

SEN-PAY est également un projet personnel destiné à mettre en pratique des concepts modernes du développement logiciel :

* Architecture hexagonale
* Clean Architecture
* Domain-Driven Design
* SOLID
* REST API
* JWT / RBAC
* Transactions distribuées
* Idempotence
* Event-driven architecture
* Webhooks
* Design Patterns
* Tests automatisés
* Docker
* CI/CD
* Observabilité
* Sécurité applicative

---

# 👨‍💻 Auteur

**Amadou Malado NDIAYE**

Ingénieur Logiciel | Développeur Full Stack | Architecture Logicielle

**Stack principale :**

```text
Java • Spring Boot • Angular • Laravel
PostgreSQL • Docker • Git • Linux
```

📍 Dakar, Sénégal

---

# ⚠️ Disclaimer

SEN-PAY est un projet technique et de démonstration destiné à explorer la conception d'une plateforme de gestion des cotisations et des paiements numériques.

Les intégrations avec des fournisseurs de paiement réels nécessitent l'utilisation de leurs API officielles, leurs environnements de test, leurs exigences de sécurité et leurs conditions d'intégration.

Aucune donnée financière réelle ne doit être utilisée dans l'environnement de démonstration.

---

⭐ **Projet développé pour démontrer des compétences en architecture logicielle, Java/Spring Boot, Angular, FinTech, API REST, sécurité et intégration de systèmes de paiement.**
