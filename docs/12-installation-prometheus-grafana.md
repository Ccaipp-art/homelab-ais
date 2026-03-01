# 12 – Monitoring Installation (Prometheus, Node Exporter & Grafana)

---

# 🇫🇷 VERSION FRANÇAISE

---

## 🎯 Objectif

Mettre en place une supervision système basique sur un serveur Ubuntu 24.04 afin de surveiller :

* CPU
* Mémoire
* Disque
* Réseau
* Uptime

La stack utilisée :

* Node Exporter
* Prometheus
* Grafana

---

## 🧱 Architecture

```
Node Exporter (port 9100)
        ↓
Prometheus (port 9090)
        ↓
Grafana (port 3000)
```

* Node Exporter expose les métriques système.
* Prometheus collecte et stocke les métriques.
* Grafana visualise les données.

---

## 🖥️ Environnement

* Ubuntu Server 24.04 LTS
* UFW activé
* Accès SSH configuré
* Réseau local (NAT ou bridge)

---

# 🔹 Étape 1 – Installation de Node Exporter

### Installation

```bash
sudo apt update
sudo apt install -y prometheus-node-exporter
```

### Vérification

```bash
systemctl status prometheus-node-exporter
ss -tulpen | grep 9100
```

Le service doit être :

```
active (running)
```

### Test navigateur

```
http://SERVER_IP:9100/metrics
```

Résultat attendu : affichage de métriques en texte brut.

---

# 🔹 Étape 2 – Installation de Prometheus

### Installation

```bash
sudo apt install -y prometheus
```

### Vérification

```bash
systemctl status prometheus
ss -tulpen | grep 9090
```

---

## 🔧 Configuration minimale

Éditer :

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Configuration utilisée :

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node"
    static_configs:
      - targets: ["localhost:9100"]
```

Redémarrer :

```bash
sudo systemctl restart prometheus
```

---

## 🔍 Validation

Accéder à :

```
http://SERVER_IP:9090
```

Puis :

Status → Targets

Les deux cibles doivent être "UP".

---

## 🔐 Configuration Firewall

```bash
sudo ufw allow 9090/tcp
sudo ufw allow 9100/tcp
```

---

# 🔹 Étape 3 – Installation de Grafana

Grafana n’est pas disponible dans les dépôts Ubuntu par défaut.
On ajoute le dépôt officiel.

---

## 📦 Ajout du dépôt officiel

```bash
sudo apt install -y apt-transport-https software-properties-common wget
sudo mkdir -p /etc/apt/keyrings
wget -q -O - https://apt.grafana.com/gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/grafana.gpg
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
```

---

## 📥 Installation

```bash
sudo apt install -y grafana
```

---

## ▶ Activation

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

Vérification :

```bash
systemctl status grafana-server
ss -tulpen | grep 3000
```

---

## 🔐 Ouverture Firewall

```bash
sudo ufw allow 3000/tcp
```

---

# 🔹 Configuration Grafana

### Accès

```
http://SERVER_IP:3000
```

Login par défaut :

```
admin / admin
```

Changer le mot de passe.

---

## 🔹 Ajouter Prometheus comme Data Source

Settings → Data Sources → Add Data Source → Prometheus

URL :

```
http://localhost:9090
```

Cliquer sur :

Save & Test

---

## 🔹 Importer un Dashboard

Dashboard ID utilisé :

```
1860
```

Dashboard : Node Exporter Full

---

# 🧠 Compréhension technique

### Node Exporter

* Expose les métriques
* Ne stocke aucune donnée
* Fonctionne sur le port 9100

### Prometheus

* Scrape les métriques toutes les 15 secondes
* Stocke les données en base Time Series
* Fonctionne sur le port 9090

### Grafana

* Interroge Prometheus
* Ne stocke pas les données
* Visualise les métriques
* Fonctionne sur le port 3000

---

# 🛠 Troubleshooting

### Prometheus inaccessible

Vérifier :

```
systemctl status prometheus
sudo ufw status
```

### Port bloqué

```
sudo ufw allow PORT/tcp
```

### Target DOWN

Vérifier que Node Exporter fonctionne :

```
systemctl status prometheus-node-exporter
```

---

# 🎤 Points d’entretien

* Différence entre métriques et logs
* Différence entre CPU Usage et Load Average
* Pourquoi utiliser une base Time-Series
* Pourquoi Grafana ne stocke pas les données
* Importance de la gestion des dépôts officiels

---

# 🇬🇧 COMPLETE ENGLISH VERSION (FULL & DETAILED)

---

# Monitoring Installation – Prometheus, Node Exporter & Grafana

---

## 🎯 Objective

Deploy a basic monitoring stack on Ubuntu Server 24.04 to observe and analyze:

* CPU usage
* Load average
* Memory usage
* Disk utilization
* Network traffic
* System uptime

This monitoring stack is designed to provide visibility into the health and performance of the infrastructure.

Stack components:

* **Node Exporter** → exposes system metrics
* **Prometheus** → collects and stores metrics
* **Grafana** → visualizes metrics

---

## 🧱 Architecture Overview

```text
Node Exporter (port 9100)
        ↓
