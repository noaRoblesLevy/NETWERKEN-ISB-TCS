# 1. Bedrijf en Netwerkontwerp

[[00-Index|← Index]]

---

## 1.1 Bedrijfsprofiel

**Naam:** Katoen Natie (KTN)  
**Sector:** Havenlogistiek  
**Locatie:** Haven van Antwerpen, België  
**Core business:** Overslag, opslag en distributie van goederen in de haven. KTN beheert terminals voor bulk- en stuklading, biedt logistieke diensten aan en coördineert de goederenstromen tussen schepen, spoor en weg.  
**Publieke diensten:** Website (HTTP/HTTPS via HAProxy), remote access voor medewerkers via VPN  

---

## 1.2 Netwerkvereisten

| Vereiste | Oplossing |
|----------|-----------|
| Hoge beschikbaarheid gateway | VRRP (VyOS-A + VyOS-B) |
| Hoge beschikbaarheid webdienst | HAProxy + Keepalived (2 Apache servers) |
| Veilige remote toegang | OpenVPN met PKI-certificaten (Easy-RSA) |
| Scheiding intern / publiek | DMZ (servers) gescheiden van LAN (medewerkers) |
| Toekomstbestendigheid | IPv6 dual-stack op alle interfaces en servers |

---

## 1.3 Globaal netwerkschema

```
Internet
    │
    │ WAN: 203.0.113.0/29
    │
┌───┴───────────────────────┐
│   VyOS-A (master)         │   VRRP
│   VyOS-B (backup)         │ ← 2 redundante routers
└───┬───────────┬───────────┘
    │ DMZ       │ LAN
    │           │
┌───┴───┐   ┌──┴────┐
│  DMZ  │   │  LAN  │
│ .0/24 │   │ .0/24 │
├───────┤   ├───────┤
│WEB1   │   │PC1    │
│WEB2   │   │PC2    │
│HAProxy│   │DNS/   │
│VPN    │   │DHCP   │
└───────┘   └───────┘
```

---

## 1.4 Zones overzicht

| Zone | Subnet IPv4 | Subnet IPv6 | Doel |
|------|------------|-------------|------|
| WAN | 203.0.113.0/29 | 2001:db8:1::/64 | Verbinding met internet |
| DMZ | 172.16.10.0/24 | fd00:ac10:a::/64 | Publieke servers (web, VPN) |
| LAN | 10.10.0.0/24 | fd00:a:a::/64 | Interne medewerkers |

---

## 1.5 Gebruikte technologieën

| Technologie | Gebruik |
|-------------|---------|
| VyOS 1.4 | Router/Firewall (2x) |
| VRRP | Redundante gateway voor DMZ en LAN |
| Apache 2 | Webserver (2 nodes in DMZ) |
| HAProxy | Load balancer voor webdienst |
| Keepalived | Virtueel IP voor HAProxy |
| OpenVPN | VPN-server in DMZ |
| Easy-RSA | PKI (CA, server- en clientcertificaten) |
| dnsmasq | DHCP + DNS in LAN |
| IPv6 dual-stack | X-Factor: ULA intern, SLAAC voor clients |

> Alle demo's zijn uitgevoerd in **VirtualBox** — geen Packet Tracer voor de live demo's.
