# DevOps Infrastructure Kit

Une solution complète pour mettre en place un environnement DevOps avec les outils essentiels dans des conteneurs Docker, avec support pour Windows.

## 🚀 Composants

- **GitLab.com** : Plateforme complète de gestion de code source avec CI/CD intégré (version cloud)
- **Jenkins** : Serveur d'automatisation pour la construction, les tests et le déploiement
- **Portainer** : Interface de gestion Docker facile à utiliser
- **Registry Docker** : Registre privé pour stocker vos images Docker
- **Prometheus** : Système de surveillance et d'alerte
- **Grafana** : Plateforme de visualisation et d'analyse pour les métriques

## 🔧 Prérequis

- Docker et Docker Compose installés sur votre machine Windows
- WSL2 configuré pour Docker Desktop (recommandé)
- Au moins 4 Go de RAM disponible (8 Go recommandés)
- 20 Go d'espace disque libre
- Un compte GitLab.com (gratuit)

## 🏁 Installation

1. Clonez ce dépôt:
   ```bash
   git clone https://github.com/votre-username/devops-infrastructure-kit.git
   cd devops-infrastructure-kit
   ```

2. Créez les dossiers nécessaires pour la configuration:
   ```bash
   mkdir -p data/{jenkins,portainer,registry,prometheus,grafana}
   mkdir -p config/prometheus
   ```

3. Créez un fichier de configuration pour Prometheus:
   ```bash
   cat > config/prometheus/prometheus.yml << 'EOF'
   global:
     scrape_interval: 15s
     evaluation_interval: 15s

   scrape_configs:
     - job_name: 'prometheus'
       static_configs:
         - targets: ['localhost:9090']
   EOF
   ```

4. Démarrez les services (en utilisant le fichier docker-compose-gitlab-com.yml):
   ```bash
   docker-compose -f docker-compose-gitlab-com.yml up -d
   ```

## 📊 Accès aux services

Une fois les services démarrés, vous pouvez y accéder via les adresses suivantes:

| Service    | URL                      | Identifiants par défaut         |
|------------|--------------------------|--------------------------------|
| GitLab.com | https://gitlab.com       | Votre compte GitLab.com        |
| Jenkins    | http://localhost:9080    | Voir les instructions          |
| Portainer  | http://localhost:9000    | À créer lors du premier accès  |
| Registry   | http://localhost:5000    | N/A                            |
| Prometheus | http://localhost:9090    | N/A                            |
| Grafana    | http://localhost:3000    | `admin` / `admin`              |

### Instructions d'accès initiales

#### GitLab.com
1. Créez un compte ou connectez-vous à https://gitlab.com
2. Créez un groupe et des projets selon vos besoins
3. Générez un jeton d'accès personnel pour l'intégration avec Jenkins

#### Jenkins
1. Accédez à http://localhost:9080
2. Obtenez le mot de passe d'administrateur initial avec:
   ```bash
   docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```
3. Suivez l'assistant d'installation pour configurer Jenkins
4. Installez le plugin GitLab pour l'intégration avec GitLab.com

#### Portainer
1. Accédez à http://localhost:9000
2. Créez un compte administrateur lors de votre première visite

## 🔄 Maintenance

### Mise à jour des services
```bash
docker-compose pull
docker-compose down
docker-compose up -d
```

### Sauvegarde des données
Toutes les données sont stockées dans le dossier `./data`. Pour effectuer une sauvegarde, arrêtez d'abord les services puis copiez ce dossier.

```bash
docker-compose down
cp -r data /path/to/backup/location
docker-compose up -d
```

## ⚠️ Remarques importantes

- Ce déploiement est conçu pour un environnement de développement ou de test. Pour un environnement de production, des configurations supplémentaires seraient nécessaires (sécurité, haute disponibilité, etc.).
- GitLab nécessite une quantité significative de ressources. Si vous rencontrez des problèmes de performance, considérez l'augmentation des ressources allouées à Docker.
- Par défaut, tous les services sont exposés uniquement à localhost. Pour une accessibilité externe, vous devrez modifier les configurations réseau.

## ⚙️ Configuration de l'intégration GitLab.com et Jenkins

Pour une intégration efficace entre GitLab.com et Jenkins, consultez le guide détaillé dans [GITLAB-JENKINS-INTEGRATION.md](GITLAB-JENKINS-INTEGRATION.md).

## 🖥️ Alternative - Installation locale de GitLab CE

Si vous utilisez Linux ou souhaitez quand même déployer GitLab CE localement malgré les limitations sur Windows, consultez le fichier [GITLAB-CE-INSTALLATION.md](GITLAB-CE-INSTALLATION.md).

## 📝 Licence

Ce projet est distribué sous licence MIT. Voir le fichier [LICENSE](LICENSE) pour plus de détails.
