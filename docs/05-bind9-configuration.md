# 📄 **DOCUMENT TECHNIQUE DÉTAILLÉ — 05-Configuration du service DNS Bind9**

### *Infrastructure Lab · Ubuntu Server 24.04 LTS*

*(Version prête pour Notion et GitHub · anonymisée)*

---

# 🧭 1. Introduction

Ce document décrit en détail la mise en place d’un **serveur DNS interne complet** basé sur Bind9 dans un laboratoire d’administration système Linux.
Ce serveur DNS constitue la **colonne vertébrale du réseau** et sera utilisé par :

* les serveurs internes du lab
* les clients Linux (Ubuntu & Rocky)
* les futurs services (Nginx, Prometheus, Grafana, Ansible…)

L’objectif est de reproduire une **infrastructure d’entreprise** avec :

* un domaine interne (`lab.lan`)
* une zone reverse cohérente
* un DNS récursif sécurisé
* une résolution interne fiable et testée

Toutes les IP du présent document sont **anonymisées** au format `192.168.10.X`.

---

# 🧱 2. Architecture et conception du service DNS

## 2.1 Objectifs techniques

* Fournir une résolution DNS interne centralisée
* Assurer une résolution inverse correcte
* Isoler la récursion au réseau LAN
* Protéger le DNS des requêtes externes
* Structurer proprement les fichiers Bind9
* Permettre l’utilisation de noms courts (`srv1`)
* Préparer l’intégration avec d’autres services (Nginx, supervision, Ansible)

## 2.2 Vue d’ensemble du système

| Élément               | Détail                                  |
| --------------------- | --------------------------------------- |
| Domaine interne       | `lab.lan`                               |
| Serveur DNS           | Ubuntu Server 24.04                     |
| Adresse IP            | `192.168.10.10`                         |
| Serveur DNS primaire  | `ns1.lab.lan`                           |
| Zone directe          | `/etc/bind/zones/db.lab.lan`            |
| Zone reverse          | `/etc/bind/zones/db.192.168.10`         |
| Zone reverse associée | `10.168.192.in-addr.arpa`               |
| Forwarders            | 1.1.1.1 (Cloudflare), 9.9.9.9 (Quad9)   |
| Clients               | Ubuntu Server minimal, Rocky Linux 9    |
| Utilisation           | Résolution interne + récursion Internet |

---

# ⚙️ 3. Installation et préparation du serveur DNS

## 3.1 Installation des paquets

```bash
sudo apt update
sudo apt install -y bind9 bind9-utils dnsutils
```

### Pourquoi ces paquets ?

* `bind9` : daemon dns
* `bind9-utils` : outils d’administration (`named-checkconf`, `named-checkzone`)
* `dnsutils` : outils de test (`dig`, `nslookup`, `host`)

---

# 📁 4. Configuration de Bind9

La configuration a été structurée pour respecter les bonnes pratiques :

* séparation des fichiers
* zones dans un dossier dédié
* options propres et sécurisées

---

## 4.1 Configuration globale : `/etc/bind/named.conf.options`

```conf
options {
    directory "/var/cache/bind";

    recursion yes;
    allow-recursion { 127.0.0.1; 192.168.10.0/24; };

    listen-on { 127.0.0.1; 192.168.10.10; };
    listen-on-v6 { none; };

    allow-query { 127.0.0.1; 192.168.10.0/24; };
    allow-transfer { none; };

    forwarders {
        1.1.1.1;
        9.9.9.9;
    };

    dnssec-validation yes;
    auth-nxdomain no;
};
```

### Explications :

* **recursion yes** : permet au DNS de résoudre Internet
* **allow-recursion** : limite la récursion au LAN
* **listen-on** : évite d’exposer le DNS à Internet
* **allow-transfer none** : empêche le vol de zone (sécurité fondamentale)
* **forwarders** : délégation aux DNS externes
* **disable IPv6** : simplification en environnement lab

---

## 4.2 Déclaration des zones : `/etc/bind/named.conf.local`

```conf
zone "lab.lan" {
    type master;
    file "/etc/bind/zones/db.lab.lan";
};

zone "10.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.192.168.10";
};
```

---

## 4.3 Zone directe : `/etc/bind/zones/db.lab.lan`

```conf
$TTL    86400
@       IN      SOA     ns1.lab.lan. admin.lab.lan. (
                        2025120701
                        3600
                        1800
                        1209600
                        86400 )

        IN      NS      ns1.lab.lan.

ns1     IN      A       192.168.10.10
srv1    IN      A       192.168.10.10
```

### Pourquoi ces entrées ?

* L'entrée `ns1` est **obligatoire** : c’est le serveur DNS lui-même
* `srv1` représente un serveur interne (par exemple pour Nginx)

---

## 4.4 Zone reverse : `/etc/bind/zones/db.192.168.10`

```conf
$TTL    86400
@       IN      SOA     ns1.lab.lan. admin.lab.lan. (
                        2025120701
                        3600
                        1800
                        1209600
                        86400 )

        IN      NS      ns1.lab.lan.

10      IN      PTR     srv1.lab.lan.
```

### Importance de la zone reverse :

* utilisée par plusieurs services (SSH, logs, monitoring…)
* cohérence réseau indispensable

---

# 🧪 5. Vérification de la configuration Bind9

## 5.1 Vérifications syntaxiques

```bash
sudo named-checkconf
named-checkzone lab.lan /etc/bind/zones/db.lab.lan
named-checkzone 10.168.192.in-addr.arpa /etc/bind/zones/db.192.168.10
```

## 5.2 Redémarrage propre

```bash
sudo systemctl restart bind9
sudo systemctl status bind9
```

---

# 🛠️ 6. Intégration au système (DNS local)

## 6.1 Configuration Netplan

```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: false
      addresses:
        - 192.168.10.10/24
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses:
          - 192.168.10.10
        search:
          - lab.lan
```

### Commandes d’application :

```bash
sudo netplan apply
sudo systemctl restart systemd-resolved
```

---

## 6.2 Vérification système : `resolvectl`

```
resolvectl status
resolvectl dns ens33
```

---

# 🧪 7. Tests fonctionnels

```
dig srv1.lab.lan
dig srv1.lab.lan.
dig -x 192.168.10.10
dig google.com
getent hosts srv1
ping srv1
```

### Résultats attendus :

* réponses immédiates
* aucun SERVFAIL
* résolution de noms courts via le search domain

---

# 🛑 8. Erreurs rencontrées et solutions

## Erreur 1 — Mauvaise orthographe dans les directives

```
allow-transfert
fowarders
```

✔️ Correction : directives correctes

---

## Erreur 2 — Chemin incorrect

```
file "etc/bind/zones/db.lab.lan";
```

✔️ Correction :

```
file "/etc/bind/zones/db.lab.lan";
```

---

## Erreur 3 — Zone non chargée

✔️ Solution : serial + syntaxe corrigée

---

## Erreur 4 — Résolution impossible pour `srv1`

✔️ Cause : dig ignore les search domains
✔️ Solution : utiliser `getent` / `ping`

---

# 🟢 9. Résultat final

Le serveur DNS :

* répond correctement
* supporte les noms courts / FQDN
* fournit la récursion
* permet une architecture cohérente du lab
* est prêt pour Nginx, monitoring et Ansible

C’est un **serveur DNS production-like** pour un environnement d’apprentissage professionnel.

---

# 🧭 10. Étapes suivantes

* Intégration du DNS dans les clients (Ubuntu et Rocky)
* Installation de Nginx + DNS appliqué
* Mise en place du monitoring (Node Exporter + Prometheus)
* Inventaire Ansible basé sur le DNS
