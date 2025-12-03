# Homelab AIS — Ubuntu & Rocky • Automation & Security

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
* renforcer et approfondir l’automatisation (Ansible)
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

* Installation propre et minimaliste
* Gestion des services (systemd)
* Sécurisation SSH
* Firewall (UFW / firewalld)
* Gestion des utilisateurs et sudoers

### ✔️ Réseau & services

* DNS interne (Bind9 / Unbound)
* Nginx + reverse proxy
* TLS auto-signé

### ✔️ Automatisation (déjà commencé → approfondissement prévu)

* Ansible multi-OS
* Inventaire structuré
* Playbooks pour hardening et déploiement de services
* Reproductibilité des environnements

### ✔️ Supervision (déjà expérimenté → amélioration prévue)

* Prometheus
* Node Exporter (Ubuntu + Rocky)
* Dashboards Grafana

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

Une version encore plus détaillée (journal d’apprentissage, debugging, réflexions) est sur **Notion** :
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

* Installation & configuration Linux server
* Ansible multi-OS (Ubuntu + Rocky)
* Automatisation basique (inventaire + modules ping)
* Sécurisation SSH par clés
* Architecture VMware NAT
* Gestion des paquets : `apt`, `dnf`, `epel-release`
* Documentation Markdown complète
* Workflow Git (branches, merge, PR, graph clair)

---

## 📬 Contact

Pour toute suggestion ou conseil :

📫 **[theoh.francois@laposte.net](mailto:theoh.francois@laposte.net)**
🌴 *Linktree : [https://linktr.ee/tfs_ccaipp](https://linktr.ee/tfs_ccaipp)*

📍 *Ouvert aux opportunités d’alternance (administration systèmes / DevOps) à partir d’avril 2026.*
