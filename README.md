# Homelab AIS — Ubuntu & Rocky • Automation & Security

<p align="center">
  <em>A scalable, well-documented Linux infrastructure project focused on Learning / SysAdmin / Security</em>
</p>

---

## 🛠️ Technologies & Tools

<p align="center">
  <img src="https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black" />
  <img src="https://img.shields.io/badge/Ubuntu_Server-E95420?logo=ubuntu&logoColor=white" />
  <img src="https://img.shields.io/badge/Rocky_Linux-10B981?logo=rockylinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Ansible-EE0000?logo=ansible&logoColor=white" />
  <img src="https://img.shields.io/badge/VMware-607078?logo=vmware&logoColor=white" />
  <img src="https://img.shields.io/badge/Prometheus-E6522C?logo=prometheus&logoColor=white" />
  <img src="https://img.shields.io/badge/Grafana-F46800?logo=grafana&logoColor=white" />
  <img src="https://img.shields.io/badge/Nginx-009639?logo=nginx&logoColor=white" />
  <img src="https://img.shields.io/badge/Markdown-000000?logo=markdown&logoColor=white" />
</p>

---

## 👋 About

I am currently training as a **Secure Infrastructure Administrator** and building a complete **homelab** to:

- strengthen my Linux skills (Ubuntu & Rocky Linux)
- practice system hardening and core network services
- deepen automation skills (Ansible)
- implement monitoring and observability (Prometheus + Grafana)
- document my learning progress in a professional way
- prepare for a **system administration apprenticeship** starting in April 2026

📌 **This repository is updated regularly** as I progress.

---

## 🏗️ Homelab Architecture

The lab is based on **3 VMware virtual machines**: one admin workstation and two managed servers.

```text
                +-------------------------+
                |  Ubuntu Desktop LTS     |
                |  Administration host    |
                |  VSCode • SSH Keys      |
                |  Git • Ansible          |
                +-----------+-------------+
                            |
                            | SSH + Ansible
                            |
    +-----------------------+-------------------------+
    |                                                 |
    |                                                 |
+---v-----------+                               +-----v---------+
| Ubuntu Server |                               | Rocky Linux 9 |
|  24.04 LTS    |                               | RHEL-like OS  |
| "svr-main"    |                               | "rocky-client"|
| DNS • Nginx   |                               | SELinux       |
| Hardening     |                               | Firewalld     |
| Monitoring    |                               | Ansible tests |
+---------------+                               +---------------+
```

🔜 *A PNG diagram will be added soon in `/diagrams`.*

---

## 🎯 Learning Objectives

### ✔️ Linux System Administration

* clean and minimal installations
* service management (systemd)
* SSH hardening
* firewall configuration (UFW / firewalld)
* user and sudoers management

### ✔️ Networking & Services

* internal DNS (Bind9 / Unbound)
* Nginx reverse proxy
* self-signed TLS certificates

### ✔️ Automation (started → deeper work planned)

* multi-OS Ansible usage
* structured inventory
* playbooks for hardening and service deployment
* reproducible environments

### ✔️ Monitoring (already explored → improvements planned)

* Prometheus
* Node Exporter (Ubuntu + Rocky)
* Grafana dashboards

### ✔️ Best Practices

* clear, versioned documentation
* clean Git structure (branches, PRs)
* VMware baseline snapshots

---

## 🗂 Repository Structure

```text
/
├── ansible/        → inventory and first playbooks
├── config/         → service configuration files (coming)
├── diagrams/       → architecture diagrams
├── docs/           → structured technical documentation
├── notes/          → TODOs, ideas, issues encountered
└── scripts/        → shell scripts (monitoring, backup…)
```

---

## 📄 Documentation

Full documentation (installation, tests, errors, reasoning, technical choices)
is available in:

📁 **`docs/`**

An even more detailed version (learning journal, debugging, reflections) will be on **Notion**:
👉 *(Link in progress)*

---

## 🚀 Current Status

### 🟩 Foundations completed

* [x] Creation of the 3 virtual machines
* [x] Ubuntu Server 24.04 installed
* [x] Rocky Linux 9 installed
* [x] SSH key-based access (Desktop → Ubuntu / Rocky)
* [x] Multi-OS Ansible inventory
* [x] `ansible -m ping` OK on Ubuntu & Rocky
* [x] Structured documentation (docs/ + README)
* [x] Baseline snapshots
* [x] Clean and readable Git repository structure

### 🟧 In progress / planned

* [ ] Internal DNS (Bind9)
* [ ] Reverse proxy (Nginx)
* [ ] Self-signed TLS (OpenSSL)
* [ ] Monitoring (Prometheus + Grafana)
* [ ] Hardening (SSH / services / firewall)
* [ ] Advanced Ansible playbooks
* [ ] MinIO + backup scripts

---

## 🧠 Skills Developed So Far

* Linux server installation and configuration
* Multi-OS Ansible (Ubuntu + Rocky)
* Basic automation (inventory + ping module)
* SSH security with keys
* VMware NAT architecture
* Package management: `apt`, `dnf`, `epel-release`
* Complete Markdown documentation
* Git workflow (branches, merges, PRs, clean graph)

---

## 📬 Contact

For any suggestion or feedback:

📫 **[theoh.francois@laposte.net](mailto:theoh.francois@laposte.net)**

📍 *Open to job opportunities ( DevOps) starting now.*

---

# Homelab AIS — Ubuntu & Rocky • Automatisation & Sécurité

<p align="center">
  <em>Un projet d’infrastructure Linux évolutif, documenté et orienté Apprentissage / AdminSys / Sécurité</em>
