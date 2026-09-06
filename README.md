# Smartovate — Pipeline CI/CD Complet avec AWS CodePipeline et ECS Fargate

## 📌 Présentation du projet

Ce projet a été réalisé dans le cadre d'un stage chez **Smartovate Ltd**, entreprise spécialisée dans le conseil et les solutions Cloud.

L'objectif est de concevoir et mettre en œuvre un **pipeline CI/CD complet et automatisé sur Amazon Web Services (AWS)** permettant de déployer une application conteneurisée depuis son dépôt de code jusqu'à son exécution sur **Amazon ECS avec AWS Fargate**.

Le pipeline automatise l'ensemble du cycle de déploiement :

```text
Développeur
    │
    │ git push
    ▼
GitHub
    │
    ▼
AWS CodePipeline
    │
    ├──────────────► Source
    │
    ▼
AWS CodeBuild
    │
    ├── Tests unitaires
    ├── Docker Build
    └── Push de l'image
    │
    ▼
Amazon ECR
    │
    │ nouvelle image Docker
    ▼
Amazon ECS / Fargate
    │
    ▼
Application Load Balancer
    │
    ▼
Application accessible
```

L'infrastructure Cloud est déployée automatiquement ou de manière reproductible grâce à **Terraform (Infrastructure as Code)**.

---

# 🎯 1. Objectifs du projet

Le projet répond aux problématiques suivantes :

* réduire les interventions manuelles lors des déploiements ;
* diminuer les risques d'erreurs humaines ;
* automatiser les tests et la construction des images Docker ;
* centraliser les images Docker dans Amazon ECR ;
* automatiser le déploiement sur Amazon ECS Fargate ;
* assurer la disponibilité de l'application grâce à plusieurs tâches ECS ;
* rendre l'infrastructure reproductible avec Terraform ;
* mettre en place des notifications concernant l'état du pipeline ;
* améliorer la fiabilité et la rapidité du processus de mise en production.

L'objectif final est d'obtenir un processus de déploiement reproductible suivant le principe :

```text
Code → Test → Build → Registry → Deploy → Run
```

---

# 🏗️ 2. Architecture globale

L'architecture cible repose sur les services AWS suivants :

```text
                         ┌─────────────────────┐
                         │      Developer      │
                         └──────────┬──────────┘
                                    │
                              git push
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │       GitHub        │
                         │   Branch: main     │
                         └──────────┬──────────┘
                                    │
                                    ▼
                     ┌─────────────────────────────┐
                     │      AWS CodePipeline       │
                     │                             │
                     │  Source → Build → Deploy    │
                     └──────────────┬──────────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │     AWS CodeBuild   │
                         │                     │
                         │ • Unit Tests       │
                         │ • Docker Build     │
                         │ • Docker Push      │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │     Amazon ECR      │
                         │                     │
                         │ Docker Registry     │
                         └──────────┬──────────┘
                                    │
                                    ▼
                     ┌─────────────────────────────┐
                     │        Amazon ECS           │
                     │                             │
                     │       Fargate Cluster       │
                     │                             │
                     │   ┌────────┐  ┌────────┐   │
                     │   │ Task 1 │  │ Task 2 │   │
                     │   └────┬───┘  └────┬───┘   │
                     └────────┼───────────┼────────┘
                              │           │
                              └─────┬─────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Application Load    │
                         │ Balancer (ALB)      │
                         └──────────┬──────────┘
                                    │
                                    ▼
                              Application
```

---

# ☁️ 3. Technologies utilisées

| Domaine                      | Technologie                     |
| ---------------------------- | ------------------------------- |
| Cloud Provider               | Amazon Web Services (AWS)       |
| CI/CD                        | AWS CodePipeline                |
| Source Code                  | GitHub                          |
| Build & Tests                | AWS CodeBuild                   |
| Conteneurisation             | Docker                          |
| Container Registry           | Amazon ECR                      |
| Container Orchestration      | Amazon ECS                      |
| Compute                      | AWS Fargate                     |
| Load Balancing               | Application Load Balancer (ALB) |
| Infrastructure as Code       | Terraform                       |
| Networking                   | Amazon VPC                      |
| Security                     | AWS IAM / Security Groups       |
| Logs                         | Amazon CloudWatch               |
| Notifications                | Amazon SNS / EventBridge        |
| Gestion de projet            | Jira                            |
| Application de démonstration | Python Flask                    |

---

# 📋 4. Périmètre fonctionnel

## Inclus

Le projet comprend :

