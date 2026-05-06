# RP-01 — Audit et Sécurité Réseau

**Réalisation Professionnelle — BTS SIO SISR**
**ANDREO Vincent — IRIS Nice — 2026**

---

## Contexte et objectifs

Dans le cadre du BTS SIO option SISR (Épreuve E5), cette réalisation professionnelle porte sur l'**audit complet de la sécurité d'une infrastructure réseau virtualisée** hébergée sur un hyperviseur KVM/QEMU sous Debian 12.

L'objectif était d'identifier les vulnérabilités, de déployer une architecture sécurisée (DMZ + pare-feu nftables), de durcir les hôtes, puis de valider l'ensemble par des tests d'intrusion. Cette RP inclut également la **conception** de la migration AD → OpenLDAP Linux (802.1X RADIUS sur switches Cisco Catalyst 2960-S + AP C9105AXI-E).

---

## Architecture déployée

![Architecture réseau](assets/network_architecture.svg)
---

## Déroulement en 4 phases

### Phase 1 — Audit initial (Semaine 1)

- Découverte réseau complète avec **Nmap** (scan SYN, UDP, scripts NSE)
- Scan de vulnérabilités avec **OpenVAS** sur tous les hôtes
- **38 CVE identifiées** dont 8 critiques (CVSS ≥ 9.0)
- Rapport d'audit complet avec priorisation des risques (CVSS)

### Phase 2 — Déploiement nftables + DMZ (Semaine 2)

- Installation et configuration de **nftables** sur Debian 12 (pare-feu Linux kernel netfilter)
- Création de la **DMZ** (zone démilitarisée) avec interfaces eth0/eth1/eth2
- Politique par défaut : **deny-all** (drop toutes les connexions non autorisées)
- Règles NAT, port forwarding contrôlé, logging des paquets bloqués
- Fichier de configuration : `/etc/nftables.conf`

### Phase 3 — Durcissement CIS Benchmark (Semaine 3)

- Application des recommandations **CIS Benchmark Debian Linux**
- Configuration SSH : port non standard, désactivation root, authentification par clé uniquement
- Déploiement **Suricata** (IDS/IPS multi-thread) avec règles ET/Open
- Déploiement **CrowdSec** avec collections : `crowdsecurity/linux`, `crowdsecurity/sshd`, `crowdsecurity/nginx`
- Déploiement **WireGuard** (wg-easy) — VPN managé via interface web (port 51820/UDP, UI 51821/TCP)
- Score **Lynis : 79/100** (vs 58/100 avant durcissement)
- Gestion des exceptions documentées : `ip_forward` (requis par KVM), faux positifs CVE

### Phase 4 — Tests d'intrusion (Semaine 4)

- **12 tests d'intrusion** exécutés sur l'infrastructure durcie
- Tests : scan évasif Nmap, brute-force SSH (Hydra), exploitation web (SQLMap, Nikto), traversée DMZ
- **Résultat : 12/12 PASS** — aucune intrusion aboutie
- Rapport de pentest avec preuves d'écran et recommandations finales

---

## Résultats obtenus

| Indicateur | Avant | Après |
|------------|-------|-------|
| Score Lynis | 58/100 | **79/100** |
| CVE critiques ouvertes | 8 | **0** |
| Ports exposés inutilement | 17 | **1** (container école) |
| Tests d'intrusion réussis | N/A | **0/12**
| WireGuard VPN | Non déployé | **✅ Actif** (healthy) |
| Services avec auth par clé | 0% | **100%** |
| IPs bannies CrowdSec (CAPI) | 0 | **16 077** |

---

## Fichiers du dépôt

| Fichier | Description |
|---------|-------------|
| `RP-01-ANDREO-Vincent.docx` | Rapport technique complet (4 phases, scripts, captures) |
| `Fiche-RP01-ANDREO-Vincent.docx` | Fiche officielle ANNEXE 7-1-A (formulaire BTS) |
| `Présentation-5min-RP01-ANDREO-Vincent.docx` | Texte de présentation orale 5 minutes |

---

## Commandes clés

```bash
# Vérifier les règles nftables actives
nft list ruleset

# Lancer un audit Lynis complet
lynis audit system --quick

# Scanner l'infrastructure avec Nmap
nmap -sS -sV -O -A --script vuln 192.168.40.0/24

# Vérifier le statut CrowdSec
cscli metrics
cscli decisions list

# Voir les alertes Suricata en temps réel
tail -f /var/log/suricata/fast.log
```

---

## Compétences validées (Bloc B2 + B3 — E5 SISR)

- **B2.1** — Concevoir une solution d'infrastructure réseau (migration AD → OpenLDAP, 802.1X, refonte DMZ)
- **B2.2** — Installer, tester et déployer (nftables, Suricata, CrowdSec, WireGuard, Debian 12)
- **B2.3** — Exploiter, dépanner et superviser (Lynis 58→79/100, monitoring, correction CVE)
- **B3.1** — Protéger les données à caractère personnel (RGPD, politique de sécurité)
- **B3.2** — Préserver l'identité numérique (authentification SSH, VPN WireGuard)
- **B3.3** — Sécuriser les équipements et les usages (CIS Benchmark, pentest 12/12 PASS)

---

## Environnement technique

- **OS Hyperviseur** : Debian 12 (Bookworm) — KVM/QEMU
- **OS VMs** : Debian 12 sur toutes les machines
- **Réseau** : 192.168.40.0/24 (LAN), DMZ isolée
- **Virtualisation** : `virsh`, `virt-manager`
- **École** : IRIS Nice — Promotion BTS SIO SISR 2025-2026

---

*Réalisation professionnelle — Épreuve E5 BTS SIO SISR.*
