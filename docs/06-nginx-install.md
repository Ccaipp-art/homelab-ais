# Nginx sécurisé (HTTPS auto-signé) avec DNS interne — Homelab AIS

> Objectif : déployer un premier service web **interne** (lab) en suivant une progression propre :
> **DNS → HTTP → VirtualHost → HTTPS → Redirection → Headers de sécurité**
>
> 🔒 **Anonymisation** : toutes les IP/hosts internes sont remplacés par des placeholders.
> - Serveur : `srv1.lab.<tld>` → `192.168.x.x`
> - DNS : `<ip_dns>`
> - LAN : `192.168.x.0/24`

---

## Sommaire
1. Contexte et objectifs
2. Rappels simples : DNS / HTTP / HTTPS
3. Architecture du lab (vue logique)
4. Pré-requis et checklist de départ
5. Déploiement étape par étape (débutant-friendly)
6. Vérifications (tests à faire)
7. Dépannage (erreurs fréquentes + diagnostics)
8. Sécurité & anonymisation (ce qui ne doit pas être public)
9. Fichiers modifiés / ajoutés (trace exhaustive)
10. Ce qu’il reste à faire (session suivante)
11. Journal de session (à copier dans Notion)

---

## 1) Contexte et objectifs

Cette session vise à mettre en place une base Nginx **propre et sécurisée** pour un lab de type *Administrateur Infrastructures Sécurisées (AIS)*.

À la fin, on obtient :
- un service web interne accessible via un nom DNS : `srv1.lab.<tld>`
- un **VirtualHost dédié** (pas le site par défaut)
- un accès en **HTTPS** (certificat auto-signé)
- une redirection **HTTP → HTTPS**
- des **headers de sécurité de base**

✅ Cette base est prête pour la suite :
- reverse proxy
- supervision
- automatisation Ansible

---

## 2) Rappels simples (débutant-friendly)

### DNS (à quoi ça sert ?)
Le DNS associe un **nom** à une **adresse IP**.
- Sans DNS : tu dois te souvenir d’une IP
- Avec DNS : tu utilises un nom clair (`srv1.lab.<tld>`)

### HTTP (simple)
HTTP permet à un navigateur (client) de demander une page à un serveur web.
⚠️ Problème : en HTTP, les données circulent **en clair**.

### HTTPS (simple)
HTTPS = HTTP + chiffrement (TLS).
- protège contre l’écoute réseau
- améliore l’intégrité des échanges
- authentifie le serveur via un certificat

Dans un lab, on utilise souvent un **certificat auto-signé** :
- le navigateur affiche un avertissement (normal)
- mais le chiffrement fonctionne

---

## 3) Architecture (vue logique)

