# Ansible Web Cluster - Projet de démonstration

Déploiement automatisé d'un mini-cluster de 3 serveurs web (nginx) avec
Ansible : inventaire multi-hosts, rôles réutilisables, gestion des secrets
avec Ansible Vault, authentification SSH par clé, et environnement de test
reproductible via Docker.

## Architecture

```
                    ┌─────────────────┐
                    │   Contrôleur     │
                    │   Ansible (WSL)  │
                    └────────┬─────────┘
                             │ SSH (clé privée)
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                     ▼
  ┌──────────┐         ┌──────────┐          ┌──────────┐
  │  web1    │         │  web2    │          │  web3    │
  │  nginx   │         │  nginx   │          │  nginx   │
  └──────────┘         └──────────┘          └──────────┘
   (conteneurs Docker, ports liés à 127.0.0.1 uniquement)
```

## Compétences démontrées

- Écriture de playbooks et **rôles Ansible** réutilisables (`common`, `webserver`)
- Gestion d'un **inventaire multi-hosts** avec variables de groupe
- Utilisation de **templates Jinja2** pour générer des fichiers de config dynamiques
- Gestion des secrets avec **Ansible Vault** (token applicatif + mot de passe sudo)
- **Authentification SSH par clé**, mot de passe désactivé côté SSH
- Idempotence : le playbook peut être rejoué sans effet de bord
- Environnement de test reproductible avec **Docker / docker-compose**
- Bonnes pratiques de sécurité : secrets jamais en clair dans le repo, services non exposés hors localhost

## Structure du projet

```
.
├── docker/
│   ├── Dockerfile              # Image des serveurs cibles (Ubuntu + SSH, auth par clé)
│   ├── docker-compose.yml      # 3 conteneurs simulant 3 serveurs
│   ├── ansible_lab_key.pub     # Clé publique (committée, sans risque)
│   ├── ansible_lab_key         # Clé privée (générée localement, JAMAIS committée)
│   └── .env                    # Mot de passe sudo du build (généré localement, JAMAIS committé)
├── inventory/
│   └── hosts.ini                # Inventaire des 3 machines, connexion par clé
├── group_vars/
│   └── all/
│       ├── vars.yml             # Variables globales
│       └── vault.yml            # Secrets chiffrés (ansible-vault)
├── roles/
│   ├── common/                  # Mise à jour système, MOTD (utilise un secret du vault)
│   └── webserver/                # Installation et config nginx
├── site.yml                     # Playbook principal
└── README.md
```

## Prérequis

- Docker Desktop (avec intégration WSL2 activée si sous Windows)
- WSL2 avec Ansible installé (`sudo apt install ansible`)
- OpenSSL (pour générer un mot de passe fort) et OpenSSH client (`ssh-keygen`)

## Utilisation

### 1. Générer une clé SSH dédiée au lab

```bash
cd docker
ssh-keygen -t ed25519 -f ./ansible_lab_key -N "" -C "ansible-cv-lab"
```

### 2. Créer le fichier `.env` avec un mot de passe fort (sudo uniquement, jamais utilisé en SSH)

```bash
echo "DEPLOY_PASSWORD=$(openssl rand -base64 24)" > .env
cat .env   # note bien la valeur générée, elle sera réutilisée à l'étape 5
```

### 3. Lancer les serveurs cibles

```bash
docker compose up -d --build
docker ps   # vérifier que les ports sont bien liés à 127.0.0.1, pas 0.0.0.0
```

### 4. Chiffrer les secrets applicatifs (une seule fois, si pas déjà fait)

```bash
cd ..
ansible-vault encrypt group_vars/all/vault.yml
```

### 5. Ajouter le mot de passe sudo au vault

```bash
ansible-vault edit group_vars/all/vault.yml --ask-vault-pass
```

Ajouter une ligne avec la valeur générée à l'étape 2 :
```yaml
vault_deploy_sudo_pass: "colle-ici-le-mot-de-passe-du-.env"
```

### 6. Vérifier la connexion (authentification par clé, pas de mot de passe SSH)

```bash
chmod 600 docker/ansible_lab_key
ansible -i inventory/hosts.ini webservers -m ping --ask-vault-pass
```

### 7. Déployer

```bash
ansible-playbook -i inventory/hosts.ini site.yml --ask-vault-pass
```

## Sécurité

- `docker/ansible_lab_key` (clé privée) et `docker/.env` sont exclus du repo via `.gitignore` — ils doivent être régénérés localement par quiconque clone ce projet.
- L'authentification SSH par mot de passe est désactivée dans l'image Docker (`PasswordAuthentication no`), seule la clé publique fournie au build permet de se connecter.
- Le mot de passe sudo est distinct du mot de passe de build et n'existe qu'à l'intérieur du vault chiffré.
- Les ports des conteneurs sont liés à `127.0.0.1` uniquement, pas accessibles depuis le réseau.

## Pour aller plus loin

- Ajouter un rôle `firewall` (ufw)
- Ajouter un rôle `monitoring` (node_exporter + Prometheus)
- Passer d'un inventaire statique à un inventaire dynamique Docker
- Intégrer le playbook dans une pipeline CI/CD (GitHub Actions)

## Auteur

Cécile — projet réalisé dans le cadre d'une formation pratique à Ansible.