* création et configuration du dépôt Git ;
* gestion des branches `main` et `develop` ;
* création du repository Amazon ECR ;
* configuration du scan des images ;
* configuration de la lifecycle policy ECR ;
* automatisation du build avec CodeBuild ;
* exécution des tests unitaires ;
* construction de l'image Docker ;
* publication de l'image dans ECR ;
* déploiement de l'infrastructure AWS avec Terraform ;
* création du VPC et des composants réseau nécessaires ;
* création du cluster ECS ;
* création du service ECS Fargate ;
* création de la Task Definition ;
* création de l'Application Load Balancer ;
* orchestration avec CodePipeline ;
* déploiement automatique de l'application ;
* notifications de succès ou d'échec ;
* documentation et procédures de déploiement.

## Exclus

Les éléments suivants ne font pas partie du périmètre principal :

* développement d'une application métier complexe ;
* architecture réseau avancée ;
* Transit Gateway ;
* tests de charge complexes ;
* SAST/DAST avancés ;
* architecture multi-régions.

Une application Flask simple est utilisée comme **application de démonstration** afin de valider le pipeline.

---

# 🗂️ 5. Organisation du projet

```text
Pipeline-CI-CD-Complet/
│
├── app/
│   ├── app.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── tests/
│       └── test_app.py
│
├── infra/
│   ├── versions.tf
│   ├── variables.tf
│   ├── main.tf
│   ├── network.tf
│   ├── alb.tf
│   ├── ecs.tf
│   └── outputs.tf
│
├── docs/
│   ├── RUNBOOK.md
│   └── captures/
│
├── buildspec.yml
├── .gitignore
└── README.md
```

---

# 🚀 6. Fonctionnement du pipeline CI/CD

Le pipeline suit quatre grandes étapes.

## Étape 1 — Source

Le développeur modifie le code puis effectue :

```bash
git add .
git commit -m "Update application"
git push origin main
```

Le push sur `main` déclenche automatiquement le pipeline AWS CodePipeline.

---

## Étape 2 — Build et Tests

AWS CodeBuild récupère le code source et exécute les différentes étapes définies dans :

```text
buildspec.yml
```

Le processus réalise notamment :

1. installation des dépendances ;
2. exécution des tests unitaires ;
3. authentification auprès d'Amazon ECR ;
4. construction de l'image Docker ;
5. création d'un tag unique basé sur le commit ;
6. push de l'image vers ECR.

Exemple de logique :

```text
Source Code
     │
     ▼
Install dependencies
     │
     ▼
Unit Tests
     │
     ▼
Docker Build
     │
     ▼
Docker Tag
     │
     ▼
Docker Push → ECR
```

---

# 🐳 7. Gestion des images Docker

Les images sont stockées dans **Amazon Elastic Container Registry (ECR)**.

Le repository est configuré pour :

* être privé ;
* effectuer un scan des images ;
* utiliser des tags uniques ;
* conserver un nombre limité d'images grâce à une lifecycle policy.

Le principe de versionnement recommandé est :

```text
<repository>:<commit-sha>
```

Par exemple :

```text
smartovate/demo-api:a82f31c
```

Cela permet d'identifier précisément quelle version du code correspond à chaque image Docker.

---

# ☁️ 8. Infrastructure AWS avec Terraform

L'infrastructure est décrite sous forme de code dans le répertoire :

```text
infra/
```

Terraform permet de créer de manière reproductible les ressources AWS nécessaires.

## Ressources principales

### Réseau

* VPC ;
* subnets ;
* Internet Gateway ;
* route tables ;
* Security Groups.

### Load Balancing

* Application Load Balancer ;
* Target Group ;
* Listener HTTP port 80.

### ECS

* ECS Cluster ;
* Task Definition ;
* ECS Service ;
* tâches Fargate.

### ECR

* repository privé ;
* lifecycle policy ;
* configuration de scan.

---

# 🐳 9. Déploiement ECS Fargate

L'application est exécutée sous forme de conteneur Docker dans **Amazon ECS avec AWS Fargate**.

La Task Definition définit notamment :

* l'image Docker provenant d'ECR ;
* le CPU ;
* la mémoire ;
* le port du conteneur ;
* les variables d'environnement ;
* les logs CloudWatch.

Le service ECS est configuré afin de maintenir plusieurs tâches de l'application en fonctionnement.

Exemple :

```text
ECS Service
     │
     ├── Fargate Task 1
     │       └── Flask Container
     │
     └── Fargate Task 2
             └── Flask Container
```

L'Application Load Balancer distribue les requêtes entre les tâches disponibles.

---

# 🔄 10. Déploiement automatique

Lorsqu'une nouvelle version du code est poussée sur `main` :

```text
git push
   │
   ▼
GitHub
   │
   ▼
CodePipeline
   │
   ▼
CodeBuild
   │
   ├── Tests
   ├── Docker Build
   └── Push ECR
   │
   ▼
ECS Deploy
   │
   ▼
Nouvelle Task Definition
   │
   ▼
Fargate
   │
   ▼
ALB
   │
   ▼
Application mise à jour
```