```text
Client (Desktop / Rocky)
        |
        v
DNS interne (Bind9)  <ip_dns>
        |
        v
srv1.lab.<tld>  →  Nginx sur 192.168.x.x
                   ├── HTTP  (80)  → redirection 301 vers HTTPS
                   └── HTTPS (443) → page web + headers sécurité
````

---

## 4) Pré-requis et checklist de départ

### Machines

* **Ubuntu Server** : serveur principal (Bind9 + Nginx)
* **Ubuntu Desktop** : poste d’admin (SSH, tests, GitHub)
* (optionnel) **Rocky Linux** : client de test

### Checklist

* [ ] Le serveur DNS répond (Bind9 actif)
* [ ] Le client résout `srv1.lab.<tld>`
* [ ] SSH OK vers le serveur
* [ ] Firewall actif (UFW) et contrôlé

---

## 5) Déploiement étape par étape (progressif)

> 📌 Tout ce qui suit est fait sur **Ubuntu Server** (via SSH depuis Ubuntu Desktop),
> sauf les tests “client”.

---

### Étape A — Valider le DNS (bloquant)

Sur **Ubuntu Desktop** :

```bash
dig srv1.lab.<tld> @<ip_dns>
ping -c 3 srv1.lab.<tld>
```

✅ Résultat attendu :

* `status: NOERROR` (dig)
* ping OK (ou au minimum résolution OK)

> Si `NXDOMAIN`, ne pas continuer : le nom n’existe pas dans la zone DNS.

---

### Étape B — Installer Nginx

Sur **Ubuntu Server** :

```bash
sudo apt update
sudo apt install nginx -y
systemctl status nginx --no-pager
curl http://localhost
```

✅ Résultat attendu : page “Welcome to nginx!”

---

### Étape C — Ouvrir le firewall (HTTP/HTTPS)

Sur **Ubuntu Server** :

```bash
sudo ufw allow 'Nginx Full'
sudo ufw status
```

✅ Résultat attendu : règles 80/443 autorisées

---

### Étape D — Créer un VirtualHost dédié (propre)

#### D1. Créer le dossier web

Sur **Ubuntu Server** :

```bash
sudo mkdir -p /var/www/srv1.lab.<tld>
sudo nano /var/www/srv1.lab.<tld>/index.html
```

Exemple minimal :

```html
<h1>srv1.lab.<tld></h1>
<p>Service web interne — Homelab AIS</p>
```

#### D2. Créer le VirtualHost HTTP

Sur **Ubuntu Server** :

```bash
sudo nano /etc/nginx/sites-available/srv1.lab.<tld>
```

Contenu :

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name srv1.lab.<tld>;

    root /var/www/srv1.lab.<tld>;
    index index.html;

    access_log /var/log/nginx/srv1_access.log;
    error_log  /var/log/nginx/srv1_error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

#### D3. Activer le site

```bash
sudo ln -s /etc/nginx/sites-available/srv1.lab.<tld> /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

✅ Test :

```bash
curl http://srv1.lab.<tld>
```

---

### Étape E — Ajouter HTTPS (certificat auto-signé)

#### E1. Créer un dossier SSL

Sur **Ubuntu Server** :

```bash
sudo mkdir -p /etc/nginx/ssl
sudo chmod 700 /etc/nginx/ssl
```

#### E2. Générer le certificat auto-signé

```bash
sudo openssl req -x509 -nodes -newkey rsa:4096 \
  -keyout /etc/nginx/ssl/srv1.lab.<tld>.key \
  -out /etc/nginx/ssl/srv1.lab.<tld>.crt \
  -days 365
```

⚠️ Important : `Common Name (CN)` = **`srv1.lab.<tld>`** (exact)

#### E3. Créer le VirtualHost HTTPS

```bash
sudo nano /etc/nginx/sites-available/srv1.lab.<tld>-ssl
```

Contenu :

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name srv1.lab.<tld>;

    root /var/www/srv1.lab.<tld>;
    index index.html;

    ssl_certificate     /etc/nginx/ssl/srv1.lab.<tld>.crt;
    ssl_certificate_key /etc/nginx/ssl/srv1.lab.<tld>.key;

    access_log /var/log/nginx/srv1_ssl_access.log;
    error_log  /var/log/nginx/srv1_ssl_error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

#### E4. Activer le site HTTPS

```bash
sudo ln -s /etc/nginx/sites-available/srv1.lab.<tld>-ssl /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

✅ Test (accepte le cert auto-signé) :

```bash
curl -k https://srv1.lab.<tld>
```

---

### Étape F — Rediriger HTTP → HTTPS

On modifie le **VirtualHost HTTP** pour qu’il ne serve plus de contenu (redirection only).

Sur **Ubuntu Server** :

```bash
sudo nano /etc/nginx/sites-available/srv1.lab.<tld>
```

Remplacer par :

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name srv1.lab.<tld>;

    return 301 https://$host$request_uri;
}
```

Appliquer :

```bash
sudo nginx -t
sudo systemctl reload nginx
```

✅ Test :

```bash
curl -I http://srv1.lab.<tld>
```

Attendu :

* `HTTP/1.1 301 Moved Permanently`
* `Location: https://srv1.lab.<tld>/...`

---

### Étape G — Ajouter des headers de sécurité (baseline)

Sur **Ubuntu Server**, modifier le VirtualHost HTTPS :

