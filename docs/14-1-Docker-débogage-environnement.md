# 📅 Session du jour — Docker & Débogage environnement

---

# 🎯 Objectif de la session

* Reprendre le projet Flask
* Installer Docker
* Dockeriser l’application
* Comprendre et corriger les erreurs rencontrées

---

# 🧩 Travaux réalisés

## 🌐 1. Incident réseau — environnement VMware

### Symptôme

* Perte d’accès Internet dans les VM
* Erreurs lors de l’installation de paquets :

```text
Temporary failure resolving
```

---

### Diagnostic

* Test de connectivité :

  * `ping 8.8.8.8`
  * `ping google.com`

* Identification du problème :

  * services VMware arrêtés côté machine hôte :

    * VMware NAT Service
    * VMware DHCP Service

* Contexte aggravant :

  * déconnexion du disque externe
  * changement de lettre (E → D)

---

### Résolution

* Redémarrage des services VMware
* Vérification du réseau dans les VM

---

### Apprentissage

* Les VM dépendent des services réseau du système hôte
* Toujours tester :

  1. connectivité IP
  2. DNS
  3. configuration hyperviseur

---

## 🐳 2. Installation de Docker

### Actions

* Installation via le dépôt officiel Docker
* Vérification du service :

```bash
docker --version
```

---

### Problème rencontré

```text
permission denied while trying to connect to the Docker daemon socket
```

---

### Cause

* utilisateur non actif dans le groupe `docker`

---

### Résolution

* ajout au groupe :

```bash
sudo usermod -aG docker $USER
```

* rechargement de session (`newgrp docker`)

---

### Validation

```bash
docker run hello-world
```

---

## 🐍 3. Préparation de l’application Flask

* Reprise du projet existant
* Vérification du bon fonctionnement local
* Validation des endpoints :

  * `/`
  * `/health`

---

## 🐳 4. Dockerisation de l’application Flask

### Création des fichiers

* `Dockerfile`
* `.dockerignore`

---

### Configuration

* utilisation d’une image Python légère
* copie des fichiers applicatifs
* installation des dépendances via `requirements.txt`
* exposition du port 5000

---

### Modification importante

Dans `app.py` :

```python
app.run(host="0.0.0.0", port=5000)
```

👉 nécessaire pour l’accès depuis l’extérieur du container

---

## 🛑 5. Erreur critique rencontrée (build Docker)

### Symptôme

```text
pip install -r requirements.txt échoue
```

---

### Cause

* `requirements.txt` contenait de nombreux packages système
* fichier généré depuis un environnement pollué

Exemples :

* `apturl`
* `dbus-python`
* `systemd-python`
* `ufw`

---

### Résolution

* simplification du fichier :

```text
Flask==3.1.3
```

---

### Apprentissage

* un `requirements.txt` doit contenir uniquement les dépendances applicatives
* éviter d’utiliser `pip freeze` sans environnement propre

---

## 🚀 6. Résultat final

* image Docker construite avec succès
* container lancé :

```bash
docker run --rm -p 5000:5000 flask-demo:1.0
```

* API accessible via :

```bash
curl http://127.0.0.1:5000/
curl http://127.0.0.1:5000/health
```

---

# 🧠 Concepts appris

* fonctionnement du réseau VMware (NAT / DHCP)
* gestion des permissions Docker
* cycle de vie d’une image Docker (build → run)
* importance des dépendances applicatives propres
* différence entre environnement local et container

---

# 🛑 Difficultés rencontrées

* perte réseau complète des VM
* permissions Docker non appliquées immédiatement
* erreur Docker build liée au `requirements.txt`

---

# 🚀 Prochaines étapes

* déployer le container sur le serveur `svr-main`
* mettre en place un reverse proxy Nginx
* automatiser le déploiement
* introduire CI/CD

---

# 📌 Conclusion

Session centrée sur la partie DevOps :

* résolution d’un incident réel
* installation Docker
* première application conteneurisée

👉 étape clé dans la transition vers le déploiement applicatif.
