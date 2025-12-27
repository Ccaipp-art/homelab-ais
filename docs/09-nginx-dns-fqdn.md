# 📘 09 — Nginx : DNS réel, FQDN et cohérence d’infrastructure

## 🧭 Contexte

Lors de la session précédente, le service Nginx a été **automatisé via Ansible** et validé fonctionnellement.

Cependant, les tests étaient encore réalisés **par adresse IP**, ce qui ne reflète pas un usage réaliste en environnement professionnel.

Dans une infrastructure réelle, un service web :

* est accédé par **nom DNS**,
* dépend directement du **serveur DNS interne**,
* doit rester cohérent entre :

  * résolution de nom,
  * configuration Nginx,
  * certificats TLS.

Cette session vise donc à **intégrer le service Nginx existant dans l’infrastructure DNS**, sans redéployer l’application ni modifier l’architecture Ansible.

---

## 🎯 Objectifs de la session

* Mettre en place un **FQDN interne** pour le service web
* Configurer Bind9 pour résoudre ce nom
* Vérifier la résolution DNS côté serveur et côté client
* Adapter la configuration existante de Nginx pour répondre à ce FQDN
* Valider le comportement HTTP / HTTPS par nom DNS
* Comprendre l’impact du DNS sur un service web

---

## 🧩 Prérequis

* Serveur DNS **Bind9** fonctionnel
* Poste d’administration utilisant Bind9 comme résolveur
* Service Nginx déjà déployé et automatisé
* Accès SSH aux machines du laboratoire

> Tous les noms de domaine, adresses IP et noms de machines sont volontairement **anonymisés**.

---

## 🏷️ Choix du FQDN

Le nom retenu pour le service web est :

```text
web.lab.lan
```

Ce choix permet :

* d’identifier clairement le rôle du service (`web`)
* de s’inscrire dans un domaine interne (`lab.lan`)
* de préparer l’extension future de l’infrastructure

👉 Un FQDN représente un **service**, pas une machine.

---

## 🌐 Intégration DNS (Bind9)

### Ajout de l’enregistrement DNS

Dans la zone DNS interne `lab.lan`, un nouvel enregistrement est ajouté :

```dns
web IN A <IP_DU_SERVEUR_NGINX>
```

Cette entrée permet d’associer le nom `web.lab.lan` au serveur hébergeant Nginx.

---

### Gestion du serial SOA

À chaque modification de la zone DNS, le **serial SOA est incrémenté** afin de garantir la propagation des changements.

Le format utilisé est :

```text
YYYYMMDDNN
```

---

### Validation de la zone

```bash
named-checkzone lab.lan <fichier_de_zone>
```

Puis rechargement du service DNS :

```bash
systemctl reload bind9
```

---

## 🔍 Validation de la résolution DNS

### Côté serveur DNS

```bash
dig web.lab.lan @127.0.0.1
```

### Côté client

```bash
getent hosts web.lab.lan
```

Résultat attendu :

```text
<IP_DU_SERVEUR_NGINX> web.lab.lan
```

Aucune modification de `/etc/hosts` n’est utilisée.

---

## ⚙️ Adaptation de la configuration Nginx existante

### Utilisation d’un FQDN réel

La configuration Nginx existante est ajustée pour répondre explicitement au FQDN :

```nginx
server_name web.lab.lan;
```

Le `server_name` générique (`_`) est volontairement abandonné afin de refléter un usage réel.

---

### Certificat TLS cohérent

Le certificat TLS auto-signé déjà en place est vérifié afin de correspondre au FQDN utilisé :

```text
CN=web.lab.lan
```

👉 DNS, Nginx et TLS doivent être cohérents pour garantir un comportement HTTPS correct.

---

## 🧪 Vérifications fonctionnelles

### Accès HTTPS via FQDN

```bash
curl -kI https://web.lab.lan
```

Résultat attendu :

```text
HTTP/1.1 200 OK
```

---

### Redirection HTTP → HTTPS

```bash
curl -I http://web.lab.lan
```

Résultat attendu :

```text
HTTP/1.1 301 Moved Permanently
Location: https://web.lab.lan/
```

---

### Headers de sécurité

```bash
curl -kI https://web.lab.lan
```

Présence attendue :

* X-Frame-Options
* X-Content-Type-Options
* Referrer-Policy
* Permissions-Policy

---

## 🛑 Enseignements clés

### Tester par IP est trompeur

Un service accessible par IP peut masquer :

* des erreurs de configuration DNS
* un mauvais `server_name`
* un certificat TLS incohérent

👉 Les tests doivent toujours être réalisés **par nom DNS**.

---

### Le DNS est une brique centrale

Sans DNS fonctionnel :

* le service web devient inutilisable
* les tests HTTPS perdent leur sens
* les erreurs sont difficiles à diagnostiquer

---

## 📝 Points clés à retenir

* Le DNS conditionne l’accès aux services
* Un FQDN est indispensable pour tester HTTPS correctement
* DNS, Nginx et TLS doivent être alignés
* Cette session **intègre** un service existant, elle ne le redéploie pas

---

## 🧭 Impact sur le laboratoire

Cette session permet de passer :

* d’un service web fonctionnel isolé
* à un **service intégré à l’infrastructure DNS**

Elle renforce la cohérence globale du laboratoire et prépare les étapes suivantes :

* sécurisation réseau
* supervision
* certificats réels
* exposition contrôlée des services

