# Ansible Web Cluster — Projet de démonstration

Déploiement automatisé d'un mini-cluster de 3 serveurs web (nginx) avec
Ansible : inventaire multi-hosts, rôles réutilisables, gestion des secrets
avec Ansible Vault, et environnement de test reproductible via Docker.

## Architecture

```
                    ┌─────────────────┐
                    │   Contrôleur     │
                    │   Ansible (WSL)  │
                    └────────┬─────────┘
                             │ SSH
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                     ▼
  ┌──────────┐         ┌──────────┐          ┌──────────┐
  │  web1    │         │  web2    │          │  web3    │
  │  nginx   │         │  nginx   │          │  nginx   │
  └──────────┘         └──────────┘          └──────────┘
   (conteneurs Docker simulant 3 serveurs Linux distincts)
```

## Compétences démontrées

- Écriture de playbooks et **rôles Ansible** réutilisables (`common`, `webserver`)
- Gestion d'un **inventaire multi-hosts** avec variables de groupe
- Utilisation de **templates Jinja2** pour générer des fichiers de config dynamiques
- Gestion des secrets avec **Ansible Vault**
- Idempotence : le playbook peut être rejoué sans effet de bord
- Environnement de test reproductible avec **Docker / docker-compose**

## Structure du projet

```
.
├── docker/
│   ├── Dockerfile              # Image des serveurs cibles (Ubuntu + SSH)
│   └── docker-compose.yml      # 3 conteneurs simulant 3 serveurs
├── inventory/
│   └── hosts.ini                # Inventaire des 3 machines
├── group_vars/
│   ├── all.yml                  # Variables globales
│   └── vault.yml                # Secrets (chiffré avec ansible-vault)
├── roles/
│   ├── common/                  # Mise à jour système, MOTD
│   └── webserver/                # Installation et config nginx
├── site.yml                     # Playbook principal
└── README.md
```

## Prérequis

- Docker Desktop
- WSL2 avec Ansible installé (`sudo apt install ansible`)

## Utilisation

### 1. Lancer les serveurs cibles

```bash
cd docker
docker compose up -d --build
```

### 2. Chiffrer les secrets (une seule fois)

```bash
ansible-vault encrypt group_vars/vault.yml
```

### 3. Vérifier la connexion

```bash
ansible -i inventory/hosts.ini webservers -m ping --ask-vault-pass
```

### 4. Déployer

```bash
ansible-playbook -i inventory/hosts.ini site.yml --ask-vault-pass
```

### 5. Vérifier le résultat

Ouvrir dans un navigateur :
- http://localhost:2221 → ⚠️ voir note ci-dessous (port SSH, pas HTTP)

> Note : dans cette démo, seul le port SSH est exposé. Pour visualiser
> les pages nginx depuis Windows, ajouter un mapping de port HTTP dans
> `docker-compose.yml` (ex: `"8081:80"`) — bon exercice pour aller plus loin !

## Pour aller plus loin

- Ajouter un rôle `firewall` (ufw)
- Ajouter un rôle `monitoring` (node_exporter + Prometheus)
- Passer d'un inventaire statique à un inventaire dynamique Docker
- Intégrer le playbook dans une pipeline CI/CD (GitHub Actions)

## Auteur

Cécile — projet réalisé dans le cadre d'une formation pratique à Ansible.