Le développeur n'a donc pas besoin de réaliser manuellement le build, le push de l'image ou le déploiement ECS.

---

# 📅 11. Organisation en sprints

## Sprint 1 — Infrastructure de base et dépôt

**Période : 1 juillet – 14 juillet 2026**

### US 1.1 — Création du dépôt

Objectif :

> Disposer d'un dépôt permettant de versionner le code de l'application et de l'infrastructure.

Critères :

* dépôt GitHub créé ;
* branche `main` créée ;
* branche `develop` créée ;
* code de démonstration versionné ;
* infrastructure Terraform versionnée.

### US 1.2 — Amazon ECR

Objectif :

> Mettre en place le registre permettant de stocker les images Docker.

Critères :

* repository ECR privé ;
* scan des images ;
* lifecycle policy ;
* conservation des dernières images.

---

# 🔨 12. Sprint 2 — Build et tests

**Période : 15 juillet – 28 juillet 2026**

### US 2.1 — AWS CodeBuild

Objectif :

> Automatiser les tests et la construction de l'image Docker.

Critères :

* projet CodeBuild ;
* fichier `buildspec.yml` ;
* tests unitaires ;
* Docker build réussi.

### US 2.2 — Push vers ECR

Objectif :

> Publier automatiquement l'image Docker générée vers ECR.

Critères :

* authentification ECR ;
* tag unique ;
* push réussi ;
* permissions IAM configurées.

---

# ☁️ 13. Sprint 3 — ECS Fargate et infrastructure cible

**Période : 29 juillet – 11 août 2026**

### US 3.1 — ECS et ALB

Objectif :

> Déployer l'environnement d'exécution avec Terraform.

Critères :

* VPC ;
* ECS Cluster ;
* ALB ;
* Target Group ;
* Listener HTTP ;
* Security Groups.

### US 3.2 — Service ECS Fargate

Objectif :

> Exécuter l'application Docker dans ECS Fargate.

Critères :

* Task Definition ;
* configuration CPU/Mémoire ;
* service ECS ;
* plusieurs tâches ;
* association avec l'ALB ;
* application accessible via le DNS de l'ALB.

---

# 🔄 14. Sprint 4 — CodePipeline et notifications

**Période : 12 août – 25 août 2026**

### US 4.1 — AWS CodePipeline

Objectif :

> Orchestrer automatiquement l'ensemble du processus CI/CD.

Le pipeline contient :

```text
SOURCE → BUILD → DEPLOY
```

Critères :

* pipeline CodePipeline créé ;
* déclenchement sur `main` ;
* CodeBuild intégré ;
* déploiement ECS intégré ;
* exécution de bout en bout réussie.

### US 4.2 — Notifications

Objectif :

> Informer l'équipe de l'état du pipeline.

Composants :

```text
CodePipeline
     │
     ▼
EventBridge
     │
     ▼
SNS
     │
     ▼
Email
```

Les notifications peuvent être envoyées lors :

* d'un succès ;
* d'un échec ;
* d'un changement d'état du pipeline.

---

# 🧪 15. Tests et validation

La validation du projet repose notamment sur les scénarios suivants.

### Test 1 — Modification du code

```text
Modification → git push → Pipeline déclenché
```

### Test 2 — Tests unitaires

```text
CodeBuild → Tests → SUCCESS
```

### Test 3 — Construction Docker

```text
CodeBuild → Docker Build → SUCCESS
```

### Test 4 — Publication ECR

```text
Docker Image → ECR → SUCCESS
```

### Test 5 — Déploiement ECS

```text
ECR → ECS → Fargate → RUNNING
```

### Test 6 — Accessibilité

```text
Internet → ALB → ECS → Flask
```

### Test 7 — Déploiement d'une nouvelle version

```text
Nouveau commit
      ↓
CodePipeline
      ↓
Nouvelle image
      ↓
ECS
      ↓
Nouvelle version accessible
```

---

# ⚠️ 16. Bugs potentiels et solutions

## Bug 1 — ECS Task Unhealthy

### Symptôme

Le déploiement ECS échoue parce que l'ALB considère les tâches comme `Unhealthy`.

### Causes possibles

* mauvais chemin de Health Check ;
* application trop lente au démarrage ;
* mauvais port ;
* Security Group incorrect.

### Vérifications

1. consulter les logs CloudWatch ;
2. vérifier le Target Group ;
3. vérifier le port du conteneur ;
4. vérifier le Health Check ;
5. augmenter le `health_check_grace_period`.

---

## Bug 2 — Échec d'authentification ECR

### Symptôme

```text
AccessDeniedException
```

ou :

```text
Cannot perform an interactive login from a non TTY device
```

### Vérifications

Le rôle IAM de CodeBuild doit disposer des permissions nécessaires à ECR, notamment :