</p>

---

## 🛠️ Technologies & Outils

<p align="center">
  <img src="https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black" />
  <img src="https://img.shields.io/badge/Ubuntu_Server-E95420?logo=ubuntu&logoColor=white" />
  <img src="https://img.shields.io/badge/Rocky_Linux-10B981?logo=rockylinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Ansible-EE0000?logo=ansible&logoColor=white" />
  <img src="https://img.shields.io/badge/VMware-607078?logo=vmware&logoColor=white" />
  <img src="https://img.shields.io/badge/Prometheus-E6522C?logo=prometheus&logoColor=white" />
  <img src="https://img.shields.io/badge/Grafana-F46800?logo=grafana&logoColor=white" />
  <img src="https://img.shields.io/badge/Nginx-009639?logo=nginx&logoColor=white" />
  <img src="https://img.shields.io/badge/Markdown-000000?logo=markdown&logoColor=white" />
</p>

---

## 👋 À propos

Je suis en formation **Administrateur Infrastructures Sécurisées** et je construis un **homelab complet** pour :

* renforcer mes compétences Linux (Ubuntu et Rocky Linux)
* pratiquer le durcissement système et les services réseau
* approfondir l’automatisation (Ansible)
* mettre en place de la supervision (Prometheus + Grafana)
* documenter ma montée en compétence de manière professionnelle
* me préparer à une **alternance en administration système** à partir d’avril 2026

📌 **Ce dépôt est mis à jour régulièrement** au rythme de ma progression.

---

## 🏗️ Architecture du Homelab

Le lab repose sur **3 VMs VMware** : un poste d’admin et deux machines clientes.

```text
                +-------------------------+
                |  Ubuntu Desktop LTS     |
                |  Poste d’administration |
                |  VSCode • SSH Keys      |
                |  Git • Ansible          |
                +-----------+-------------+
                            |
                            | SSH + Ansible
                            |
    +-----------------------+-------------------------+
    |                                                 |
    |                                                 |
+---v-----------+                               +-----v---------+
| Ubuntu Server |                               | Rocky Linux 9 |
|  24.04 LTS    |                               |  Client RHEL  |
| "svr-main"    |                               | "rocky-client"|
| DNS • Nginx   |                               | SELinux       |
| Hardening     |                               | Firewalld     |
| Monitoring    |                               | Tests Ansible |
+---------------+                               +---------------+
```

🔜 *Schéma PNG disponible prochainement dans `/diagrams`.*

---

## 🎯 Objectifs pédagogiques

### ✔️ Administration Linux

* installation propre et minimaliste
* gestion des services (systemd)
* sécurisation SSH
* firewall (UFW / firewalld)
* gestion des utilisateurs et sudoers

### ✔️ Réseau & services

* DNS interne (Bind9 / Unbound)
* Nginx + reverse proxy
* TLS auto-signé

### ✔️ Automatisation (déjà commencé → approfondissement prévu)

* Ansible multi-OS
* inventaire structuré
* playbooks pour hardening et déploiement de services
* reproductibilité des environnements

### ✔️ Supervision (déjà expérimenté → amélioration prévue)

* Prometheus
* Node Exporter (Ubuntu + Rocky)
* dashboards Grafana

### ✔️ Bonnes pratiques

* documentation claire et versionnée
* structure Git propre (branches, PR)
* snapshots baseline VMware

---

## 🗂 Organisation du dépôt

```text
/
├── ansible/        → inventaire et premiers playbooks
├── config/         → fichiers de configuration des services (à venir)
├── diagrams/       → schémas d’architecture
├── docs/           → documentation technique structurée
├── notes/          → TODO, idées, problèmes rencontrés
└── scripts/        → scripts shell (monitoring, backup…)
```

---

## 📄 Documentation

La documentation complète (installation, tests, erreurs, raisonnement, choix techniques)
est disponible dans le dossier :

📁 **`docs/`**

Une version encore plus détaillée (journal d’apprentissage, debugging, réflexions) sera sur **Notion** :
👉 *(Lien en cours de création)*

---

## 🚀 État d’avancement

### 🟩 Fondations terminées

* [x] Création des 3 machines virtuelles
* [x] Installation Ubuntu Server 24.04
* [x] Installation Rocky Linux 9
* [x] SSH par clés (Desktop → Ubuntu / Rocky)
* [x] Inventaire Ansible multi-OS
* [x] `ansible -m ping` OK sur Ubuntu & Rocky
* [x] Documentation structurée (docs/ + README)
* [x] Snapshots baselines
* [x] Organisation du dépôt Git propre et lisible

### 🟧 En cours / à approfondir

* [ ] DNS interne (Bind9)
* [ ] Reverse proxy (Nginx)
* [ ] TLS auto-signé (OpenSSL)
* [ ] Monitoring (Prometheus + Grafana)
* [ ] Hardening (SSH / services / firewall)
* [ ] Playbooks Ansible avancés
* [ ] MinIO + scripts de sauvegarde

---

## 🧠 Compétences acquises jusqu’ici

* installation & configuration Linux server
* Ansible multi-OS (Ubuntu + Rocky)
* automatisation basique (inventaire + module ping)
* sécurisation SSH par clés
* architecture VMware NAT
* gestion des paquets : `apt`, `dnf`, `epel-release`
* documentation Markdown complète
* workflow Git (branches, merge, PR, graph clair)

---

## 📬 Contact

Pour toute suggestion ou conseil :

📫 **[theoh.francois@laposte.net](mailto:theoh.francois@laposte.net)**
📍 *Ouvert aux opportunités de travail (DevOps) dès à présent.*
