# 04 — Stabilisation Réseau & Préparation au Déploiement DNS

> ⚠️ Note : Toutes les adresses IP utilisées dans ce document sont **fictives** et ne correspondent pas à la configuration réelle du laboratoire.  
> Elles sont fournies uniquement à titre d’exemple.

## 1. 🎯 Objectif

Stabiliser la configuration réseau du laboratoire avant l’installation du service DNS (Bind9).  
L’objectif principal est d’obtenir un adressage cohérent, prévisible et adapté aux services d’infrastructure.

---

## 2. 🔎 Contexte

- Le laboratoire repose sur plusieurs VMs sous VMware.
- Le serveur principal doit fonctionner avec une adresse IP **statique**.
- Le DHCP interne de VMware attribuait une seconde IP non souhaitée.
- Le poste d’administration devait être configuré en **mode Bridged** pour obtenir une IP du réseau local physique (LAN).

---

## 3. ⚙️ Actions réalisées

### 3.1 Suppression de l’attribution DHCP non souhaitée

- Désactivation du service `VMware DHCP Service`.
- Suppression de l’adresse IP dynamique résiduelle.
- Le serveur conserve désormais une seule IP statique, par exemple :

```

Serveur infrastructure : 192.168.50.10/24 (exemple)

````

Commandes utilisées :
```bash
sudo ip addr flush dev <interface>
sudo systemctl restart systemd-networkd
sudo netplan apply
````

---

### 3.2 Correction de la configuration Netplan (serveur)

* Identification correcte de l’interface réseau (`ens33`, `enp0s3`, `eth0`, etc.).
* Migration vers une syntaxe Netplan moderne (remplacement de `gateway4:` par `routes:`).
* Exemple de configuration fictive :

```yaml
network:
  version: 2
  ethernets:
    <interface>:
      dhcp4: no
      addresses:
        - 192.168.50.10/24        # IP statique fictive
      routes:
        - to: default
          via: 192.168.50.1       # Passerelle fictive
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
```

Vérifications :

```bash
ip a
ip route
```

---

### 3.3 Reconfiguration du poste d’administration (Bridged)

* Passage du mode NAT vers **Bridged**.
* Nettoyage des adaptateurs bridgés :

  * Sélection du **seul adaptateur physique réel**.
  * Exclusion des interfaces virtuelles inutiles (ex. Wi-Fi Direct virtuel, VirtualBox host-only).
* Le poste d’administration obtient maintenant une IP du réseau local physique, par exemple :

```
Poste d’administration : 192.168.1.42/24 (exemple)
```

Forçage du DHCP si nécessaire :

```bash
sudo dhclient -v <interface>
```

---

## 4. 🧩 Architecture réseau finale (IP fictives)

```
Réseau local physique (DHCP)
        │
        ├── Poste d’administration (bridged) → 192.168.1.42
        │
Réseau NAT VMware (isolé)
        ├── Serveur infrastructure → 192.168.50.10
        └── Machine cliente (prévue) → 192.168.50.20
```

* Le poste admin est indépendant du NAT VMware.
* Le serveur d’infrastructure possède une IP statique stable.
* Le réseau du lab est maintenant prévisible et maîtrisé.

---

## 5. 🔍 Vérifications réalisées

* Une seule IP statique active sur le serveur.
* Aucune IP secondaire injectée par VMware.
* Le poste d’administration reçoit correctement son IP LAN.
* Les routes réseau sont propres et stables.
* Tests réussis : `ip a`, `ip route`, `ping`, résolution DNS externe.

---

## 6. 🛠 Problèmes rencontrés & résolutions

| Problème                                 | Cause                          | Solution                                        |
| ---------------------------------------- | ------------------------------ | ----------------------------------------------- |
| IP secondaire indésirable sur le serveur | DHCP VMware actif              | Désactivation du service DHCP + reset interface |
| Mauvais nom d’interface dans Netplan     | Erreur de détection initiale   | Identification via `ip a` puis mise à jour      |
| Syntaxe Netplan dépréciée                | Ancienne directive `gateway4:` | Migration vers `routes:`                        |
| Pas d’IP en mode bridged                 | Mauvais adaptateur sélectionné | Sélection du bon adaptateur physique            |
| DHCP non reçu sur Desktop                | Requête non transmise          | `dhclient` + correction bridging                |

---

## 7. 🧭 Impact pour la suite

Grâce à la stabilisation réseau :

* Bind9 peut être installé proprement.
* Les zones DNS (directes et inverses) pourront être définies sur des IP stables.
* Le monitoring (Prometheus / Grafana) et Ansible pourront utiliser des FQDN fiables.
* L’architecture du lab reflète une structure professionnelle :

```
Poste admin → LAN
Infrastructure → NAT avec IP statiques
```

---

## 8. 📌 Notes

* Toutes les IP de ce document sont **entièrement fictives** pour garantir la confidentialité.
* Aucun élément sensible ou spécifique à l’environnement réel n’est exposé.
