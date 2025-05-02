# Utilisation avec GitLab.com

Ce document explique comment utiliser ce projet avec GitLab.com plutôt qu'avec l'installation Docker locale de GitLab CE.

## Pourquoi utiliser GitLab.com?

GitLab.com offre plusieurs avantages par rapport à une installation GitLab locale:

- Aucune maintenance ou mise à jour à gérer
- Pas de consommation de ressources sur votre machine locale
- Accès à toutes les fonctionnalités de GitLab Enterprise (avec les niveaux payants)
- Haute disponibilité et sauvegardes automatiques
- Accès depuis n'importe où, sans configuration réseau complexe

## Configuration

### 1. Créer un compte sur GitLab.com

Si vous n'en avez pas déjà un, créez un compte sur [GitLab.com](https://gitlab.com/users/sign_up).

### 2. Créer un jeton d'accès personnel

Pour permettre à Jenkins d'interagir avec GitLab.com:

1. Connectez-vous à votre compte GitLab.com
2. Cliquez sur votre avatar en haut à droite, puis sélectionnez "Préférences"
3. Dans le menu de gauche, sélectionnez "Jetons d'accès"
4. Créez un nouveau jeton avec les autorisations suivantes:
   - `read_repository`
   - `write_repository`
   - `api`
5. Copiez le jeton généré (vous ne pourrez plus le voir après avoir quitté la page)

### 3. Configurer le docker-compose alternatif

Utilisez le fichier `docker-compose-gitlab-com.yml` au lieu du fichier docker-compose standard:

```bash
docker-compose -f docker-compose-gitlab-com.yml up -d
```

### 4. Configurer l'intégration Jenkins-GitLab

1. Installez le plugin GitLab dans Jenkins:
   - Accédez à http://localhost:9080
   - Allez dans "Administrer Jenkins" > "Gestion des plugins"
   - Dans l'onglet "Disponible", recherchez "GitLab"
   - Installez "GitLab Plugin" et redémarrez Jenkins

2. Configurez la connexion à GitLab.com:
   - Accédez à "Administrer Jenkins" > "Configuration système"
   - Faites défiler jusqu'à la section "GitLab"
   - Cliquez sur "Ajouter GitLab Server"
   - Remplissez avec ces informations:
     - Nom: `GitLab.com`
     - URL: `https://gitlab.com/`
     - Jeton d'API: (ajoutez votre jeton d'accès comme identifiant de type "GitLab API token")
   - Cliquez sur "Tester la connexion" pour vérifier
   - Enregistrez la configuration

### 5. Créer un projet GitLab et le connecter à Jenkins

1. Créez un nouveau projet sur GitLab.com
2. Créez un fichier `.gitlab-ci.yml` dans votre dépôt pour définir les pipelines GitLab CI/CD
3. Dans Jenkins, créez un nouveau job de type "Pipeline" ou "Multibranch Pipeline"
4. Configurez le job pour utiliser votre dépôt GitLab.com
5. Utilisez GitLab Webhooks pour déclencher les builds Jenkins (voir ci-dessous)

### Configuration des Webhooks

Pour que GitLab.com puisse notifier Jenkins des événements (push, merge requests, etc.):

1. Votre instance Jenkins doit être accessible depuis Internet
   - Vous pouvez utiliser un service comme [ngrok](https://ngrok.com/) pour exposer temporairement votre Jenkins local
   - Pour un environnement de production, configurez un proxy inverse approprié
   
2. Créez un webhook dans votre projet GitLab:
   - Accédez à votre projet sur GitLab.com
   - Allez dans "Paramètres" > "Webhooks"
   - URL: `http://your-jenkins-url/project/your-project-name` ou `http://your-jenkins-url/gitlab-webhook/post`
   - Sélectionnez les événements qui déclencheront le webhook
   - Ajoutez le webhook

## Utilisation avec le Registry Docker

Vous pouvez configurer votre projet GitLab.com pour pousser des images vers votre registry Docker local:

1. Assurez-vous que votre registry est accessible (par défaut sur http://localhost:5000)
2. Dans votre fichier `.gitlab-ci.yml`, ajoutez une étape pour construire et pousser l'image:

```yaml
build_and_push:
  stage: build
  script:
    - docker build -t localhost:5000/my-project:$CI_COMMIT_SHORT_SHA .
    - docker push localhost:5000/my-project:$CI_COMMIT_SHORT_SHA
```

## Notes importantes

- Lorsque vous utilisez GitLab.com, votre code est hébergé sur les serveurs de GitLab.com. Assurez-vous que cela est conforme à vos exigences de sécurité et de confidentialité.
- L'accès à certaines fonctionnalités de GitLab.com peut nécessiter un abonnement payant.
- Pour un environnement de production, il est recommandé d'utiliser une URL publique stable pour Jenkins plutôt qu'une solution temporaire comme ngrok.
