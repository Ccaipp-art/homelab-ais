# 📘 **Documentation — Stabilisation Réseau & DNS + Intégration Client Linux (Version Anonyme)**

## 1. 🎯 Objectif de la session

Stabiliser l'infrastructure réseau du homelab et assurer :

1. Un plan d’adressage cohérent et fixe (réseau privé VMnetX)
2. Un serveur DNS Bind9 fonctionnel
3. Une résolution interne/externe sur toutes les machines
4. L’intégration d’un client Linux dans le DNS interne
5. La réparation de l’accès SSH par clé
6. Un contrôle complet via Ansible

Cette session corrige des incohérences réseau apparues lors des configurations précédentes.

---

# 2. 🔎 Contexte initial

Avant corrections :

* Le poste d’administration n’utilisait plus le bon réseau VMware
* Le serveur DNS possédait **deux IP** (statique + DHCP) → conflits réseau
* Le client Linux était configuré avec **une mauvaise IP**
* Bind9 renvoyait des erreurs **SERVFAIL**
* Le pare-feu UFW bloquait totalement le port 53
* L’accès SSH par clé refusait l’authentification
* Ansible n’était plus opérationnel sur le serveur principal

---

# 3. ⚙️ Actions réalisées

## 3.1. Réparation du réseau VMware / VMnetX

* Réactivation du service NAT VMware
* Réactivation du DHCP VMware
* Vérification de la configuration du réseau virtuel
* Reconfiguration du poste d’administration en IP statique :

```
IP : 192.168.X.10
Passerelle : 192.168.X.1
DNS : 192.168.X.20
```

Résultat :
➡️ Poste d’administration reconnecté à l’infrastructure.

---

## 3.2. Correction de la configuration réseau du serveur principal (Ubuntu Server)

Configuration Netplan ajustée :

```yaml
network:
  version: 2
  ethernets:
    ensX:
      dhcp4: false
      addresses:
        - 192.168.X.20/24
      routes:
        - to: default
          via: 192.168.X.1
      nameservers:
        addresses:
          - 192.168.X.20
        search:
          - lab.local
```

Application :

```
sudo netplan apply
```

Résultat :
➡️ Suppression de l’ancienne adresse DHCP
➡️ DNS capable d’écouter sur une IP stable

---

## 3.3. Ouverture des ports DNS dans UFW

```bash
sudo ufw allow in proto udp from 192.168.X.0/24 to any port 53 comment 'DNS-UDP'
sudo ufw allow in proto tcp from 192.168.X.0/24 to any port 53 comment 'DNS-TCP'
```

Résultat :
➡️ Bind9 accessible depuis tout le réseau du lab.

---

## 3.4. Tests de Bind9 (résolution interne + Internet)

### Résolution interne :

```bash
dig serveur.lab.local @192.168.X.20
```

### Résolution externe :

```bash
dig example.com @192.168.X.20
```

Résultat :
➡️ Réponses valides pour les zones internes et externes

---

## 3.5. Configuration réseau du client Linux (Rocky/Fedora/RHEL-like)

Ajustement de l’IP et du DNS :

```bash
sudo nmcli connection modify ensX ipv4.addresses "192.168.X.30/24"
sudo nmcli connection modify ensX ipv4.gateway "192.168.X.1"
sudo nmcli connection modify ensX ipv4.dns "192.168.X.20"
sudo nmcli connection modify ensX ipv4.dns-search "lab.local"
sudo nmcli connection modify ensX ipv4.ignore-auto-dns yes
```

Redémarrage de l’interface :

```
sudo nmcli connection down ensX && sudo nmcli connection up ensX
```

Résultat :
➡️ Résolution DNS interne + externe opérationnelle
➡️ Noms courts fonctionnels (`serveur` → `serveur.lab.local`)

---

## 3.6. Réparation SSH par clé sur le serveur principal

Erreur initiale :

```
Could not open ... authorized_keys
Permission denied (publickey)
```

Correction des permissions :

```bash
sudo chown -R user:user /home/user
sudo chmod 750 /home/user
sudo chmod 700 /home/user/.ssh
sudo chmod 600 /home/user/.ssh/authorized_keys
sudo systemctl restart ssh
```

Résultat :
➡️ Authentification SSH par clé restaurée

---

## 3.7. Intégration des machines dans Ansible

Inventory utilisé :

```ini
[ubuntu]
srv-main ansible_host=192.168.X.20 ansible_user=user

[client]
linux-client ansible_host=192.168.X.30 ansible_user=user
```

Tests réussis :

```bash
ansible ubuntu -i ansible/inventory.ini -m ping
ansible client -i ansible/inventory.ini -m ping
```

Résultat :

```
srv-main     | SUCCESS => {"ping": "pong"}
linux-client | SUCCESS => {"ping": "pong"}
```

---

# 4. 🔍 Résultats obtenus

* Réseau VMnetX stabilisé
* IP fixes appliquées aux trois machines
* Bind9 pleinement fonctionnel
* Client Linux ajouté au DNS interne
* SSH par clé réparé
* Ansible opérationnel sur l’ensemble des nœuds

L’infrastructure est désormais **cohérente, stable et prête pour la suite du lab**.

---

# 5. 🛑 Points de vigilance retenus

* Ne jamais laisser une VM avec **2 IP** sur la même interface
* Toujours corriger l’IP **avant** d’installer/configurer Bind9
* UFW doit explicitement autoriser le port 53
* Les permissions du dossier `.ssh` doivent être strictes
* Toujours tester avec :

  * `dig`
  * `getent hosts`
  * `ansible -m ping`

---

# 6. 📘 Fichiers mis à jour

* `/etc/netplan/*.yaml`
* `/etc/bind/named.conf.options`
* `~/ansible/inventory.ini`
* `~/.ssh/authorized_keys` (serveur principal)

---

# 7. 🧭 Prochaines étapes possibles

* Déploiement automatisé de services via Ansible
* Mise en place de la supervision (Node Exporter)
* Durcissement SSH / UFW via playbooks
* Installation de Nginx pour les premiers services web du lab
