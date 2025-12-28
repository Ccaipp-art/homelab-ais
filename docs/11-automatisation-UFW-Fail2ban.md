# 🔐 Session 11 — Automatisation de la sécurité (UFW, SSH, Fail2ban)

> 📌 **Homelab – Formation Administrateur Infrastructures Sécurisées**
> Cette session a pour objectif de montrer comment **automatiser proprement la sécurité d’un serveur Linux**, sans casser les services existants.

---

## 🎯 Objectifs de la session

* Automatiser la sécurité réseau avec **Ansible**
* Mettre en place un **durcissement SSH avancé**
* Déployer **Fail2ban** de façon raisonnée
* Nettoyer une configuration existante (doublons firewall)
* Garantir la **continuité de service** (SSH, Web, DNS)
* Obtenir une configuration **lisible, maintenable et rejouable**

👉 Le tout **sans reset brutal**, comme en environnement réel.

---

## 🧩 Contexte (anonymisé)

* **OS serveur** : Ubuntu Server LTS
* **Rôles hébergés** :

  * Serveur Web (HTTP / HTTPS)
  * DNS interne (lab)
* **Accès** : SSH par clé
* **Sécurité existante** : configurée partiellement à la main

⚠️ Cette session ne part pas d’un serveur “vide”, mais d’un serveur **déjà en production lab**.

---

## 🏗️ Structure Ansible

Un rôle dédié à la sécurité a été créé afin de séparer clairement les responsabilités.

```text
roles/security/
├── tasks/
│   ├── main.yml
│   ├── ufw.yml
│   ├── ssh.yml
│   └── fail2ban.yml
├── handlers/
│   └── main.yml
├── templates/
│   ├── jail.local.j2
│   └── 99-ansible-hardening.conf.j2
└── vars/
    └── main.yml
```

👉 Cette structure permet :

* une lecture simple
* une évolution progressive
* une automatisation propre

---

## 🔥 Étape 1 — Sécurité réseau (UFW)

### Choix retenus

* Politique par défaut :

  * **Entrant : refusé**
  * **Sortant : autorisé**
* Utilisation des **profils UFW officiels** :

  * `OpenSSH`
  * `Nginx Full`
* Règles spécifiques conservées pour le lab :

  * DNS (`53/tcp` et `53/udp`) restreint au réseau local

### Pourquoi utiliser les profils UFW ?

* Plus lisibles que des ports “bruts”
* Maintenus par la distribution
* Plus faciles à auditer et expliquer

---

## 🔐 Étape 2 — Durcissement SSH avancé

### Méthode utilisée

Au lieu de modifier directement `/etc/ssh/sshd_config`, un **fichier drop-in** est utilisé :

```text
/etc/ssh/sshd_config.d/99-ansible-hardening.conf
```

👉 Avantages :

* plus sûr
* réversible
* compatible avec les mises à jour système

### Mesures appliquées

* Interdiction du login root
* Désactivation de l’authentification par mot de passe
* Authentification par clé uniquement
* Restriction des utilisateurs autorisés
* Limitation des tentatives de connexion
* Désactivation des forwards inutiles

### Sécurité anti lock-out

Avant chaque rechargement SSH :

```bash
sshd -t
```

👉 Si la configuration est invalide, **le service n’est pas rechargé**.

---

## 🚫 Étape 3 — Protection brute-force (Fail2ban)

### Philosophie

* Un seul jail (`sshd`)
* Pas de sur-configuration
* Paramètres compréhensibles

### Configuration

* Surveillance des logs via `systemd`
* Délais et tentatives limités
* IPs de confiance ignorées (lab)

### Point important

Le serveur utilise une **authentification SSH par clé uniquement**.
➡️ Les attaques par mot de passe classiques ne génèrent donc pas de bannissement automatique.

👉 Le fonctionnement de Fail2ban a été validé via **tests contrôlés**, sans affaiblir la sécurité.

---

## 🧹 Étape 4 — Nettoyage du firewall

### Problème rencontré

Des règles firewall redondantes existaient :

* profils UFW
* ports ouverts manuellement

### Solution

* Conservation des profils (`OpenSSH`, `Nginx Full`)
* Suppression des règles port-par-port redondantes
* Vérification systématique après chaque suppression

### Résultat

* Firewall plus lisible
* Configuration plus propre
* Audit facilité

---

## 🔍 Tests de validation

Tous les services ont été testés après automatisation :

* 🔐 Connexion SSH
* 🌐 Accès HTTP / HTTPS
* 🌍 Résolution DNS
* 🚫 Fonctionnement de Fail2ban

👉 Aucun service n’a été cassé.

---

## 🧠 Ce que cette session démontre

* Automatiser **sans casser l’existant**
* Travailler par **convergence d’état**
* Sécuriser progressivement
* Comprendre ce que l’on automatise
* Produire une configuration claire et défendable

---

## 🧭 Conclusion

Cette session marque une étape clé du homelab :

* 🔁 Sécurité rejouable via Ansible
* 🔐 SSH durci proprement
* 🔥 Firewall clair et maintenable
* 🚫 Protection brute-force active

👉 Une base solide pour aller plus loin (multi-OS, supervision, cloud local…).
