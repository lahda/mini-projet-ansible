# Mini Projet Ansible - Infrastructure as Code

Ce projet déploie une infrastructure automatisée utilisant **Ansible** et **Vagrant** pour provisionner des serveurs avec Docker et Apache dans des conteneurs.

## 📋 Table des matières

- [Aperçu du projet](#aperçu-du-projet)
- [Architecture](#architecture)
- [Prérequis](#prérequis)
- [Structure du projet](#structure-du-projet)
- [Installation](#installation)
- [Utilisation](#utilisation)
- [Configuration](#configuration)
- [Rôles Ansible](#rôles-ansible)
- [Déploiement](#déploiement)

---

## 🎯 Aperçu du projet

Ce projet automatise le déploiement d'une infrastructure complète avec :

- **1 serveur Ansible Master** (nœud de contrôle) - IP: `192.168.99.20`
- **2 serveurs clients** (nœuds gérés):
  - **client1** (Production) - IP: `192.168.99.21`
  - **client2** (Staging) - IP: `192.168.99.22`

Chaque client reçoit :
- ✅ Docker et Docker Compose
- ✅ Python 3 et les outils nécessaires
- ✅ Un conteneur Apache HTTP Server
- ✅ Une page d'accueil personnalisée générée via Jinja2

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│            Vagrant (Local Machine)                   │
├─────────────────────────────────────────────────────┤
│                                                       │
│  ┌────────────────────┐     ┌──────────────────┐   │
│  │  Ansible Master    │     │    Clients       │   │
│  │  (192.168.99.20)   │     ├──────────────────┤   │
│  │                    │────▶│ Client1 (Prod)   │   │
│  │  - Ansible Core    │     │ 192.168.99.21    │   │
│  │  - SSH Keys        │     │                  │   │
│  │  - Python 3        │     │ Client2 (Staging)│   │
│  │  - sshpass         │     │ 192.168.99.22    │   │
│  └────────────────────┘     └──────────────────┘   │
│                                                       │
│  Chaque client:                                      │
│  - Docker & Docker Compose                          │
│  - Apache Container (httpd:latest)                  │
│  - Page HTML personnalisée                          │
│                                                       │
└─────────────────────────────────────────────────────┘
```

---

## 📦 Prérequis

### Sur la machine locale :
- **Vagrant** (v2.2.0+)
- **VirtualBox** (ou autre provider compatible)
- **Git** (optionnel, pour cloner le projet)

### Sur les VMs :
- **Ubuntu 22.04 LTS (Jammy)** - définie dans le Vagrantfile
- **2 vCPU et 1 GB RAM** par VM (configurable)

---

## 📁 Structure du projet

```
mini-projet-ansible/
├── Vagrantfile                    # Configuration Vagrant pour les VMs
├── install_ansible.sh             # Script d'installation Ansible
├── README.md                       # Documentation du projet
│
└── mini-projet/                   # Répertoire de travail Ansible
    ├── ansible.cfg               # Configuration Ansible
    ├── deploy-app.yml            # Playbook principal de déploiement
    ├── host.yml                  # Inventaire des hosts
    ├── group_vars/
    │   └── all.yml              # Variables globales pour tous les hosts
    ├── host_vars/
    │   ├── client1.yml          # Variables spécifiques à client1
    │   └── client2.yml          # Variables spécifiques à client2
    └── roles/
        ├── basic-apache-container/          # Rôle personnalisé principal
        │   ├── tasks/
        │   │   ├── main.yml                # Tâches principales
        │   │   ├── setup-ubuntu.yml        # Préparation du système
        │   │   └── install-docker.yml      # Installation Docker
        │   ├── templates/
        │   │   └── index.html.j2           # Template HTML avec Jinja2
        │   ├── defaults/
        │   │   └── main.yml               # Variables par défaut
        │   ├── vars/
        │   │   └── main.yml               # Variables du rôle
        │   └── README.md
        ├── geerlingguy.docker/             # Rôle tiers Docker (non utilisé actuellement)
        │   ├── defaults/
        │   ├── tasks/
        │   ├── vars/
        │   └── molecule/
        └── geerlingguy.pip/                # Rôle tiers Pip (non utilisé actuellement)
            ├── defaults/
            ├── tasks/
            └── molecule/
```

---

## 🚀 Installation

### Étape 1 : Cloner le projet

```bash
git clone https://github.com/lahda/mini-projet-ansible.git
cd mini-projet-ansible
```

### Étape 2 : Démarrer l'infrastructure Vagrant

```powershell
# Windows PowerShell
vagrant up
```

Cela créera et démarrera :
- 1 VM Ansible Master
- 2 VMs Client

Le script `install_ansible.sh` s'exécutera automatiquement sur chaque VM.

### Étape 3 : Se connecter au serveur Ansible Master

```bash
vagrant ssh ansible
```

---

## 💻 Utilisation

### Vérifier l'inventaire Ansible

Une fois connecté au serveur Master (ansible) :

```bash
cd /vagrant/mini-projet
ansible all -i host.yml -m ping
```

### Lancer le déploiement complet

```bash
cd /vagrant/mini-projet
ansible-playbook -i host.yml deploy-app.yml
```

### Déployer sur un environnement spécifique

**Production (client1) :**
```bash
ansible-playbook -i host.yml deploy-app.yml --extra-vars "target_hosts=prod"
```

**Staging (client2) :**
```bash
ansible-playbook -i host.yml deploy-app.yml --extra-vars "target_hosts=staging"
```

### Accéder aux applications déployées

Après le déploiement :
- **Client1 (Production)** : http://192.168.99.21
- **Client2 (Staging)** : http://192.168.99.22

---

## ⚙️ Configuration

### Variables Globales (`group_vars/all.yml`)

```yaml
ansible_user: vagrant                           # Utilisateur SSH
ansible_ssh_common_args: '-o StrictHostKeyChecking=no'  # Options SSH
```

### Variables du Rôle `basic-apache-container`

**Defaults (`defaults/main.yml`) :**
```yaml
webapp_port: 80              # Port exposé du conteneur
apache_port: 80              # Port interne Apache
file_template: index.html.j2 # Template HTML
welcome: "Bienvenue sur la page web gérée par Ansible"
```

**Vars (`vars/main.yml`) :**
```yaml
system_user: vagrant         # Utilisateur propriétaire des fichiers
```

### Inventaire (`host.yml`)

Le fichier inventaire organise les hosts en groupes :

```yaml
all:
  children:
    prod:      # Production (client1)
      hosts:
        client1:
          ansible_host: 192.168.99.21
    staging:   # Staging (client2)
      hosts:
        client2:
          ansible_host: 192.168.99.22
```

---

## 🎭 Rôles Ansible

### 1. `basic-apache-container` (Personnalisé)

**Objectif** : Automatiser l'installation et le déploiement d'Apache dans Docker.

**Tâches :**

1. **setup-ubuntu.yml** - Préparation du système
   - Vérification de la distribution Ubuntu
   - Installation des dépendances (apt-transport-https, curl, git, python3, etc.)

2. **install-docker.yml** - Installation Docker
   - Installation de docker.io et docker-compose
   - Installation du SDK Python pour Docker
   - Démarrage du service Docker
   - Ajout de l'utilisateur vagrant au groupe docker

3. **Génération du fichier index.html**
   - Utilisation du template Jinja2 : `index.html.j2`
   - Personnalisation via la variable `welcome`

4. **Déploiement du conteneur Apache**
   - Image : `httpd:latest`
   - Port mappé : 80
   - Volume : fichier index.html
   - Politique de redémarrage : always

### 2. `geerlingguy.docker` (Rôle tiers)

Rôle de la communauté pour installer Docker. Actuellement non utilisé mais disponible pour utilisation future.

### 3. `geerlingguy.pip` (Rôle tiers)

Rôle de la communauté pour gérer les packages Python. Actuellement non utilisé mais disponible pour utilisation future.

---

## 📝 Configuration Ansible (`ansible.cfg`)

```ini
[defaults]
host_key_checking = False          # Désactiver la vérification des clés SSH
inventory = ./host.yml             # Chemin de l'inventaire
roles_path = ./roles               # Chemin des rôles personnalisés
```

---

## 🔧 Playbook de Déploiement (`deploy-app.yml`)

```yaml
---
- hosts: "{{ target_hosts | default('prod') }}"
  become: true
  roles:
    - role: basic-apache-container
```

- **Hosts** : cible les hosts définis par la variable `target_hosts` (par défaut 'prod')
- **become: true** : exécute les tâches avec les droits sudo
- **Role** : applique le rôle `basic-apache-container`

---

## 📊 Installation Ansible sur Master

Le script `install_ansible.sh` automatise :

### Pour le Master :
```bash
# Installation Ansible Core
sudo apt install ansible-core=2.17.14-1ppa~jammy -y

# Installation de sshpass (pour authentification SSH par mot de passe)
apt install -y sshpass

# Installation optionnelle de Zsh (si ENABLE_ZSH=true)
```

### Pour les Nodes :
```bash
# Installation des dépendances Python
sudo apt install -y python3 python3-pip
```

---

## 🔐 Gestion SSH

- **Authentification** : SSH par clé privée (`/home/vagrant/.ssh/client*_private_key`)
- **Utilisateur** : `vagrant`
- **StrictHostKeyChecking** : désactivé pour automatisation

---

## 🐛 Dépannage

### Problème : Ansible ne peut pas se connecter aux clients

```bash
# Vérifier la connectivité SSH
ssh -i /home/vagrant/.ssh/client1_private_key vagrant@192.168.99.21

# Vérifier le ping Ansible
ansible all -i host.yml -m ping -vvv
```

### Problème : Docker n'est pas accessible

```bash
# Vérifier que l'utilisateur vagrant est dans le groupe docker
groups vagrant

# Si nécessaire, redémarrer la session ou relancer le rôle
```

### Problème : Le conteneur Apache ne démarre pas

```bash
# Vérifier les logs Docker
docker logs webapp

# Vérifier les tâches Ansible
ansible-playbook -i host.yml deploy-app.yml -vvv
```

---

## 📋 Liste de contrôle de déploiement

- [ ] VirtualBox et Vagrant installés
- [ ] `vagrant up` exécuté avec succès
- [ ] Connexion à Ansible Master : `vagrant ssh ansible`
- [ ] Ping Ansible actif : `ansible all -i host.yml -m ping`
- [ ] Playbook lancé : `ansible-playbook -i host.yml deploy-app.yml`
- [ ] Conteneurs vérifié : `docker ps` sur client1 et client2
- [ ] Pages web accessibles : http://192.168.99.21 et http://192.168.99.22

---

## 📄 Licence

GPLv3 - Voir le fichier LICENSE dans le rôle `basic-apache-container`

---

## 👤 Auteur

Projet créé par l'équipe de formation EAZY Training

---

## 🔗 Ressources utiles

- [Documentation Ansible officielle](https://docs.ansible.com/)
- [Documentation Vagrant](https://www.vagrantup.com/docs)
- [Jinja2 Templates](https://jinja.palletsprojects.com/)
- [Docker Documentation](https://docs.docker.com/)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/tips_tricks/index.html)

---

**Dernière mise à jour** : Juin 2026