```bash
sudo nano /etc/nginx/sites-available/srv1.lab.<tld>-ssl
```

Ajouter dans le bloc `server {}` :

```nginx
# --- Security headers (baseline) ---
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "DENY" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;

# HSTS (only after HTTPS + redirect are confirmed working)
add_header Strict-Transport-Security "max-age=31536000" always;
```

Appliquer :

```bash
sudo nginx -t
sudo systemctl reload nginx
```

✅ Test :

```bash
curl -kI https://srv1.lab.<tld>
```

---

## 6) Vérifications (tests à faire)

### Tests DNS

Sur Desktop (ou Rocky) :

```bash
dig srv1.lab.<tld> @<ip_dns>
getent hosts srv1.lab.<tld>
```

### Tests HTTP/HTTPS

Sur Desktop + Server :

```bash
curl -I http://srv1.lab.<tld>
curl -kI https://srv1.lab.<tld>
curl -k https://srv1.lab.<tld>
```

### Tests navigateur

* `http://srv1.lab.<tld>` doit rediriger vers `https://srv1.lab.<tld>`
* avertissement SSL normal (auto-signé)

---

## 7) Dépannage (erreurs fréquentes + diagnostic)

### NXDOMAIN (DNS)

**Symptôme** : `dig` renvoie `status: NXDOMAIN`
**Cause typique** : incohérence entre le nom testé et la zone DNS (ex: `.local` vs `.lan`)
**Fix** :

* vérifier la zone réelle
* aligner le FQDN utilisé partout

### Nginx sert encore la page par défaut

**Cause** :

* mauvais `server_name`
* site non activé (symlink absent)
  **Check** :

```bash
ls -l /etc/nginx/sites-enabled/
sudo nginx -T | grep -n "server_name"
```

### HTTPS ne répond pas

**Causes** :

* port 443 bloqué (UFW)
* vhost SSL pas activé
* `listen 443 ssl` manquant
  **Check** :

```bash
sudo ufw status
sudo ss -lntp | grep nginx
sudo nginx -t
```

### Boucle de redirection

**Cause** : redirection mise dans le mauvais vhost (ou sur HTTPS)
**Règle** : la redirection ne vit que dans le vhost **80**.

---

## 8) Sécurité & anonymisation (ce qui ne doit pas être public)

### À NE PAS publier

* IP réelles du lab
* noms réels de machines / utilisateurs
* fichiers complets de zones DNS si elles révèlent ton réseau
* clés privées TLS (`*.key`) (jamais)
* logs complets s’ils contiennent des infos internes

### Ce qui est OK en public

* schémas logiques
* commandes génériques
* extraits de config anonymisés
* tests `curl` / `nginx -t` (sans IP internes)

---

## 9) Fichiers modifiés / ajoutés (trace exhaustive)

### Nginx

* `/etc/nginx/sites-available/srv1.lab.<tld>` (HTTP → redirection)
* `/etc/nginx/sites-available/srv1.lab.<tld>-ssl` (HTTPS + headers)
* symlinks :

  * `/etc/nginx/sites-enabled/srv1.lab.<tld>`
  * `/etc/nginx/sites-enabled/srv1.lab.<tld>-ssl`
* contenu :

  * `/var/www/srv1.lab.<tld>/index.html`

### Certificats (ne jamais publier la clé)

* `/etc/nginx/ssl/srv1.lab.<tld>.crt`
* `/etc/nginx/ssl/srv1.lab.<tld>.key` 🔒

### Logs (créés par Nginx)

* `/var/log/nginx/srv1_access.log`
* `/var/log/nginx/srv1_error.log`
* `/var/log/nginx/srv1_ssl_access.log`
* `/var/log/nginx/srv1_ssl_error.log`

### Firewall

* UFW : règle `Nginx Full` (80/443)

---

## 10) Ce qu’il reste à faire (session suivante)

Session suivante recommandée : **Logs & inspection**

* comprendre les formats de logs
* générer du trafic depuis Desktop/Rocky
* lire erreurs et codes HTTP
* base pour monitoring / alerting

---

