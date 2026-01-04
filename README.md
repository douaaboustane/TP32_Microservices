# Pipeline CI/CD Jenkins - Microservices

Ce projet présente la configuration d'un pipeline CI/CD complet utilisant Jenkins, SonarQube et Docker pour déployer une architecture de microservices.

## Description

L'application est composée de 4 microservices :
- **car** : Service de gestion des véhicules
- **client** : Service de gestion des clients
- **gateway** : Service Gateway (API Gateway)
- **server_eureka** : Serveur Eureka pour la découverte de services

Le pipeline CI/CD assure :
- Compilation Maven des microservices
- Analyse de code avec SonarQube (qualité, code smells, bugs, vulnérabilités)
- Déploiement automatique via Docker Compose
- Déclenchement automatique par webhook GitHub (via Ngrok)

## Structure du Projet

```
jenkins2/
├── car/                  # Microservice Car
├── client/               # Microservice Client
├── gateway/              # Microservice Gateway
├── server_eureka/        # Serveur Eureka
├── deploy/               # Configuration Docker Compose pour le déploiement
└── Jenkinsfile           # Script de pipeline Jenkins
```

## Prérequis

- JDK 17 (ou version compatible) avec variable JAVA_HOME configurée
- Maven (installé localement ou géré par Jenkins)
- Git (ligne de commande)
- Docker et Docker Compose
- Jenkins (installation locale)
- SonarQube (déployé via Docker Compose)
- Ngrok (compte et authtoken)
- Compte GitHub avec accès au dépôt

## Installation et Configuration

### 1. Récupération du Projet

Le dépôt a été cloné depuis GitHub :
```
git clone https://github.com/lachgar/jenkins2.git
cd jenkins2
```

### 2. Configuration de Jenkins

#### Installation de Jenkins
1. Télécharger et installer Jenkins depuis jenkins.io
2. Pendant l'installation, sélectionner LocalSystem comme type de service
3. Choisir un port (par défaut 8080)
4. Indiquer le chemin vers JDK 17

#### Configuration des Outils dans Jenkins
1. Manage Jenkins > Tools
2. Ajouter Maven :
   - Nom : `maven`
   - Installer automatiquement ou indiquer le chemin local
3. Ajouter SonarQube Scanner :
   - Installer automatiquement

### 3. Déploiement de SonarQube

Démarrer SonarQube avec Docker Compose :
```
docker compose -f sonarqube-compose.yml up -d
```

Accéder à SonarQube : http://localhost:9999
- Login par défaut : admin / admin (à changer au premier login)

#### Création des Projets SonarQube
1. Dans SonarQube, créer deux projets :
   - Projet Key : `car`, Name : `car`
   - Projet Key : `client`, Name : `client`
2. Générer un token pour chaque projet :
   - My Account > Security > Generate Token
   - Créer un token pour car et un pour client

#### Configuration SonarQube dans Jenkins
1. Manage Jenkins > System
2. Section SonarQube servers, ajouter deux serveurs :
   - Name : `SonarQube-Car`, URL : `http://localhost:9999`, Token : token du projet car
   - Name : `SonarQube-Client`, URL : `http://localhost:9999`, Token : token du projet client

### 4. Configuration de Ngrok

#### Installation et Configuration
1. Télécharger Ngrok depuis ngrok.com
2. Créer un compte et récupérer l'authtoken
3. Configurer l'authtoken :
```
ngrok config add-authtoken <votre-token>
```

#### Création du Tunnel
Démarrer un tunnel vers Jenkins :
```
ngrok http http://localhost:8080
```

Noter l'URL publique affichée par Ngrok (format : https://xxxx-xx-xx-xx-xx.ngrok-free.app)

### 5. Configuration du Webhook GitHub

1. Aller sur le dépôt GitHub : https://github.com/lachgar/jenkins2
2. Settings > Webhooks > Add webhook
3. Configuration :
   - Payload URL : `https://<URL_NGROK>/github-webhook/`
   - Content type : `application/json`
   - Events : Just the push event
   - Activer le webhook

### 6. Création du Job Pipeline Jenkins

1. Jenkins Dashboard > New Item
2. Nom : `cicd-microservices`
3. Type : Pipeline
4. Configuration :
   - Cocher "GitHub project", URL : `https://github.com/lachgar/jenkins2`
   - Build Triggers : Cocher "GitHub hook trigger for GITScm polling"
   - Pipeline : Pipeline script, copier le contenu du fichier Jenkinsfile
5. Save

## Exécution du Pipeline

### Lancement Manuel
1. Ouvrir le job `cicd-microservices` dans Jenkins
2. Cliquer sur "Build Now"
3. Suivre l'exécution dans Console Output

### Déclenchement Automatique
Le pipeline se déclenche automatiquement lors d'un push sur GitHub :
```
git add .
git commit -m "message"
git push
```

## Étapes du Pipeline

1. **Checkout Repository** : Récupération du code source depuis GitHub
2. **Compilation et Analyse de Code** (en parallèle) :
   - Build et analyse SonarQube pour le service Car
   - Build et analyse SonarQube pour le service Client
   - Build du service Gateway
   - Build du serveur Eureka
3. **Déploiement Docker** : Déploiement des services avec Docker Compose

## Vérification

### Vérifier les Conteneurs Docker
```
docker ps
```

### Vérifier les Analyses SonarQube
Accéder à http://localhost:9999 et vérifier les projets car et client

### Vérifier le Webhook GitHub
Dans GitHub > Settings > Webhooks > Recent Deliveries, vérifier les codes 200 après un push

## Dépannage

**Jenkins ne se lance pas**
- Vérifier que le port n'est pas occupé
- Changer le port si nécessaire dans la configuration Jenkins

**SonarQube inaccessible**
- Vérifier que les conteneurs sont démarrés : `docker ps`
- Vérifier les logs : `docker compose -f sonarqube-compose.yml logs`

**Analyse SonarQube échoue**
- Vérifier que les noms dans Jenkins System correspondent exactement au pipeline (SonarQube-Car, SonarQube-Client)
- Vérifier que les tokens sont valides

**Webhook GitHub échoue**
- Vérifier que l'URL Ngrok est à jour (l'URL change à chaque redémarrage de Ngrok)
- Vérifier le format de l'URL : `https://xxxx.ngrok-free.app/github-webhook/`
- Vérifier dans GitHub > Webhooks > Recent Deliveries

**Docker Compose échoue dans Jenkins**
- Vérifier que Jenkins a accès au daemon Docker
- Vérifier que Docker Desktop est démarré (Windows)

**Build Maven échoue**
- Vérifier que JAVA_HOME est configuré
- Vérifier que Maven est correctement configuré dans Jenkins Tools (nom : `maven`)

## Fichiers Importants

- `Jenkinsfile` : Script de pipeline Jenkins
- `sonarqube-compose.yml` : Configuration Docker Compose pour SonarQube
- `deploy/docker-compose.yml` : Configuration Docker Compose pour les microservices

## Références

- Dépôt GitHub : https://github.com/lachgar/jenkins2
- Documentation Jenkins : https://www.jenkins.io/doc/
- Documentation SonarQube : https://docs.sonarqube.org/
- Documentation Docker : https://docs.docker.com/

