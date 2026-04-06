# 🇫🇷 SYNTHÈSE — SUPERVISION NIVEAU 2 (NGINX + DNS)

---

## 🎯 Objectif

Mettre en place une supervision **réaliste et exploitable** de services :

* un service web (**Nginx**)
* un service réseau (**DNS**)

👉 Le but n’est pas seulement de “voir des métriques”, mais de :

> comprendre l’état réel des services et détecter des anomalies.

---

## 🧱 Architecture mise en place

```
Nginx → stub_status → exporter → Prometheus → Grafana
DNS → Blackbox exporter → Prometheus → Grafana
```

---

# 🌐 1. Supervision de Nginx

---

## 🎯 Principe

Nginx expose des métriques internes via :

```nginx
stub_status;
```

👉 Ces métriques sont récupérées par un exporter.

---

## ⚙️ Mise en place

### 1. Activer `stub_status` dans Nginx

```nginx
location /nginx_status {
    stub_status;
    allow 127.0.0.1;
    deny all;
}
```

👉 Accessible uniquement en local (sécurisé)

---

### 2. Installer l’exporter

```bash
sudo apt install prometheus-nginx-exporter
```

---

### 3. Configurer l’exporter

```bash
ARGS="--nginx.scrape-uri=http://127.0.0.1/nginx_status"
```

👉 Important : utiliser `127.0.0.1` (problème rencontré sinon)

---

### 4. Vérifier

```bash
curl http://localhost:9113/metrics
```

👉 Vérifier :

```text
nginx_up 1
```

---

## 🧠 Problème rencontré

### ❌ Symptôme

```text
nginx_up 0
```

### 🔍 Cause

* mauvaise URL (`srv1.lab.lan`)
* IP non autorisée → `403 Forbidden`

### ✅ Solution

* utiliser `127.0.0.1`
* cohérence avec `allow 127.0.0.1`

---

## 📊 Résultat

Dans Prometheus :

```
nginx → UP
```

Dans Grafana :

* connexions actives
* requêtes
* activité globale

---

# 🌍 2. Supervision DNS

---

## 🎯 Principe

Contrairement à Nginx :

❌ DNS ne fournit pas de métriques HTTP
✅ On utilise un test fonctionnel (probe)

---

## 🧰 Outil utilisé

👉 Blackbox Exporter

---

## ⚙️ Mise en place

### 1. Installation

```bash
sudo apt install prometheus-blackbox-exporter
```

---

### 2. Configuration

Fichier :

```bash
/etc/prometheus/blackbox.yml
```

Ajout :

```yaml
modules:
  dns_udp:
    prober: dns
    timeout: 5s
    dns:
      query_name: google.com
      query_type: A
```

---

### 3. Test manuel

```bash
curl "http://localhost:9115/probe?target=127.0.0.1:53&module=dns_udp"
```

Résultat attendu :

```text
probe_success 1
```

---

### 4. Intégration dans Prometheus

```yaml
- job_name: "dns"
  metrics_path: /probe
  params:
    module: [dns_udp]

  static_configs:
    - targets:
      - 127.0.0.1:53

  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - target_label: __address__
      replacement: 127.0.0.1:9115
```

---

## 📊 Résultat

Dans Prometheus :

```
dns → UP
```

---

## 🧠 Ce que mesure réellement DNS

* succès de la requête (`probe_success`)
* temps de réponse (`probe_duration_seconds`)
* temps de résolution DNS

---

# 📊 3. Visualisation Grafana

---

## 🎯 Bonne pratique appliquée

👉 **1 panel = 1 métrique**

---

## Panels créés

### 🟢 DNS Status

```promql
probe_success{job="dns"}
```

* UP / DOWN
* couleurs adaptées

---

### ⏱ DNS Response Time

```promql
probe_duration_seconds{job="dns"}
```

---

### 🔍 DNS Lookup Time

```promql
probe_dns_lookup_time_seconds
```

---

## 🧠 Erreur corrigée

❌ plusieurs métriques dans un seul panel
✅ séparation claire

---

# 🧠 Enseignements clés

---

## 1. Tous les services ne se supervisent pas pareil

| Type               | Exemple | Méthode  |
| ------------------ | ------- | -------- |
| métriques internes | Nginx   | exporter |
| test fonctionnel   | DNS     | blackbox |

---

