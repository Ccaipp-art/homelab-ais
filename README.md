# Homelab AIS – Ubuntu & Rocky

## 👋 Contexte

Je m'appelle Théo FRANCOIS, je suis en formation **Administrateur Infrastructures Sécurisées**
et je prépare une alternance à partir d'avril 2026.

Ce dépôt présente un **laboratoire d'infrastructure que j'ai construit chez moi**, pour :
- consolider mes bases Linux (Ubuntu & Rocky)
- pratiquer l'administration système et la sécurité
- expérimenter la supervision et un début d'automatisation
- me préparer à un environnement de type entreprise (TotalEnergies, etc.)

## 🏗️ Architecture du lab

- **VM Ubuntu Desktop LTS** : poste d'admin (VSCode, Ansible, SSH, documentation)
- **VM Ubuntu Server 24.04** : serveur principal (DNS, Nginx, supervision, durcissement)
- **VM Rocky Linux 9** : client type entreprise (RHEL-like, SELinux, Firewalld, tests)

A faire
![Schéma d'architecture](diagrams/architecture-homelab.png)

## 🎯 Objectifs pédagogiques

- Mise en place d'une petite infra cohérente et reproductible
- Apprentissage du durcissement système (SSH, firewall, services)
- Découverte de la supervision (Prometheus + Grafana)
- Premier pas vers l'automatisation (Ansible)
- Mise en pratique de bonnes pratiques d'admin et de documentation

## 🗂 Organisation du dépôt

- `docs/` : documentation technique (par VM et par thème)
- `ansible/` : inventaire et playbooks Ansible
- `scripts/` : scripts shell (backup, checks, etc.)
- `config/` : fichiers de configuration (Nginx, Prometheus, etc.)
- `diagrams/` : schémas d'architecture
- `notes/` : TODO et questions pour la suite

## 🔗 Documentation détaillée

La documentation exhaustive de ce lab (journal d'apprentissage, erreurs, réflexions)
est disponible sur Notion :

👉 [Accéder à la page Notion](https://... lien ici ...)

## 🚧 État d'avancement

- [ ] Création des 3 VM
- [ ] SSH + clés
- [ ] DNS interne
- [ ] Nginx + reverse proxy
- [ ] Prometheus + Grafana
- [ ] Ansible (hardening + installation de services)
- [ ] MinIO + script de backup

Ce dépôt est vivant : je le mets à jour au fur et à mesure de ma progression.
