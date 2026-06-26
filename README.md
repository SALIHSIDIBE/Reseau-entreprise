# 🌐 Conception, Configuration et Gestion d'un Réseau Local et Étendu

> **Rapport de Fin d'Études** — Spécialité : Réseaux Informatiques & Télécommunications  
> Université de l'Unité Africaine (UUA) — Année académique 2022/2023  
> **Auteur : SIDIBE Sâlih**

[![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)](https://github.com/sidiama/reseau-entreprise-cpt)
[![VLAN](https://img.shields.io/badge/VLANs-4%20segments-brightgreen?style=for-the-badge)](https://github.com/sidiama/reseau-entreprise-cpt)
[![OSPF](https://img.shields.io/badge/Routage-OSPF-orange?style=for-the-badge)](https://github.com/sidiama/reseau-entreprise-cpt)
[![SSH](https://img.shields.io/badge/Sécurité-SSH%20%2B%20ACL-red?style=for-the-badge)](https://github.com/sidiama/reseau-entreprise-cpt)

---

## 🎯 Objectif du projet

Ce projet a pour objectif de **concevoir, configurer et gérer** un réseau local (LAN) et étendu (WAN) pour une entreprise fictive **TechNova**, composée de deux sites :

| Site | Rôle |
|---|---|
| **Siège principal** | Administration, Comptabilité, RH, IT |
| **Filiale distante** | Administration, Comptabilité, RH, IT |

Les configurations ont été entièrement réalisées et simulées sur **Cisco Packet Tracer**.

---

## 🏗️ Topologie réseau

```
╔══════════════════════════════════╗         ╔══════════════════════════════════╗
║        SIÈGE PRINCIPAL           ║         ║        FILIALE DISTANTE          ║
║                                  ║         ║                                  ║
║  [Server0]      [Routeur siège]  ║──WAN────║  [Routeur filiale]  [Server1]    ║
║  DHCP/DNS       192.168.5.1      ║10.0.0.x ║  192.168.6.1        DHCP/DNS     ║
║                       │          ║         ║        │                         ║
║                   [Switch0]      ║         ║    [Switch1]                     ║
║                  Trunk VLAN      ║         ║   Trunk VLAN                     ║
║                 /   |   |   \    ║         ║  /    |    |    \                ║
║              V10   V20  V30  V40 ║         ║ V10  V20  V30   V40              ║
║             Admin Compt  RH   IT ║         ║Admin Compt RH    IT              ║
║          .1.x  .2.x .3.x .4.x    ║         ║ .10.x .20.x .30.x .40.x          ║
║                                  ║         ║                                  ║
║           [Access Point]         ║         ║        [Access Point]            ║
║           Wi-Fi WPA2             ║         ║        Wi-Fi WPA2                ║
║           [Smartphone0]          ║         ║        [Smartphone2]             ║
║           [PC test VLAN30]       ║         ║                                  ║
╚══════════════════════════════════╝         ╚══════════════════════════════════╝

                    **Liaison WAN point à point :** `10.0.0.0/30`  
**Routeur Siège :** `10.0.0.1` | **Routeur Filiale :** `10.0.0.2`

```

---

## 🛠️ Équipements utilisés

| Équipement | Modèle Cisco | Rôle | Prix unitaire |
|---|---|---|---|
| Routeurs | Cisco 4321 ISR | Routage LAN/WAN, NAT, OSPF, SSH | ~1 500 € |
| Commutateurs | Cisco Catalyst 2960 (IOS 15) | Gestion VLANs, Trunking | ~300 € |
| Points d'accès | AP-PT (Aironet 1800 Series) | Wi-Fi WPA2-PSK | ~200 € |
| Serveurs | Server-PT | DHCP + DNS | — |

---

## 📋 Plan d'adressage IP

### Routeur 1 — Siège principal

| Interface / VLAN | Réseau / Adresse IP | Détails |
|---|---|---|
| GigabitEthernet0/0/0 (LAN) | 192.168.5.1 /24 | Interface LAN du siège |
| Serial0/1/0 (WAN) | 10.0.0.1 /30 | Liaison WAN vers la filiale |
| VLAN 10 – Administration | 192.168.1.0 /24 | Passerelle : 192.168.1.1 — PC1 : 192.168.1.2 |
| VLAN 20 – Comptabilité | 192.168.2.0 /24 | Passerelle : 192.168.2.1 — PC1 : 192.168.2.2 |
| VLAN 30 – RH | 192.168.3.0 /24 | Passerelle : 192.168.3.1 — PC1 : 192.168.3.2 |
| VLAN 40 – IT | 192.168.4.0 /24 | Passerelle : 192.168.4.1 — PC1 : 192.168.4.2 |

### Routeur 2 — Filiale distante

| Interface / VLAN | Réseau / Adresse IP | Détails |
|---|---|---|
| GigabitEthernet0/0/0 (LAN) | 192.168.6.1 /24 | Interface LAN de la filiale |
| Serial0/1/0 (WAN) | 10.0.0.2 /30 | Liaison WAN vers le siège |
| VLAN 10 – Administration | 192.168.10.0 /24 | Passerelle : 192.168.10.1 — PC1 : 192.168.10.2 |
| VLAN 20 – Comptabilité | 192.168.20.0 /24 | Passerelle : 192.168.20.1 — PC1 : 192.168.20.2 |
| VLAN 30 – RH | 192.168.30.0 /24 | Passerelle : 192.168.30.1 — PC1 : 192.168.30.2 |
| VLAN 40 – IT | 192.168.40.0 /24 | Passerelle : 192.168.40.1 — PC1 : 192.168.40.2 |

---

## ⚙️ Configurations réalisées

### 1️⃣ Configuration des interfaces (Routeurs)

Configuration des interfaces LAN (`GigabitEthernet0/0/0`) et WAN (`Serial0/1/0`) sur les deux routeurs.  
Mise en place du **NAT** sur les deux routeurs pour permettre l'accès Internet aux appareils du LAN.

**NAT — Routeur Siège :**
```bash
interface GigabitEthernet0/0/0
 ip address 192.168.5.1 255.255.255.0
 ip nat inside

interface Serial0/1/0
 ip address 10.0.0.1 255.0.0.0
 ip nat outside
```

---

### 2️⃣ Configuration des VLANs

Création des 4 VLANs sur chaque switch via **VLAN DATABASE** :

| VLAN | Nom | Département |
|---|---|---|
| VLAN 10 | Administration | Direction générale |
| VLAN 20 | Comptabilité | Service financier |
| VLAN 30 | RH | Ressources Humaines |
| VLAN 40 | IT | Informatique |

Affectation des ports en **mode access** (ex. FastEthernet0/1 → VLAN 10) et configuration du **trunking** sur les ports connectés aux routeurs (`GigabitEthernet0/1`).

**Sous-interfaces et encapsulation dot1Q sur les routeurs :**
```bash
interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.1.1 255.255.255.0

interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.2.1 255.255.255.0

interface GigabitEthernet0/0/0.30
 encapsulation dot1Q 30
 ip address 192.168.3.1 255.255.255.0

interface GigabitEthernet0/0/0.40
 encapsulation dot1Q 40
 ip address 192.168.4.1 255.255.255.0
```

---

### 3️⃣ Routage

**Routage statique** (base de validation) :
```bash
# Routeur Siège
ip route 192.168.6.0 255.255.255.0 10.0.0.2

# Routeur Filiale
ip route 192.168.5.0 255.255.255.0 10.0.0.1
```

**Routage dynamique OSPF** (choisi pour ses avantages vs RIP) :

| Critère | OSPF | RIP |
|---|---|---|
| Évolutivité | ✅ Grands réseaux | ❌ Limité à 15 sauts |
| Convergence | ✅ Rapide | ❌ Lente |
| Support VLSM | ✅ Oui | ❌ Non (RIP v1) |

```bash
# Configuration OSPF — Routeur Siège
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.2.0 0.0.0.255 area 0
 network 192.168.3.0 0.0.0.255 area 0
 network 192.168.4.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
```

---

### 4️⃣ Services réseau

**DHCP** — Pools configurés sur les serveurs pour chaque VLAN :

| Pool | Passerelle | Adresse de départ | Masque |
|---|---|---|---|
| VLAN10 | 192.168.1.1 | 192.168.1.2 | 255.255.255.0 |
| VLAN20 | 192.168.2.1 | 192.168.2.2 | 255.255.255.0 |
| VLAN30 | 192.168.3.1 | 192.168.3.2 | 255.255.255.0 |
| VLAN40 | 192.168.4.1 | 192.168.4.2 | 255.255.255.0 |

**DNS** — Résolution de `www.technova.com` → `192.168.50.100`

**ACL** — Restriction d'accès à la filiale : seul le VLAN 10 (Administration) du siège est autorisé :
```bash
access-list 100 permit ip 192.168.1.0 0.0.0.255 any
access-list 100 deny ip any any
```

---

### 5️⃣ Sécurité — SSH

Activation de SSH et désactivation de Telnet sur les deux routeurs :
```bash
hostname Routeur_Siege
ip domain-name technova.com
crypto key generate rsa
username admin privilege 15 secret motdepasse
line vty 0 4
 login local
 transport input ssh
```

---

## 🧪 Simulations et Tests

### Résultats des tests de connectivité

| Test | Source | Destination | Résultat |
|---|---|---|---|
| Intra-VLAN siège | PC0 (VLAN 10) | PC1 (VLAN 20) | ⚠️ 100% loss |
| Inter-VLAN filiale | PC4 (VLAN 10) | PC5 (VLAN 20) | ⚠️ 25% loss |
| Inter-VLAN filiale | PC4 (VLAN 10) | PC6 (VLAN 30) | ❌ 100% loss |
| PC4 vers Routeur filiale | PC4 (VLAN 10) | 192.168.6.1 | ✅ 0% loss |
| Inter-sites (routeurs) | Routeur Siège | 10.0.0.2 | ❌ 0% success |
| Inter-sites (PC) | PC VLAN 10 siège | PC VLAN 10 filiale | ❌ 100% loss |
| DHCP VLAN 30 | PC test | Server DHCP | ❌ Adresse APIPA |

### Analyse des contraintes

La plupart des tests inter-VLAN et inter-sites ont échoué. Les causes identifiées :
- Erreurs de configuration des interfaces trunk entre switches et routeurs
- Problèmes de configuration des sous-interfaces dot1Q
- Configuration du DHCP Relay Agent manquante

Ces difficultés constituent une **base d'apprentissage essentielle** pour progresser en administration réseau.

---

## 📁 Structure du repository

```
reseau-entreprise-cpt/
│
├── README.md                         
│
└── rapport/
      └── Projet_Reseaux_TechNova.pdf   ← Rapport complet (31 pages)
```

---

## 📚 Sigles et abréviations

| Sigle | Signification |
|---|---|
| ACL | Access Control List |
| DHCP | Dynamic Host Configuration Protocol |
| DNS | Domain Name System |
| NAT | Network Address Translation |
| OSPF | Open Shortest Path First |
| SSH | Secure Shell |
| VLAN | Virtual Local Area Network |
| VLSM | Variable Length Subnet Masking |
| WAN | Wide Area Network |

---

## 👨‍💻 Auteur

**SIDIBE Sâlih**  

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat-square&logo=linkedin&logoColor=white)](https://linkedin.com/in/salih-sidibe)
[![GitHub](https://img.shields.io/badge/GitHub-000?style=flat-square&logo=github&logoColor=white)](https://github.com/sidiama)

---

> 📅 Projet réalisé dans le cadre du rapport de fin d'études — Licence Réseaux & Télécommunications — UUA Ouagadougou — 2022/2023