## 2. “UP” ≠ “fonctionnel”

Un service peut :

* être actif
* mais ne pas répondre correctement

---

## 3. Importance de la lisibilité

> Un dashboard doit être compréhensible en quelques secondes.

---

## 4. Debug réel

Exemple :

* erreur 403
* mauvais endpoint
* mauvaise IP

👉 partie essentielle du projet

---

# 🧭 Conclusion

Cette étape permet de :

* comprendre deux types de supervision
* construire une chaîne complète (service → métrique → visualisation)
* développer un raisonnement d’exploitation

---

---

# 🇬🇧 SUMMARY — MONITORING LEVEL 2 (NGINX + DNS)

---

## 🎯 Objective

Build a **realistic and usable monitoring system** for:

* a web service (**Nginx**)
* a network service (**DNS**)

👉 The goal is not just to display metrics, but to:

> understand service health and detect issues.

---

## 🧱 Architecture

```
Nginx → stub_status → exporter → Prometheus → Grafana
DNS → Blackbox exporter → Prometheus → Grafana
```

---

# 🌐 1. Nginx Monitoring

---

## 🎯 Concept

Nginx exposes internal metrics using:

```nginx
stub_status;
```

These metrics are collected by an exporter.

---

## ⚙️ Setup

### 1. Enable stub_status

```nginx
location /nginx_status {
    stub_status;
    allow 127.0.0.1;
    deny all;
}
```

---

### 2. Install exporter

```bash
sudo apt install prometheus-nginx-exporter
```

---

### 3. Configure exporter

```bash
ARGS="--nginx.scrape-uri=http://127.0.0.1/nginx_status"
```

---

### 4. Verify

```bash
curl http://localhost:9113/metrics
```

Expected:

```text
nginx_up 1
```

---

## 🧠 Issue encountered

### ❌ Problem

```text
nginx_up 0
```

### 🔍 Cause

* wrong endpoint (`srv1.lab.lan`)
* access denied (403)

### ✅ Fix

* use `127.0.0.1`
* match Nginx access rules

---

## 📊 Result

In Prometheus:

```
nginx → UP
```

---

# 🌍 2. DNS Monitoring

---

## 🎯 Concept

Unlike Nginx:

❌ no HTTP metrics
✅ requires functional testing

---

## 🧰 Tool

👉 Blackbox Exporter

---

## ⚙️ Setup

### 1. Install

```bash
sudo apt install prometheus-blackbox-exporter
```

---

### 2. Configure

```yaml
modules:
  dns_udp:
    prober: dns
    timeout: 5s
    dns:
      query_name: google.com
      query_type: A
```

---

### 3. Test

```bash
curl "http://localhost:9115/probe?target=127.0.0.1:53&module=dns_udp"
```

Expected:

```text
probe_success 1
```

---

### 4. Prometheus integration

```yaml
- job_name: "dns"
  metrics_path: /probe
  params:
    module: [dns_udp]

  static_configs:
    - targets:
      - 127.0.0.1:53

  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - target_label: __address__
      replacement: 127.0.0.1:9115
```

---

## 📊 Result

```
dns → UP
```

---

## 🧠 What is measured

* request success (`probe_success`)
* response time
* DNS resolution time

---

# 📊 3. Grafana Visualization

---

## 🎯 Best practice

👉 **1 panel = 1 metric**

---

## Panels

### DNS Status

```promql
probe_success{job="dns"}
```

---

### DNS Response Time

```promql
probe_duration_seconds{job="dns"}
```

---

### DNS Lookup Time

```promql
probe_dns_lookup_time_seconds
```

---

## 🧠 Fix applied

❌ multiple metrics in one panel
✅ clean separation

---

# 🧠 Key learnings

---

## 1. Not all services are monitored the same way

| Type             | Example | Method   |
| ---------------- | ------- | -------- |
| internal metrics | Nginx   | exporter |
| functional test  | DNS     | blackbox |

---

## 2. “UP” ≠ usable

A service can be running but not functional.

---

## 3. Readability matters

> A dashboard must be understandable in a few seconds.

---

## 4. Real troubleshooting

* HTTP 403
* wrong endpoint
* access control issues

---

# 🧭 Conclusion

This phase allows you to:

* understand multiple monitoring approaches
* build a full monitoring pipeline
* develop operational reasoning skills
