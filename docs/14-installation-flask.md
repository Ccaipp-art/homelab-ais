
# 🐍 Installation d’une application Flask (environnement de développement)

---

# 🎯 Objectif

Mettre en place un environnement de développement Python isolé et installer une application Flask simple.

Cette étape constitue la base pour :

* le développement applicatif
* la conteneurisation (Docker)
* le déploiement automatisé (CI/CD)

---

# 🧩 Prérequis

* VM Ubuntu Desktop (poste de travail)
* Accès Internet fonctionnel
* Python3 installé

Vérification :

```bash
python3 --version
```

---

# ⚙️ Étapes d’installation

## 1. Installation du module venv

Par défaut, le module permettant de créer des environnements virtuels n’est pas toujours installé.

```bash
sudo apt update
sudo apt install -y python3-venv
```

---

## 2. Création de l’environnement virtuel

Dans le projet :

```bash
cd ~/workspace/homelab-ais/app/flask-demo
python3 -m venv .venv
```

---

## 3. Activation de l’environnement

```bash
source .venv/bin/activate
```

Résultat attendu :

```text
(.venv)
```

---

## 4. Installation de Flask

```bash
pip install flask
```

---

## 5. Vérification de l’environnement

```bash
which python
```

Résultat attendu :

```text
.../flask-demo/.venv/bin/python
```

---

## 6. Génération des dépendances

```bash
pip freeze > requirements.txt
```

---

# 🔍 Vérification

Lister les paquets installés :

```bash
pip list
```

Flask doit apparaître dans la liste.

---

# 🛑 Erreurs rencontrées

## ❌ Erreur : ensurepip is not available

```text
The virtual environment was not created successfully because ensurepip is not available
```

### Cause

Le package `python3-venv` n’est pas installé.

### Solution

```bash
sudo apt install python3-venv
```

---

## ❌ Erreur : Temporary failure resolving

```text
Temporary failure resolving 'fr.archive.ubuntu.com'
```

### Cause

Problème réseau ou DNS sur la VM.

### Solution

* Vérifier la connectivité réseau :

```bash
ping -c 3 8.8.8.8
ping -c 3 google.com
```

* Vérifier les services VMware (NAT / DHCP)

---

## ❌ Incident : perte réseau globale des VM

### Symptôme

* Plus d’accès Internet
* apt ne fonctionne plus

### Cause

Services VMware arrêtés côté machine hôte :

* VMware NAT Service
* VMware DHCP Service

### Solution

Redémarrage des services côté hôte Windows.

---

# 🧠 Concepts importants

## Environnement virtuel Python

Permet :

* d’isoler les dépendances
* d’éviter les conflits entre projets
* de reproduire un environnement facilement

---

## Bonne pratique

Ne jamais installer les dépendances Python globalement :

```text
Toujours utiliser un environnement virtuel (.venv)
```

---

# 🧭 Cohérence avec le laboratoire

* Ubuntu Desktop → environnement de développement
* Ubuntu Server → environnement d’exécution (production)

Cette séparation permet de reproduire une architecture professionnelle.

---

# 📌 Résultat

À la fin de cette étape :

* environnement Python isolé fonctionnel
* Flask installé
* base prête pour :

  * création d’une API
  * conteneurisation Docker
  * automatisation CI/CD
