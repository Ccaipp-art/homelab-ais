# Changelog – Homelab AIS

Toutes les évolutions importantes du laboratoire sont documentées ici.

Format : `YYYY.MM` (année.mois)

---

## [2025.12] – Phase DNS & Documentation

### Ajouté
- Documentation complète de configuration DNS Bind9 (document 05).
- Zones DNS directes et reverse (exemples anonymisés).
- Intégration du DNS système via Netplan + systemd-resolved.
- Mise en place des templates GitHub :
  - Pull Request Template
  - CONTRIBUTING.md
- Ajout du changelog.

### Modifié
- Organisation générale de la documentation.

---

## [2025.11] – Phase Hardening initiale

### Ajouté
- Documentation durcissement Ubuntu Server (UFW, SSH, Fail2ban).
- Documentation installation VMs initiales.
- Bases de structure du repo.

---

## [Unreleased]

### Prévu
- Configuration DNS côté clients (Ubuntu & Rocky).
- Installation service Nginx + virtual host basé sur DNS.
- Mise en place du monitoring (Node Exporter, Prometheus, Grafana).
- Introduction Ansible + inventaire basé sur DNS.
