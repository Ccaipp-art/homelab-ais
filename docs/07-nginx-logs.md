# 📘 07 — Nginx : logs, inspection et snapshot

## 🧭 Contexte

Cette session s’inscrit dans la continuité du déploiement de Nginx sur un serveur Ubuntu.
L’objectif n’est plus seulement d’avoir un service fonctionnel, mais de **savoir l’inspecter, le diagnostiquer et le sécuriser dans le temps**.

Un serveur web n’est réellement exploitable que si l’on sait :

* lire ses logs,
* comprendre ses messages,
* identifier rapidement l’origine d’un problème,
* revenir à un état stable si nécessaire.

---

## 🎯 Objectifs de la session

* Comprendre **où et comment Nginx journalise son activité**
* Différencier **logs applicatifs HTTP** et **erreurs serveur**
* Savoir diagnostiquer un incident simple à partir des logs
* Valider l’état opérationnel du service
* Créer un **snapshot de référence** (checkpoint stable)

---

## 🧩 Prérequis

* VM **Ubuntu Server 24.04**
* Nginx installé et fonctionnel
* Accès SSH depuis la machine d’administration
* Accès à l’hyperviseur (VMware)

---

## 🧪 Inspection du service Nginx

### État du service

```bash
sudo systemctl status nginx
```

Permet de vérifier :

* que le service est actif
* depuis quand il tourne
* s’il y a eu des redémarrages récents

---

### Logs systemd (journal)

```bash
sudo journalctl -u nginx -n 200 --no-pager
```

Utilisé pour :

* suivre les démarrages / reloads
* identifier des erreurs de lancement
* confirmer un redémarrage propre

---

### Vérification de la configuration

```bash
sudo nginx -t
sudo nginx -T | less
```

* `nginx -t` : validation syntaxique
* `nginx -T` : configuration complète réellement chargée

---

### Vérification des ports

```bash
sudo ss -tulpn | grep -E ':(80|443)\b'
```

Confirme que Nginx écoute bien sur HTTP et HTTPS.

---

## 📂 Structure des fichiers Nginx

```bash
/etc/nginx/
├── nginx.conf
├── sites-available/
├── sites-enabled/
├── snippets/
└── conf.d/
```

Les VirtualHosts actifs sont visibles dans :

```bash
/etc/nginx/sites-enabled/
```

---

## 📊 Analyse des logs Nginx

### Emplacement des logs

```bash
/var/log/nginx/
├── access.log
├── error.log
├── site_access.log
└── site_error.log
```

---

### Access log — trafic HTTP

```bash
sudo tail -n 50 /var/log/nginx/access.log
```

Contient :

* IP cliente
* méthode HTTP
* URL
* code de réponse
* user-agent

Exemples observés :

* `200` → requête réussie
* `301` → redirection
* `404` → ressource inexistante

⚠️ **Un code 404 n’est pas une erreur serveur**.

---

### Analyse rapide des réponses

```bash
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -nr
```

Permet d’identifier rapidement :

* les codes dominants
* un comportement anormal (ex : trop de 404)

---

### Error log — erreurs internes Nginx

```bash
sudo tail -n 100 /var/log/nginx/error.log
```

Ce fichier ne contient **que les erreurs serveur réelles** :

* permissions filesystem
* certificats SSL
* erreurs de configuration
* backends inaccessibles

---

### Exemple de message observé

```
[notice] using inherited sockets from "5;6;"
```

➡️ Message informatif indiquant :

* un **reload propre**
* aucune interruption de service
* sockets réseau conservées

👉 Comportement attendu et sain.

---

## 🔬 Tests contrôlés

Depuis une machine cliente :

```bash
curl -I http://IP_DU_SERVEUR
curl -I http://IP_DU_SERVEUR/this-page-should-404
```

Les requêtes apparaissent immédiatement dans `access.log`.

---

## 📸 Snapshot / checkpoint

### Objectif

Figer un état stable :

* Nginx fonctionnel
* HTTPS opérationnel
* Logs compris et validés

---

### Méthode utilisée

**Snapshot à froid (recommandé)**

```bash
sudo systemctl stop nginx
sudo shutdown -h now
```

Puis :

* arrêt complet de la VM
* snapshot VMware **ou**
* copie complète du dossier VM

Nom du checkpoint :

```
nginx-logs-ok_YYYY-MM-DD
```

---

### Vérification post-snapshot

```bash
sudo systemctl status nginx
sudo nginx -t
curl -I http://IP_DU_SERVEUR
```

---

## 🛑 Erreurs fréquentes identifiées

* Confondre code HTTP (404) et erreur serveur
* Chercher des erreurs applicatives dans `error.log`
* Oublier de tester après un reload
* Multiplier les snapshots sans logique

---

## 📝 Points clés à retenir

* `access.log` = activité HTTP normale
* `error.log` = problèmes Nginx réels
* Un `error.log` calme est un bon signe
* Reload ≠ restart
* Un snapshot doit correspondre à un **état documenté**

---

## 🧭 Impact sur le laboratoire

Cette session marque le passage :

* d’un service fonctionnel
* à un service **opérationnel et maîtrisé**

Elle constitue un **socle solide** avant :

* l’automatisation avec Ansible
* la supervision (Prometheus / Grafana)
