# Guide de démarrage rapide

Ce guide vous aidera à démarrer rapidement avec cette infrastructure DevOps.

## Option 1: Installation complète avec GitLab CE local

Cette option installe tous les services, y compris GitLab CE en local.

```bash
# Cloner le dépôt
git clone https://github.com/votre-username/devops-infrastructure-kit.git
cd devops-infrastructure-kit

# Créer la structure de dossiers
mkdir -p data/{gitlab/{config,logs,data},jenkins,portainer,registry,prometheus,grafana}
mkdir -p config/prometheus

# Créer la configuration Prometheus de base
cat > config/prometheus/prometheus.yml << 'EOF'
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
EOF

# Démarrer tous les services
docker-compose up -d
```

## Option 2: Installation sans GitLab CE (utilisation avec GitLab.com)

Cette option installe tous les services excepté GitLab CE, en supposant que vous utiliserez GitLab.com.

```bash
# Cloner le dépôt
git clone https://github.com/votre-username/devops-infrastructure-kit.git
cd devops-infrastructure-kit

# Créer la structure de dossiers
mkdir -p data/{jenkins,portainer,registry,prometheus,grafana}
mkdir -p config/prometheus

# Créer la configuration Prometheus de base
cat > config/prometheus/prometheus.yml << 'EOF'
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
EOF

# Démarrer les services sans GitLab
docker-compose -f docker-compose-gitlab-com.yml up -d
```

## Vérification des services

Une fois que les conteneurs sont opérationnels, vous pouvez vérifier leur état avec:

```bash
docker-compose ps
```

## Configuration initiale des services

### GitLab (Option 1 uniquement)

L'initialisation de GitLab peut prendre quelques minutes. Vous pouvez vérifier l'état avec:

```bash
docker logs -f gitlab
```

Une fois GitLab prêt, accédez à http://localhost:8090 et définissez le mot de passe initial.

### Jenkins

Pour obtenir le mot de passe initial de Jenkins:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Accédez à http://localhost:9080 et suivez les instructions d'installation.

### Portainer

Accédez à http://localhost:9000 et créez un compte administrateur.

## Exemple d'intégration GitLab-Jenkins

Pour des instructions détaillées sur l'intégration de GitLab (local ou GitLab.com) avec Jenkins, consultez le fichier [INTEGRATION.md](INTEGRATION.md).

## Résolution des problèmes courants

### GitLab est lent ou ne répond pas

GitLab nécessite des ressources significatives. Augmentez la mémoire allouée à Docker si nécessaire.

### Erreurs de connexion à un service

Vérifiez que tous les conteneurs sont en cours d'exécution:

```bash
docker-compose ps
```

Si un conteneur n'est pas en cours d'exécution, vérifiez les journaux:

```bash
docker-compose logs [nom-du-service]
```

### Problèmes d'espace disque

Vous pouvez vérifier l'espace utilisé par les conteneurs Docker:

```bash
docker system df
```

Pour nettoyer les ressources inutilisées:

```bash
docker system prune
```