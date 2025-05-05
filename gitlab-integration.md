# Intégration GitLab.com avec Jenkins

Ce guide détaille les étapes pour configurer l'intégration entre GitLab.com et Jenkins.

## Pourquoi utiliser GitLab.com au lieu de GitLab CE local

Sur Windows, l'exécution de GitLab CE dans Docker rencontre plusieurs problèmes :

1. Incompatibilité avec Docker Desktop pour Windows (problèmes de permissions des volumes)
2. Performances réduites et stabilité compromise
3. Configuration complexe pour faire fonctionner l'instance correctement
4. Absence de MTA (Mail Transport Agent) dans l'image Docker GitLab

GitLab.com offre une alternative fiable qui évite ces problèmes tout en fournissant des fonctionnalités similaires.

## Configuration de GitLab.com

### 1. Créer ou utiliser un compte GitLab.com

Inscrivez-vous sur [GitLab.com](https://gitlab.com) si vous n'avez pas déjà un compte.

### 2. Configurer un groupe et des projets

1. Créez un groupe pour votre organisation
2. Créez des projets dans ce groupe selon vos besoins
3. Invitez les membres de votre équipe à rejoindre le groupe

### 3. Créer un jeton d'accès personnel (PAT)

1. Dans GitLab.com, allez dans votre avatar → Préférences
2. Sélectionnez "Jetons d'accès" dans le menu latéral
3. Créez un nouveau jeton avec les autorisations suivantes :
   - `read_repository`
   - `write_repository`
   - `api`
   - `read_api`
   - `read_registry`
   - `write_registry`
4. Notez le jeton généré (vous ne pourrez plus le voir après avoir quitté la page)

## Configuration de Jenkins

### 1. Installer le plugin GitLab

1. Accédez à Jenkins via http://localhost:9080
2. Allez dans "Administrer Jenkins" → "Gérer les plugins"
3. Dans l'onglet "Disponible", recherchez "GitLab"
4. Installez "GitLab Plugin" et redémarrez Jenkins

### 2. Configurer la connexion à GitLab.com

1. Allez dans "Administrer Jenkins" → "Configuration système"
2. Faites défiler jusqu'à la section "GitLab"
3. Cliquez sur "Add GitLab Server"
4. Remplissez les champs suivants :
   - Nom : `GitLab.com`
   - URL : `https://gitlab.com/`
   - Jeton d'API : (Ajoutez votre jeton d'accès comme identifiant de type "GitLab API token")
5. Cliquez sur "Test Connection" pour vérifier
6. Enregistrez la configuration

### 3. Configurer les identifiants Git

1. Allez dans "Administrer Jenkins" → "Manage Credentials"
2. Cliquez sur "Jenkins" sous "Stores scoped to Jenkins"
3. Cliquez sur "Global credentials"
4. Cliquez sur "Add Credentials"
5. Choisissez :
   - Kind: "Username with password" (pour HTTPS) ou "SSH Username with private key" (pour SSH)
   - Scope: "Global"
   - ID: un identifiant unique (par exemple "gitlab-user")
   - Description: "GitLab User Credentials"
   - Username: votre nom d'utilisateur GitLab
   - Password/Private Key: votre mot de passe ou clé privée SSH

## Configuration d'un pipeline CI/CD

### 1. Créer un fichier `.gitlab-ci.yml` dans votre projet

```yaml
stages:
  - build
  - test
  - deploy

variables:
  JENKINS_URL: "http://jenkins:8080"

# Job pour déclencher Jenkins
trigger_jenkins:
  stage: build
  script:
    - curl -X POST "${JENKINS_URL}/job/your-jenkins-job/buildWithParameters?token=YOUR_TOKEN&BRANCH_NAME=${CI_COMMIT_REF_NAME}"
```

### 2. Créer un job Jenkins pour votre projet

1. Dans Jenkins, cliquez sur "New Item"
2. Entrez un nom et sélectionnez "Pipeline" ou "Multibranch Pipeline"
3. Dans la section "Source Code Management", sélectionnez "Git"
4. Entrez l'URL de votre dépôt GitLab : `https://gitlab.com/your-group/your-project.git`
5. Sélectionnez les identifiants que vous avez créés précédemment
6. Dans la section "Build Triggers", cochez "Build when a change is pushed to GitLab"
7. Pour un pipeline basique, ajoutez ce script dans la section "Pipeline Script" :

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                // Vos commandes de build ici
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                // Vos commandes de test ici
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                // Vos commandes de déploiement ici
            }
        }
    }
}
```

### 3. Configurer les Webhooks GitLab

Pour que GitLab.com puisse déclencher automatiquement des builds Jenkins :

1. Exposez votre Jenkins à internet (si dans un environnement local, utilisez un service comme [ngrok](https://ngrok.com))
2. Dans votre projet GitLab, allez dans "Settings" → "Webhooks"
3. Ajoutez un webhook avec :
   - URL : `http://your-jenkins-url/project/your-project-name` ou `http://your-jenkins-url/gitlab-webhook/post`
   - Secret Token : créez un token secret
   - Événements : choisissez les événements qui déclencheront le webhook (par exemple, "Push events")

## Utilisation avec le Registry Docker

Pour configurer GitLab.com pour pousser des images vers votre registry Docker local :

1. Connectez-vous à votre registry locale :
   ```bash
   docker login localhost:5000
   ```

2. Dans votre fichier `.gitlab-ci.yml`, ajoutez une étape pour construire et pousser l'image :
   ```yaml
   build_and_push:
     stage: build
     script:
       - docker build -t localhost:5000/my-project:$CI_COMMIT_SHORT_SHA .
       - docker push localhost:5000/my-project:$CI_COMMIT_SHORT_SHA
   ```

## Dépannage

### Problèmes de connexion Jenkins-GitLab

- Vérifiez que le jeton d'accès a les bonnes permissions
- Assurez-vous que Jenkins est accessible depuis Internet si vous utilisez des webhooks
- Vérifiez les journaux Jenkins pour identifier les erreurs spécifiques

### Problèmes d'authentification Git

- Vérifiez que les identifiants configurés dans Jenkins sont corrects
- Si vous utilisez SSH, assurez-vous que votre clé SSH est correctement ajoutée à votre compte GitLab

### Problèmes avec le Registry Docker

- Assurez-vous que votre registry est accessible (par défaut sur http://localhost:5000)
- Vérifiez que vous avez les droits pour pousser des images à la registry
