# 14.2 — Création et validation d’une image Docker Flask

## 🎯 Objectif

Créer une image Docker pour une API Flask locale, puis valider cette image avec des outils d’analyse et de contrôle utilisés dans les environnements DevOps / DevSecOps.

Image créée :

```text
flask-demo:1.0
```

---

# 🧩 Contexte

Le projet Flask est situé dans :

```text
app/flask-demo/
```

L’API expose deux routes :

| Route     | Fonction                |
| --------- | ----------------------- |
| `/`       | Réponse JSON principale |
| `/health` | Endpoint de healthcheck |

Le service écoute sur :

```text
port 5000
```

---

# 📁 Structure du projet

```text
app/flask-demo/
├── app.py
├── requirements.txt
├── Dockerfile
├── .dockerignore
└── tests/
    └── structure-test.yaml
```

---

# 🐳 Dockerfile final

## Dockerfile utilisé

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

RUN useradd -m appuser

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1

CMD ["python", "app.py"]
```

---

# 🧠 Explication des choix techniques

## Image de base

```dockerfile
python:3.12-slim
```

Utilisation d’une image légère basée sur Debian slim afin de :

* réduire la taille finale ;
* limiter la surface d’attaque ;
* accélérer les téléchargements et déploiements.

---

## Variables d’environnement Python

```dockerfile
PYTHONDONTWRITEBYTECODE=1
```

Empêche la création des fichiers `.pyc`.

```dockerfile
PYTHONUNBUFFERED=1
```

Permet d’afficher les logs Python immédiatement dans Docker.

---

## Utilisateur non-root

```dockerfile
USER appuser
```

Le conteneur n’est pas exécuté en root.

Objectifs :

* améliorer la sécurité ;
* limiter les impacts en cas de compromission ;
* suivre les bonnes pratiques conteneur.

---

## Healthcheck

```dockerfile
HEALTHCHECK
```

Le conteneur vérifie automatiquement l’endpoint :

```text
/health
```

Cela permettra plus tard :

* l’intégration Docker Compose ;
* le monitoring ;
* les futurs déploiements orchestrés.

---

# 🚫 Configuration du `.dockerignore`

## Objectif

Réduire le contexte Docker envoyé lors du build.

## Contenu utilisé

```text
.venv
__pycache__
*.pyc
.git
.gitignore
.vscode
.env
.pytest_cache
```

## Intérêt

Éviter d’envoyer :

* l’environnement virtuel Python ;
* les caches ;
* les fichiers Git ;
* les fichiers VSCode ;
* les secrets éventuels.

---

# 🏗️ Construction de l’image

## Build

```bash
docker build --no-cache -t flask-demo:1.0 .
```

## Vérification

```bash
docker images
```

Résultat attendu :

```text
flask-demo   1.0
```

---

# ▶️ Exécution du conteneur

## Lancement

```bash
docker run -d -p 5000:5000 --name flask-demo flask-demo:1.0
```

## Vérification

```bash
docker ps
```

État attendu :

```text
healthy
```

---

# 🌐 Tests de l’API

## Route principale

```bash
curl http://localhost:5000
```

## Healthcheck

```bash
curl http://localhost:5000/health
```

---

# 👤 Vérification utilisateur non-root

Commande :

```bash
docker run --rm flask-demo:1.0 id
```

Résultat attendu :

```text
uid=1000(appuser)
```

---

# 🔍 Analyse de l’image avec Dive

## Objectif

Analyser :

* les couches Docker ;
* l’efficacité de l’image ;
* les fichiers inutiles ;
* l’espace gaspillé.

## Commande utilisée

```bash
dive flask-demo:1.0
```

## Résultats obtenus

```text
Image size: 132 MB
Potential wasted space: 5.6 MB
Image efficiency score: 97%
```

## Interprétation

Le score de 97% indique :

* peu de fichiers inutiles ;
* peu de duplication ;
* image globalement optimisée.

---

# 🧪 Validation avec Container Structure Test

## Objectif

Valider automatiquement :

* les métadonnées ;
* l’utilisateur ;
* les fichiers présents ;
* les commandes ;
* la structure globale de l’image.

---

# 📄 Fichier `structure-test.yaml`

```yaml
schemaVersion: '2.0.0'

metadataTest:
  exposedPorts: ["5000"]
  workdir: "/app"
  user: "appuser"

commandTests:
  - name: "Python disponible"
    command: "python"
    args: ["--version"]
    exitCode: 0

  - name: "Flask importable"
    command: "python"
    args: ["-c", "import flask; print(flask.__version__)"]
    exitCode: 0

fileExistenceTests:
  - name: "Application Flask présente"
    path: "/app/app.py"
    shouldExist: true

  - name: "Requirements non présent dans runtime"
    path: "/app/requirements.txt"
    shouldExist: true
```

---

# ▶️ Exécution CST

```bash
container-structure-test test \
  --image flask-demo:1.0 \
  --config tests/structure-test.yaml
```

---

# ✅ Résultat obtenu

```text
Passes: 5
Failures: 0
PASS
```

## Tests validés

* Python installé ;
* Flask importable ;
* fichier app.py présent ;
* requirements.txt présent ;
* utilisateur image = appuser.

---

# 🔎 Analyse du Dockerfile avec Hadolint

## Objectif

Analyser le Dockerfile avant build afin de détecter :

* mauvaises pratiques ;
* problèmes de reproductibilité ;
* optimisations possibles ;
* risques sécurité.

---

# ▶️ Commande utilisée

```bash
hadolint Dockerfile
```

---

# ⚠️ Résultats obtenus

```text
DL3008 warning: Pin versions in apt get install
DL3015 info: Avoid additional packages by specifying --no-install-recommends
```

---

# ✅ Correction appliquée

Modification :

```dockerfile
apt-get install -y curl
```

vers :

```dockerfile
apt-get install -y --no-install-recommends curl
```

---

# ⚠️ Warning volontairement conservé

## DL3008

Le pinning strict des versions apt a été volontairement repoussé.

## Raison

Dans le cadre du laboratoire :

* priorité à la lisibilité ;
* priorité à l’apprentissage ;
* éviter une maintenance trop lourde des versions Debian.

Compromis accepté entre :

* reproductibilité ;
* simplicité ;
* pédagogie.

---

# 🛑 Incidents rencontrés

# Incident 1 — fichier CST introuvable

## Erreur

```text
open tests/structure-test.yaml: no such file or directory
```

## Cause

Nom de fichier incorrect :

```text
structures-tests.yaml
```

au lieu de :

```text
structure-test.yaml
```

## Correction

Renommage du fichier.

---

# Incident 2 — image exécutée en root

## Erreur

```text
Image user does not match config user: appuser
```

## Cause

L’image n’avait pas été rebuildée après ajout :

```dockerfile
USER appuser
```

## Correction

```bash
docker build --no-cache -t flask-demo:1.0 .
```

---

# ✅ Résultat final

L’image Docker Flask est désormais :

* construite ;
* fonctionnelle ;
* documentée ;
* analysée ;
* testée ;
* optimisée ;
* exécutée avec un utilisateur non-root.

---

# 🧭 Impact sur le laboratoire

Cette étape valide la transition :

```text
API Flask locale
→ image Docker propre
→ validation DevSecOps
→ futur déploiement Compose
→ futur reverse proxy Nginx
→ futur déploiement sur svr-main
```

Le poste Ubuntu Desktop devient maintenant :

```text
poste principal DevOps / build / validation
```

Tandis que `svr-main` deviendra :

```text
plateforme de déploiement et d’infrastructure
```
