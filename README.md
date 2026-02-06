# CI/CD avec Jenkins et Docker 

## 📋 Description

Ce TP a pour objectif d'introduire les concepts d'**Intégration Continue (CI)** et de **Déploiement Continu (CD)** en utilisant **Jenkins** comme serveur d'automatisation et **Docker** pour la conteneurisation. L'application utilisée est un système **Point of Sale (POS)** développé avec **Spring Boot**.

---

## 📁 Structure du Projet

```
TP30/
└── jenkins/
    └── POV-JAVA/
        ├── src/
        │   ├── main/
        │   │   ├── java/
        │   │   │   └── com/example/Point/of/sale/
        │   │   │       ├── PointOfSaleApplication.java
        │   │   │       └── controller/
        │   │   │           └── HelloController.java
        │   │   └── resources/
        │   │       └── application.properties
        │   └── test/
        ├── Dockerfile
        ├── pom.xml
        ├── mvnw
        └── mvnw.cmd
```

---

## 🛠️ Technologies Utilisées

| Technologie | Version | Description |
|------------|---------|-------------|
| **Java** | 17 | Langage de programmation |
| **Spring Boot** | 3.2.1 | Framework backend |
| **Maven** | - | Gestionnaire de dépendances |
| **Docker** | - | Conteneurisation |
| **Jenkins** | - | Serveur CI/CD |
| **PostgreSQL** | - | Base de données (driver inclus) |
| **Lombok** | - | Réduction du boilerplate code |

---

## 🔌 Endpoints API

| Méthode | Endpoint | Description |
|---------|----------|-------------|
| `GET` | `/` | Message de bienvenue |
| `GET` | `/user` | Liste des utilisateurs |
| `GET` | `/presentation` | Page de présentation |

---

## ⚙️ Configuration

### Application (application.properties)
```properties
server.port=8282
```

L'application écoute sur le **port 8282**.

---

## 🐳 Docker

### Dockerfile
```dockerfile
FROM eclipse-temurin:17-jre
WORKDIR /App
COPY target/*.jar app.jar
EXPOSE 8282
ENTRYPOINT ["java","-jar","app.jar"]
```

### Commandes Docker

```bash
# Construire l'image Docker
docker build -t point-of-sale:latest .

# Lancer le conteneur
docker run -d -p 8282:8282 --name pos-app point-of-sale:latest

# Vérifier les logs
docker logs pos-app

# Arrêter le conteneur
docker stop pos-app

# Supprimer le conteneur
docker rm pos-app
```

---

## 🏃 Exécution Locale

### Prérequis
- **Java 17** installé
- **Maven** installé (ou utiliser le wrapper `mvnw`)

### Étapes

1. **Naviguer vers le répertoire du projet :**
   ```bash
   cd TP30/jenkins/POV-JAVA
   ```

2. **Compiler le projet :**
   ```bash
   mvn clean install
   ```
   ou avec le wrapper :
   ```bash
   ./mvnw clean install
   ```

3. **Lancer l'application :**
   ```bash
   mvn spring-boot:run
   ```

4. **Tester l'application :**
   ```bash
   curl http://localhost:8282/
   # Réponse: Hello from New Test :)
   ```

---

## 🔄 Pipeline Jenkins (CI/CD)

### Exemple de Jenkinsfile

```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9'
        jdk 'JDK 17'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/votre-repo/POV-JAVA.git'
            }
        }
        
        stage('Build') {
            steps {
                dir('POV-JAVA') {
                    sh 'mvn clean compile'
                }
            }
        }
        
        stage('Test') {
            steps {
                dir('POV-JAVA') {
                    sh 'mvn test'
                }
            }
        }
        
        stage('Package') {
            steps {
                dir('POV-JAVA') {
                    sh 'mvn package -DskipTests'
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                dir('POV-JAVA') {
                    sh 'docker build -t point-of-sale:${BUILD_NUMBER} .'
                }
            }
        }
        
        stage('Docker Deploy') {
            steps {
                sh '''
                    docker stop pos-app || true
                    docker rm pos-app || true
                    docker run -d -p 8282:8282 --name pos-app point-of-sale:${BUILD_NUMBER}
                '''
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline exécuté avec succès!'
        }
        failure {
            echo '❌ Le pipeline a échoué!'
        }
    }
}
```

---

## 📊 Diagramme du Pipeline CI/CD

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Git Push  │───▶│   Jenkins   │───▶│   Build     │
└─────────────┘    │   Webhook   │    │   Maven     │
                   └─────────────┘    └──────┬──────┘
                                              │
                                              ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Deploy    │◀───│   Docker    │◀───│   Tests     │
│   Container │    │   Build     │    │   JUnit     │
└─────────────┘    └─────────────┘    └─────────────┘
```

---

## 📚 Concepts Clés du TP

### 1. **Intégration Continue (CI)**
- Compilation automatique du code à chaque commit
- Exécution des tests unitaires
- Génération des artifacts (JAR)

### 2. **Déploiement Continu (CD)**
- Construction automatique des images Docker
- Déploiement automatique des conteneurs
- Rollback en cas d'échec

### 3. **Conteneurisation avec Docker**
- Isolation de l'application
- Portabilité entre environnements
- Facilité de déploiement

---

## 🧪 Tests

Pour exécuter les tests :
```bash
mvn test
```

---

## 📝 Notes Importantes

- L'application **exclut** la configuration automatique de DataSource (`DataSourceAutoConfiguration.class`) car elle ne nécessite pas de connexion base de données pour les endpoints actuels.
- Le port **8282** est utilisé pour éviter les conflits avec d'autres services.
- Le Dockerfile utilise **eclipse-temurin:17-jre** comme image de base pour une empreinte mémoire réduite.

---

## 👨‍💻 Auteur

- **Nom** : Achraf
---

## 📄 Licence

Ce projet est réalisé dans le cadre d'un travail pratique universitaire.