```text
ecr:GetAuthorizationToken
ecr:BatchCheckLayerAvailability
ecr:CompleteLayerUpload
ecr:InitiateLayerUpload
ecr:PutImage
ecr:UploadLayerPart
```

La connexion Docker doit utiliser :

```bash
aws ecr get-login-password \
  --region $AWS_DEFAULT_REGION \
  | docker login \
  --username AWS \
  --password-stdin \
  $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
```

---

## Bug 3 — CodePipeline ne se déclenche pas

### Symptôme

Un commit est effectué sur `main`, mais le pipeline ne démarre pas.

### Vérifications

* vérifier la configuration de la source ;
* vérifier la connexion GitHub ;
* vérifier les permissions IAM ;
* vérifier EventBridge ;
* vérifier les événements CloudTrail ;
* vérifier l'état de la connexion GitHub dans AWS.

---

# 🔐 17. Sécurité

Le projet applique plusieurs bonnes pratiques :

* utilisation d'IAM pour contrôler les permissions ;
* absence d'utilisation du compte root pour les opérations courantes ;
* utilisation de rôles IAM pour les services AWS ;
* repository ECR privé ;
* Security Groups limitant les flux réseau ;
* utilisation de tags Docker uniques ;
* conservation limitée des images ECR ;
* séparation entre le code source et les secrets.

Les credentials AWS et les clés d'API ne doivent **jamais être commités dans Git**.

Exemple :

```text
.env
*.pem
credentials
aws_access_key
aws_secret_access_key
```

doivent être exclus du dépôt lorsque cela est approprié.

---

# 🛠️ 18. Prérequis

Avant de commencer, installer :

* Git ;
* Docker Desktop ;
* AWS CLI ;
* Terraform ;
* un compte AWS ;
* un compte GitHub.

Vérification :

```bash
git --version
docker --version
aws --version
terraform --version
```

Configuration AWS :

```bash
aws configure
```

Puis :

```bash
aws sts get-caller-identity
```

Cette commande permet de vérifier que les credentials AWS sont correctement configurés.

---

# 🚀 19. Déploiement de l'infrastructure

Depuis le répertoire Terraform :

```bash
cd infra
```

Initialiser Terraform :

```bash
terraform init
```

Vérifier le plan :

```bash
terraform plan
```

Déployer :

```bash
terraform apply
```

Confirmer avec :

```text
yes
```

Afficher les outputs :

```bash
terraform output
```

---

# 🐳 20. Test local de l'application

Depuis le dossier `app` :

```bash
cd app
```

Construire l'image :

```bash
docker build -t demo-api .
```

Lancer le conteneur :

```bash
docker run -p 5000:5000 demo-api
```

L'application peut ensuite être testée localement via :

```text
http://localhost:5000
```

---

# 📊 21. Résultat attendu

À la fin du projet, le résultat attendu est une chaîne CI/CD entièrement automatisée :

```text
                    ┌──────────────┐
                    │    GitHub    │
                    └──────┬───────┘
                           │
                       git push
                           │
                           ▼
                 ┌──────────────────┐
                 │  CodePipeline    │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │    CodeBuild     │
                 │                  │
                 │ Tests + Docker   │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │       ECR        │
                 │  Docker Image    │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │   ECS Fargate    │
                 │                  │
                 │ Task 1 + Task 2  │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │       ALB        │
                 └────────┬─────────┘
                          │
                          ▼
                    Application
```

Le pipeline permet ainsi de passer automatiquement du **commit développeur à une nouvelle version déployée sur AWS**.

---

# 📚 22. Documentation

La documentation complémentaire est disponible dans :

```text
docs/
```

Notamment :

```text
docs/RUNBOOK.md
```

Le RUNBOOK contient les procédures détaillées de configuration, de déploiement, de vérification et les captures d'écran nécessaires pour documenter le projet.

---

# 📌 23. Livrables

Les principaux livrables du projet sont :

* [x] Code source de l'application Flask ;
* [x] Dockerfile ;
* [x] Tests unitaires ;
* [x] `buildspec.yml` ;
* [x] Infrastructure Terraform ;
* [x] Repository ECR ;
* [x] Cluster ECS Fargate ;
* [x] Application Load Balancer ;
* [x] AWS CodeBuild ;
* [x] AWS CodePipeline ;
* [x] Notifications du pipeline ;
* [x] Documentation technique ;
* [x] Procédure de déploiement ;
* [x] Démonstration du pipeline.

---

# 👩‍💻 Auteur

**Siham Bouzagrar**

Projet réalisé dans le cadre d'un stage chez **Smartovate Ltd**.

### Sujet du stage

**Conception et mise en œuvre d'un pipeline CI/CD complet avec AWS CodePipeline et ECS Fargate**

---

# 📄 Licence

Projet réalisé par Siham Bouzagrar dans le cadre de son stage chez Smartovate Ltd.