Prometheus (port 9090)
        ↓
Grafana (port 3000)
```

### Component Roles

**Node Exporter**

* Exposes system-level metrics
* Does not store any data
* Runs on port 9100

**Prometheus**

* Scrapes metrics at regular intervals (15 seconds)
* Stores metrics in a time-series database (TSDB)
* Runs on port 9090

**Grafana**

* Queries Prometheus
* Displays metrics in dashboards
* Does not store raw metrics
* Runs on port 3000

---

## 🖥️ Environment

* Ubuntu Server 24.04 LTS
* UFW firewall enabled
* SSH configured
* Local network environment (NAT or bridged)

---

# Step 1 – Install Node Exporter

### Installation

```bash
sudo apt update
sudo apt install -y prometheus-node-exporter
```

### Verification

```bash
systemctl status prometheus-node-exporter
ss -tulpen | grep 9100
```

Expected output:

```
active (running)
```

### Test via Browser

Open:

```
http://SERVER_IP:9100/metrics
```

Expected result:

* Raw text metrics displayed
* Lines beginning with `# HELP` and `# TYPE`

This confirms that Node Exporter is correctly exposing system metrics.

---

# Step 2 – Install Prometheus

### Installation

```bash
sudo apt install -y prometheus
```

### Verify Service

```bash
systemctl status prometheus
ss -tulpen | grep 9090
```

---

## Configuration

Edit the configuration file:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Minimal configuration used:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node"
    static_configs:
      - targets: ["localhost:9100"]
```

Restart Prometheus:

```bash
sudo systemctl restart prometheus
```

---

## Validation

Open in browser:

```
http://SERVER_IP:9090
```

Navigate to:

```
Status → Targets
```

Both targets must be:

```
UP
```

---

## Firewall Configuration

Allow required ports:

```bash
sudo ufw allow 9090/tcp
sudo ufw allow 9100/tcp
```

---

# Step 3 – Install Grafana

Grafana is not available in default Ubuntu repositories.
We add the official repository to ensure proper updates and stability.

---

## Add Official Repository

```bash
sudo apt install -y apt-transport-https software-properties-common wget
sudo mkdir -p /etc/apt/keyrings
wget -q -O - https://apt.grafana.com/gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/grafana.gpg
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
```

---

## Install Grafana

```bash
sudo apt install -y grafana
```

---

## Enable and Start Service

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

Verify:

```bash
systemctl status grafana-server
ss -tulpen | grep 3000
```

---

## Firewall Rule

```bash
sudo ufw allow 3000/tcp
```

---

# Configure Grafana

Access:

```
http://SERVER_IP:3000
```

Default credentials:

```
admin / admin
```

Change password immediately after first login.

---

## Add Prometheus as Data Source

1. Go to **Settings**
2. Select **Data Sources**
3. Add **Prometheus**
4. URL:

```
http://localhost:9090
```

5. Click **Save & Test**

Expected result:

```
Data source is working
```

---

## Import System Dashboard

Dashboard ID used:

```
1860
```

This dashboard provides:

* CPU usage visualization
* Load average monitoring
* Memory usage analysis
* Disk space monitoring
* Network traffic monitoring
* Uptime tracking

---

# 🧠 Technical Understanding

### CPU Usage vs Load Average

* CPU Usage represents how busy the CPU is (percentage).
* Load Average represents how many processes are waiting for CPU time.

Load average should roughly match the number of CPU cores.

---

### Why Use a Time-Series Database?

Prometheus stores metrics as time-series data:

* Each metric is timestamped.
* Enables historical analysis.
* Enables trend detection.

---

### Why Grafana Does Not Store Data

Grafana is a visualization layer.
It queries Prometheus and renders dashboards.
Prometheus remains the single source of truth.

---

# Troubleshooting

### Prometheus Not Accessible

```bash
systemctl status prometheus
sudo ufw status
```

### Target DOWN

```bash
systemctl status prometheus-node-exporter
```

### Port Blocked

```bash
sudo ufw allow PORT/tcp
```

---

# Interview Talking Points

* Difference between metrics and logs
* Difference between CPU usage and load average
* Why time-series databases are important
* Why visualization layers should not store raw data
* Importance of repository management in production environments

---

# Status

Monitoring Level 1 – Completed
