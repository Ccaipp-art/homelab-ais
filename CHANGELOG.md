# Changelog – Homelab AIS

Toutes les évolutions importantes du laboratoire sont documentées ici.

Format : `YYYY.MM.DD` (année.mois.jour)

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

### Ajouté
- Stabilisation du réseau virtuel (VMnetX) et correction des adresses IP statiques.
- Correction DNS : Bind9 fonctionnel côté serveur (résolution interne + externe).
- Ouverture des ports DNS (TCP/53 et UDP/53) dans UFW.
- Intégration du client Linux (Rocky-like) au DNS interne.
- Normalisation des permissions SSH (`.ssh` et `authorized_keys`) sur le serveur.
- Inventaire Ansible opérationnel (serveur + client) avec tests `ping` validés.
- Ajout de la documentation « Stabilisation réseau & DNS + intégration client » (document 05 – version anonymisée).
