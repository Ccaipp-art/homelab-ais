# 📘 10 — Sécurité réseau : audit UFW & Fail2ban (SSH)

## 🧭 Contexte

Après avoir mis en place un serveur web fonctionnel (Nginx, HTTPS, DNS interne), la priorité devient la **sécurisation de l’exposition réseau**.

Cette session ne vise **pas à ajouter de nouveaux services**, mais à :

* **auditer l’existant**,
* comprendre ce qui est réellement exposé,
* valider les mécanismes de protection déjà en place,
* s’assurer qu’ils sont cohérents, actifs et maîtrisés.

En environnement professionnel, **auditer et comprendre une configuration existante est aussi important que la déployer**.

---

## 🎯 Objectifs de la session

* Identifier précisément les **ports ouverts** sur le serveur
* Vérifier la configuration et le comportement du **firewall UFW**
* Auditer la configuration **Fail2ban existante**
* Comprendre la complémentarité :

  * firewall (préventif)
  * bannissement dynamique (réactif)
* Valider l’état de sécurité **sans modifier inutilement l’existant**

---

## 🧩 Prérequis

* VM **Ubuntu Server 24.04**
* Accès SSH fonctionnel par clé
* Compte de service dédié (`svc-admin`)
* Services déjà en place :

  * SSH
  * Nginx (HTTP / HTTPS)
  * Bind9 (DNS interne)

> Les noms de domaine, IP et identifiants sont **anonymisés**.

---

## 🔍 Étape 1 — Inspection des ports exposés

```bash
sudo ss -tulpen
```

### Ce que montre cette commande

* les ports ouverts (`LISTEN`)
* les protocoles (TCP / UDP)
* les adresses d’écoute
* les services associés

### Résumé observé

| Service | Port       | État                  |
| ------- | ---------- | --------------------- |
| SSH     | 22         | Ouvert                |
| HTTP    | 80         | Ouvert                |
| HTTPS   | 443        | Ouvert                |
| DNS     | 53 TCP/UDP | Ouvert (réseau local) |

👉 **Aucun port inattendu** n’est exposé.

---

## 🔥 Étape 2 — Vérification du firewall (UFW)

```bash
sudo ufw status verbose
```

### Politique par défaut

```text
Default: deny (incoming), allow (outgoing)
```

👉 Toute connexion entrante est bloquée **sauf règle explicite**.

---

### Règles autorisées

* SSH (22/tcp)
* HTTP / HTTPS (80, 443)
* DNS (53 TCP/UDP) **limité au réseau local**

👉 Exposition minimale et maîtrisée.

---

## 🔐 Étape 3 — Audit de Fail2ban (SSH)

⚠️ Important
Fail2ban **était déjà installé et configuré** lors de l’installation initiale du serveur.
Cette session vise donc à **auditer et comprendre** cette configuration.

---

### Configuration observée

Fichier :

```bash
/etc/fail2ban/jail.local
```

Jail actif :

```ini
[sshd]
enabled = true
maxretry = 5
findtime = 10m
bantime = 30m
```

👉 Configuration volontairement **conservatrice et sécurisée**.

---

### État du service

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

Résultat observé :

* 1 jail actif (`sshd`)
* aucune IP bannie
* aucun échec détecté

👉 Fonctionnement nominal.

---

## 🧠 Comprendre la complémentarité sécurité

### UFW (firewall)

* agit **avant la connexion**
* autorise ou bloque un flux réseau
* basé sur IP / port / protocole

### Fail2ban

* agit **après analyse des logs**
* détecte les comportements anormaux
* bannit dynamiquement une IP

👉 Les deux outils sont **complémentaires**, pas concurrents.

---

## 🛑 Choix assumés pendant la session

* ❌ Pas de test de bannissement volontaire
* ❌ Pas de multiplication de jails
* ❌ Pas de durcissement excessif

👉 Objectif : **stabilité, compréhension, maîtrise**.

---

## 🧪 Vérifications finales

* Connexion SSH fonctionnelle
* Services web accessibles
* DNS opérationnel
* Aucun bannissement actif
* Logs propres

---

## 📝 Points clés à retenir

* Auditer l’existant est une compétence essentielle
* Une sécurité efficace est **simple et lisible**
* UFW bloque, Fail2ban réagit
* Trop de règles ≠ plus de sécurité
* La stabilité prime sur la surconfiguration

---

## 🧭 Impact sur le laboratoire

Cette session marque :

* la **validation de la surface d’exposition réseau**
* la confirmation d’un **socle de sécurité cohérent**
* une base saine avant :

  * l’automatisation sécurité (Ansible)
  * la supervision
  * l’ouverture contrôlée vers d’autres services
