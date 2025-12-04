# 📘 **README.md – Hardening Ubuntu Server (Semaine 1)**

## 1. 🎯 Objectif du hardening

Mettre en place un durcissement minimal mais professionnel sur le serveur principal Ubuntu 24.04 (Infra-Master) avant d’y déployer les services réseau (DNS, Nginx, Reverse Proxy, Prometheus…).

Ce hardening comprend :

* Sécurisation SSH (clé uniquement)
* Désactivation de l’authentification par mot de passe
* Gestion correcte des overrides Ubuntu (cloud-init)
* Mise en place d’UFW (firewall)
* Installation et configuration Fail2ban
* Vérifications finales
* Documentation des erreurs rencontrées et résolution

---

## 2. 🧩 Contexte & Prérequis

* VM : Ubuntu Server 24.04 (svr-main)
* Accès depuis le poste d’administration Ubuntu Desktop via clé SSH
* Objectif : préparer la Semaine 2 de la roadmap (services réseau)
* Aucun service réseau n’est encore installé → parfait pour durcir

---

## 3. ⚙️ Étapes réalisées

### 3.1 🔐 SSH → connexion par clé uniquement

Fichier principal édité :

`/etc/ssh/sshd_config`

Paramètres modifiés :

```conf
PermitRootLogin no
PasswordAuthentication no
ChallengeResponseAuthentication no
KbdInteractiveAuthentication no
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
UsePAM yes
```

Redémarrage SSH :

```bash
sudo systemctl restart ssh
```

#### ✔ Résultat attendu

Depuis Desktop :

```bash
ssh -o PreferredAuthentications=password svc-admin@IP
```

→ `Permission denied (publickey).`

SSH fonctionne uniquement par clé.

---

### 3.2 🟦 Override propre : création de 99-hardening.conf

Fichier créé :

`/etc/ssh/sshd_config.d/99-hardening.conf`

Contenu :

```conf
PasswordAuthentication no
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no
```

Vérification :

```bash
sudo sshd -t
```

---

### 3.3 🔎 Problème découvert : cloud-init override SSH

Fichier en cause :

`/etc/ssh/sshd_config.d/50-cloud-init.conf`

Contenu original :

```conf
PasswordAuthentication yes
```

✔ Ce fichier écrasait la configuration hardening → SSH acceptait encore les mots de passe.

### Solution retenue :

Modifier ce fichier :

```bash
sudo nano /etc/ssh/sshd_config.d/50-cloud-init.conf
```

Mettre :

```
PasswordAuthentication no
```

Vérification finale :

```bash
sudo sshd -T | grep -i password
```

Résultat attendu :

```
passwordauthentication no
```

---

### 3.4 🟩 UFW → Firewall minimaliste

Activation :

```bash
sudo ufw allow OpenSSH
sudo ufw enable
```

État :

```bash
sudo ufw status verbose
```

Résultat attendu :

```
Status: active
OpenSSH ALLOW Anywhere
Default: deny (incoming), allow (outgoing)
```

---

### 3.5 🟣 Fail2ban → Protection anti-bruteforce SSH

Installation :

```bash
sudo apt install fail2ban -y
```

Configuration :

`/etc/fail2ban/jail.local`

```conf
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
backend = systemd

maxretry = 5
findtime = 10m
bantime = 30m
ignoreip = 127.0.0.1/8
```

Vérification :

```bash
sudo fail2ban-client status sshd
```

**Note :**
Fail2ban n’aura aucune alerte car SSH est en mode **key-only** → fonctionnement normal et sain.

---

## 4. 🛑 Erreurs rencontrées & résolution

### ❌ 1. SSH continuait à accepter les mots de passe

**Cause :**
`50-cloud-init.conf` contenait `PasswordAuthentication yes`.

**Symptômes :**

* `ssh -o PreferredAuthentications=password` permettait encore la connexion
* `sshd -T` indiquait `passwordauthentication yes`

**Solution :**

* Création de `99-hardening.conf` (override propre)
* Puis modification de `50-cloud-init.conf`

---

### ❌ 2. Impossible de tester Fail2ban

**Cause :**
SSH key-only → aucun mot de passe n’est traité
Le serveur ne génère donc aucun “failed login attempt”.

**Solution :**

* Test non applicable
* Fail2ban reste actif pour d'autres services (HTTP/Nginx) à venir

---

### ❌ 3. UFW déjà activé d’une étape précédente

✔ Normal
→ Juste vérifier la configuration avant de continuer

---

## 5. ✔ Résultat final du hardening

Le serveur `svr-main` est maintenant :

* Protégé par **SSH key-only**
* Indéfectible face aux brute-force par mot de passe
* Protégé par un **firewall strict**
* Doté d’un **Fail2ban pro**, prêt pour Nginx et SSH
* Conforme aux bonnes pratiques entreprise

**Semaine 1 terminée.**

---

## 6. 🧭 Suite des événements (Roadmap)

### 🟦 Semaine 2 — Services réseau

* Installation Bind9 (DNS primaire)
* Configuration zone `lab.local`
* Tests depuis Ubuntu Desktop + Rocky Linux
* Installation Nginx
* Mise en place Reverse Proxy + headers de sécurité

### 🟧 Semaine 3 — Supervision + Automatisation

* Prometheus + Node Exporter sur Ubuntu + Rocky
* Grafana
* Installation Ansible
* Hardening Rocky Linux via Ansible
