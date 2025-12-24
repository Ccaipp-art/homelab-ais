# 📘 08 — Nginx : automatisation avec Ansible

## 🧭 Contexte

Après avoir déployé et inspecté un service Nginx manuellement, l’étape suivante consiste à **rendre ce déploiement reproductible, fiable et maintenable**.

En environnement professionnel, un serveur web n’est jamais configuré “à la main” sur le long terme.
Toute modification doit pouvoir être :

* rejouée,
* auditée,
* corrigée rapidement,
* appliquée de manière identique sur plusieurs machines.

Cette session introduit **Ansible** comme outil d’automatisation, avec une approche volontairement progressive et structurée.

---

## 🎯 Objectifs de la session

* Comprendre le rôle d’Ansible dans une infrastructure
* Mettre en place un **poste de contrôle** distinct du serveur
* Automatiser le déploiement complet de Nginx
* Gérer la configuration par **vhost**, sans casser la structure système
* Intégrer :

  * TLS auto-signé
  * headers de sécurité
  * contenu web
* Valider une exécution **idempotente**
* Identifier et corriger des erreurs réalistes

---

## 🧩 Prérequis

* VM **Ubuntu Desktop** (poste de contrôle)
* VM **Ubuntu Server 24.04** (serveur web)
* Accès SSH fonctionnel
* Compte de service dédié (`svc-admin`)
* Nginx déjà installé (session précédente)

> Toutes les IP, noms de machines et domaines sont volontairement **anonymisés**.

---

## 🏗️ Architecture retenue

| Élément        | Rôle                            |
| -------------- | ------------------------------- |
| Ubuntu Desktop | Exécution des playbooks Ansible |
| Ubuntu Server  | Hébergement du service Nginx    |
| Compte dédié   | Automatisation non interactive  |
| Accès          | SSH + sudo                      |

👉 Séparation claire entre **outil d’administration** et **infrastructure**.

---

## 📁 Structure Ansible

```text
ansible/
├── inventory.ini
├── nginx.yml
└── roles/
    └── nginx/
        ├── tasks/
        ├── handlers/
        ├── templates/
        └── vars/
```

Principes appliqués :

* inventaire séparé
* playbook simple et lisible
* logique encapsulée dans un rôle dédié
* aucune modification directe des fichiers système critiques

---

## ⚙️ Fonctionnalités automatisées

### Déploiement Nginx

* installation du paquet
* gestion du service via handler
* validation systématique de la configuration (`nginx -t`)

---

### Gestion propre de la configuration

* utilisation de :

  * `sites-available`
  * `sites-enabled`
  * `snippets`
* **aucun écrasement** de `/etc/nginx/nginx.conf`
* ajout d’un contrôle empêchant toute régression future

---

### Sécurisation HTTP / HTTPS

* génération automatique d’un certificat TLS auto-signé
* activation conditionnelle de HTTPS
* redirection HTTP → HTTPS
* déploiement de headers de sécurité :

  * X-Frame-Options
  * X-Content-Type-Options
  * Referrer-Policy
  * Permissions-Policy

---

### Contenu web

* création du webroot
* déploiement d’une page `index.html`
* permissions adaptées à l’utilisateur du service web

---

## 🔁 Concepts Ansible validés

* **Idempotence**
  → relancer le playbook ne provoque aucun changement inutile

* **Handlers**
  → Nginx est rechargé uniquement si nécessaire

* **Templates Jinja2**
  → configuration dynamique et réutilisable

* **Variables**
  → ports, chemins, activation TLS, server_name

* **`become` / sudo**
  → automatisation non interactive

---

## 🛑 Incidents rencontrés et analyse

### 1️⃣ `Missing sudo password`

**Symptôme**
Échec du playbook dès la collecte des facts.

**Cause**
Le compte Ansible nécessitait un mot de passe pour sudo.

**Correction**
Configuration d’un sudo non interactif via `visudo`.

**Enseignement**

> Une automatisation ne doit jamais dépendre d’une interaction humaine.

---

### 2️⃣ Nginx ne charge plus les vhosts

**Symptôme**
Les fichiers sont présents, mais Nginx n’écoute plus sur 443.

**Cause**
Un fichier `nginx.conf` minimal a supprimé l’inclusion de `sites-enabled`.

**Correction**
Restauration d’un `nginx.conf` standard + ajout d’un contrôle Ansible.

**Enseignement**

> Les fichiers structurants du système doivent être protégés.

---

### 3️⃣ HTTP 403 Forbidden

**Symptôme**
Serveur accessible, HTTPS fonctionnel, mais réponse 403.

**Cause**
Webroot vide ou index absent.

**Correction**
Création du dossier et déploiement du contenu via Ansible.

**Enseignement**

> Un service peut être sain mais inutilisable sans contenu valide.

---

### 4️⃣ TLS présent mais port 443 non écouté

**Symptôme**
Certificats présents, mais connexion refusée.

**Cause**
Vhost non chargé par la configuration effective.

**Correction**
Diagnostic via `ss` et `nginx -T`, correction des inclusions.

**Enseignement**

> Certificat ≠ service actif.

---

## 🧪 Vérifications finales

```bash
ansible-playbook nginx.yml
```

Résultat attendu :

* `changed=0`
* aucune erreur

Tests côté client :

```bash
curl -kI https://<serveur>
```

---

## 📝 Points clés à retenir

* Automatiser ≠ tout écraser
* La structure Nginx standard doit être respectée
* Les erreurs rencontrées sont **formatrices**
* L’idempotence est un critère central
* Ansible est un outil de fiabilisation, pas de magie

---

## 🧭 Impact sur le laboratoire

Cette session marque le passage :

* d’une configuration manuelle
* à une **infrastructure reproductible**

Elle prépare naturellement les prochaines étapes :

* support multi-OS
* DNS réel
* supervision
* automatisation avancée
