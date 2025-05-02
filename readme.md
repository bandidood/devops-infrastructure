# DevOps Infrastructure Kit

Une solution complète pour mettre en place un environnement DevOps avec les outils essentiels dans des conteneurs Docker.

## 🚀 Composants

- **GitLab CE** : Plateforme complète de gestion de code source avec CI/CD intégré
- **Jenkins** : Serveur d'automatisation pour la construction, les tests et le déploiement
- **Portainer** : Interface de gestion Docker facile à utiliser
- **Registry Docker** : Registre privé pour stocker vos images Docker
- **Prometheus** : Système de surveillance et d'alerte
- **Grafana** : Plateforme de visualisation et d'analyse pour les métriques

## 🔧 Prérequis

- Docker et Docker Compose installés sur votre machine
- Au moins 4 Go de RAM disponible (8 Go recommandés)
- 20 Go d'espace disque libre

## 🏁 Installation

1. Clonez ce dépôt:
   ```bash
   git clone https://github.com/bandidood/devops-infrastructure.git
   cd devops-infrastructure
   ```

2. Créez les dossiers nécessaires pour la configuration:
   ```bash
   mkdir -p data/{gitlab/{config,logs,data},jenkins,portainer,registry,prometheus,grafana}
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

4. Démarrez les services:
   ```bash
   docker-compose up -d
   ```

## 📊 Accès aux services

Une fois les services démarrés, vous pouvez y accéder via les adresses suivantes:

| Service    | URL                      | Identifiants par défaut         |
|------------|--------------------------|--------------------------------|
| GitLab     | http://localhost:8090    | `root` / Voir les instructions |
| Jenkins    | http://localhost:9080    | Voir les instructions          |
| Portainer  | http://localhost:9000    | À créer lors du premier accès  |
| Registry   | http://localhost:5000    | N/A                            |
| Prometheus | http://localhost:9090    | N/A                            |
| Grafana    | http://localhost:3000    | `admin` / `admin`              |

### Instructions d'accès initiales

#### GitLab
1. Accédez à http://localhost:8090
2. Lors de votre première visite, vous serez invité à définir un mot de passe pour l'utilisateur `root`
3. Connectez-vous avec le nom d'utilisateur `root` et le mot de passe que vous avez défini

#### Jenkins
1. Accédez à http://localhost:9080
2. Obtenez le mot de passe d'administrateur initial avec:
   ```bash
   docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```
3. Suivez l'assistant d'installation pour configurer Jenkins

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

## 🔄 Variante avec GitLab.com

Si vous préférez utiliser GitLab.com au lieu de l'installation Docker locale, consultez le fichier [docker-compose-gitlab-com.yml](docker-compose-gitlab-com.yml) et les instructions dans [GITLAB-COM.md](GITLAB-COM.md).

## 📝 Licence

Ce projet est distribué sous licence MIT. Voir le fichier [LICENSE](LICENSE) pour plus de détails.
