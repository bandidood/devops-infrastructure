# Installation de GitLab CE en local

Ce document explique comment installer GitLab CE en local, malgré les limitations sur Windows.

## Avertissement pour Windows

GitLab Docker sur Windows présente plusieurs problèmes connus :

- L'image GitLab n'est pas officiellement supportée avec Docker Desktop pour Windows
- Des problèmes de permissions avec les volumes montés
- Besoin d'un Mail Transport Agent (MTA) que l'image ne contient pas
- Besoin d'un nom d'hôte valide et accessible de l'extérieur (pas `localhost`)

## Options d'installation alternatives

### Option 1 : Utiliser WSL2 pour héberger GitLab

WSL2 (Windows Subsystem for Linux) offre un environnement Linux complet qui peut exécuter GitLab plus efficacement que Docker pour Windows.

1. Installez WSL2 sur votre système Windows

   ```powershell
   wsl --install
   ```

2. Installez une distribution Ubuntu

   ```powershell
   wsl --install -d Ubuntu-20.04
   ```

3. Dans WSL2, créez les dossiers pour les données GitLab

   ```bash
   mkdir -p ~/gitlab/{config,logs,data}
   ```

4. Créez un fichier docker-compose.yml dans votre répertoire WSL

   ```bash
   cd ~
   mkdir gitlab-docker
   cd gitlab-docker
   nano docker-compose.yml
   ```

5. Ajoutez la configuration suivante au fichier

   ```yaml
   version: '3.8'
   
   services:
     gitlab:
       image: gitlab/gitlab-ce:latest
       container_name: gitlab
       hostname: gitlab.local
       ports:
         - "8090:80"
         - "8443:443" 
         - "2222:22"
       volumes:
         - ~/gitlab/config:/etc/gitlab
         - ~/gitlab/logs:/var/log/gitlab
         - ~/gitlab/data:/var/opt/gitlab
       environment:
         GITLAB_OMNIBUS_CONFIG: |
           external_url 'http://gitlab.local'
           gitlab_rails['gitlab_shell_ssh_port'] = 2222;
           postgresql['shared_buffers'] = "256MB";
       restart: always
       shm_size: '512m'
   ```

6. Lancez GitLab

   ```bash
   docker-compose up -d
   ```

7. Ajoutez une entrée au fichier hosts Windows (C:\Windows\System32\drivers\etc\hosts)

   ```
   127.0.0.1 gitlab.local
   ```

### Option 2 : Utiliser une machine virtuelle Linux

1. Installez VirtualBox ou Hyper-V
2. Créez une machine virtuelle avec Ubuntu Server
3. Installez Docker et Docker Compose sur la VM
4. Transférez le fichier docker-compose.yml original vers la VM
5. Exécutez le conteneur GitLab dans la VM

### Option 3 : Utiliser Gitea comme alternative légère à GitLab

[Gitea](https://gitea.io/) est une alternative légère à GitLab qui fonctionne bien sur Windows avec Docker.

1. Créez un fichier `docker-compose-gitea.yml`

   ```yaml
   version: '3'
   
   services:
     gitea:
       image: gitea/gitea:latest
       container_name: gitea
       environment:
         - USER_UID=1000
         - USER_GID=1000
       restart: always
       volumes:
         - ./data/gitea:/data
         - /etc/timezone:/etc/timezone:ro
         - /etc/localtime:/etc/localtime:ro
       ports:
         - "3000:3000"
         - "2222:22"
     
     jenkins:
       image: jenkins/jenkins:lts
       container_name: jenkins
       ports:
         - "9080:8080"
         - "50000:50000"
       volumes:
         - ./data/jenkins:/var/jenkins_home
       restart: always
   
     portainer:
       image: portainer/portainer-ce:latest
       container_name: portainer
       ports:
         - "9000:9000"
       volumes:
         - /var/run/docker.sock:/var/run/docker.sock
         - ./data/portainer:/data
       restart: always
   
     registry:
       image: registry:2
       container_name: registry
       ports:
         - "5000:5000"
       volumes:
         - ./data/registry:/var/lib/registry
       restart: always
   
     prometheus:
       image: prom/prometheus:latest
       container_name: prometheus
       ports:
         - "9090:9090"
       volumes:
         - ./config/prometheus:/etc/prometheus
         - ./data/prometheus:/prometheus
       command:
         - '--config.file=/etc/prometheus/prometheus.yml'
       restart: always
   
     grafana:
       image: grafana/grafana:latest
       container_name: grafana
       ports:
         - "3001:3000"
       volumes:
         - ./data/grafana:/var/lib/grafana
       restart: always
   ```

2. Démarrez les services

   ```bash
   docker-compose -f docker-compose-gitea.yml up -d
   ```

3. Accédez à Gitea via http://localhost:3000 et suivez l'assistant d'installation

## Configuration de GitLab pour les problèmes connus

Si vous insistez pour utiliser GitLab CE avec Docker sur Windows, voici quelques solutions aux problèmes courants :

### Problème de permissions des volumes

Pour résoudre les problèmes de permissions avec les volumes montés :

1. Ne montez pas le volume `/var/opt/gitlab` 
   
   ```yaml
   volumes:
     - ./data/gitlab/config:/etc/gitlab
     - ./data/gitlab/logs:/var/log/gitlab
     # Ne pas monter /var/opt/gitlab
   ```

2. Ou utilisez des volumes nommés Docker au lieu des montages de répertoires

   ```yaml
   volumes:
     - gitlab-config:/etc/gitlab
     - gitlab-logs:/var/log/gitlab
     - gitlab-data:/var/opt/gitlab

   volumes:
     gitlab-config:
     gitlab-logs:
     gitlab-data:
   ```

### Configurer un MTA pour les notifications par email

Pour ajouter un MTA (Mail Transfer Agent) :

1. Ajoutez un service Postfix dans votre docker-compose.yml

   ```yaml
   postfix:
     image: catatnight/postfix
     container_name: postfix
     environment:
       - maildomain=gitlab.local
       - smtp_user=gitlab:password
     restart: always
   ```

2. Configurez GitLab pour utiliser ce service Postfix

   ```yaml
   environment:
     GITLAB_OMNIBUS_CONFIG: |
       external_url 'http://gitlab.local'
       gitlab_rails['smtp_enable'] = true
       gitlab_rails['smtp_address'] = "postfix"
       gitlab_rails['smtp_port'] = 25
       gitlab_rails['smtp_domain'] = "gitlab.local"
   ```

### Définir un hostname valide

1. N'utilisez pas `localhost` comme hostname
2. Définissez un nom personnalisé dans votre fichier hosts Windows
3. Assurez-vous que le `external_url` correspond au hostname
