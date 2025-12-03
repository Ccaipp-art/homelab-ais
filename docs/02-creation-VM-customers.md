# 🚀 02 — Création des VMs « Customers »

*(Ubuntu Server 24.04 + Rocky Linux 9)*

## 🎯 Objectif

Mettre en place deux VMs clientes pour la future infrastructure :

* **Ubuntu Server 24.04** : serveur principal (services réseau, automatisation, durcissement).
* **Rocky Linux 9** : poste “entreprise” compatible RHEL, utilisé pour tests multi-OS (Ansible, SELinux, firewalld).

Les deux VMs sont entièrement installées, configurées, accessibles via SSH par clé depuis la VM Desktop, et intégrées dans l’inventaire Ansible.

---

# 🧱 1. Préparation générale

## 1.1 ISO utilisées

* `ubuntu-24.04-live-server-amd64.iso`
* `Rocky-9.4-x86_64-minimal.iso`

## 1.2 Organisation du stockage (anonymisé)

Les VMs sont stockées sur un **disque externe**, dans des dossiers séparés :

```
/media/<user>/ExternalDrive/VMware/Ubuntu-Server-24.04/
/media/<user>/ExternalDrive/VMware/RockyLinux-9/
```

📌 *Objectif : structure propre, snapshots faciles, aucune confusion.*

---

# 🖥️ 2. VM « Infra-Master » — Ubuntu Server 24.04

## 2.1 Configuration VMware

* **Type** : Ubuntu 64-bit
* **CPU** : 2 vCPU
* **RAM** : 2 Go
* **Disque** : 20–40 Go (single file)
* **Réseau** : NAT
* **Graphiques** : désactivés
* **ISO** attachée

## 2.2 Installation

* Installation minimale
* Langue FR
* Partitionnement automatique + LVM
* Chiffrement LUKS désactivé
* OpenSSH activé
* Hostname : **svr-main**
* Utilisateur administrateur : **svc-admin**

## 2.3 Post-installation

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y htop net-tools curl wget git ufw
```

## 2.4 Sécurisation minimale

```bash
sudo ufw allow OpenSSH
sudo ufw enable
```

## 2.5 Snapshot baseline

Snapshot réalisé après configuration minimale + mises à jour.

---

# 🧱 3. VM « Client Enterprise » — Rocky Linux 9

## 3.1 Configuration VMware

* **Type** : Red Hat Enterprise Linux 64-bit
* **CPU** : 1–2 vCPU
* **RAM** : 2 Go
* **Disque** : 20 Go
* **Réseau** : NAT
* **Graphiques** : désactivés

## 3.2 Installation

* Installation **Minimal Install**
* Partitionnement automatique + LVM
* Chiffrement désactivé
* Hostname : **rocky-client**
* Utilisateur principal : **svc-admin** (sudoer)
* Root password défini mais accès SSH root désactivé

## 3.3 Post-installation

### Mise à jour

```bash
sudo dnf update -y
```

### Dépôt EPEL (nécessaire pour certains outils)

```bash
sudo dnf install -y epel-release
```

### Outils essentiels

```bash
sudo dnf install -y htop git net-tools bind-utils python3
```

## 3.4 Snapshot baseline

Snapshot réalisé après installation + updates.

---

# 🔐 4. Accès SSH par clé depuis la VM Desktop

La VM Ubuntu Desktop dispose de la clé SSH générée localement.
Déploiement de la clé vers chaque VM :

```bash
ssh-copy-id svc-admin@<UBUNTU_SERVER_IP>
ssh-copy-id svc-admin@<ROCKY_SERVER_IP>
```

Tests :

```bash
ssh svc-admin@<VM_IP>
```

➡️ Connexion sans mot de passe opérationnelle.

---

# 🤖 5. Intégration Ansible

## 5.1 Inventaire (anonymisé)

`inventory.ini` :

```ini
[ubuntu]
svr-main ansible_host=<UBUNTU_SERVER_IP> ansible_user=svc-admin

[rocky]
rocky-client ansible_host=<ROCKY_SERVER_IP> ansible_user=svc-admin
```

## 5.2 Tests Ansible

### Ubuntu Server :

```bash
ansible -i inventory.ini ubuntu -m ping
```

**Résultat :**

```
svr-main | SUCCESS => { "ping": "pong" }
```

### Rocky Linux :

```bash
ansible -i inventory.ini rocky -m ping
```

**Résultat :**

```
rocky-client | SUCCESS => { "ping": "pong" }
```

Ansible est donc pleinement fonctionnel sur les deux OS.

---

# 📝 6. Résumé des actions réalisées

* Création des deux VMs
* Installation minimale Ubuntu + Rocky
* Configuration réseau NAT
* SSH activé et sécurisé
* Installation des outils essentiels
* Snapshots baselines créés
* Poste d’administration prêt
* Intégration complète dans Ansible
* Tests multi-OS réussis

---

# 🧭 7. Impact sur la cohérence du lab

Les deux VMs représentent la base :

* **Ubuntu Server** : serveur principal (DNS, Nginx, monitoring, durcissement)
* **Rocky Linux** : client RHEL-like (SELinux, firewalld, tests Ansible multi-OS)
* **Ubuntu Desktop** : poste d’administration unique (SSH + Ansible)

Le tout forme une architecture **réaliste**, **professionnelle** et **pédagogique**, prête pour le déploiement des services.

