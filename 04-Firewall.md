# 4. Firewall-regels

[[00-Index|← Index]]

> Firewall draait op **VyOS-A en VyOS-B** (beide identiek geconfigureerd).  
> Principe: **default deny** — alles geblokkeerd, enkel wat expliciet toegestaan is passeert.  
> Twee paren uitgewerkt: DMZ ↔ Internet en DMZ ↔ LAN.

---

## 4.1 DMZ ↔ Internet

### In woorden

| # | Richting | Wat | Waarom |
|---|----------|-----|--------|
| 1 | Internet → DMZ | HTTP (80) en HTTPS (443) toegelaten naar HAProxy VIP | Bezoekers moeten de website bereiken |
| 2 | Internet → DMZ | UDP 1194 toegelaten naar SRV-VPN | VPN-clients moeten verbinden |
| 3 | Internet → DMZ | Gevestigde verbindingen (established/related) doorgelaten | Antwoorden op uitgaande verbindingen terugkeren |
| 4 | Internet → DMZ | Al het andere geblokkeerd | Aanvallers mogen geen andere diensten bereiken |
| 5 | DMZ → Internet | Toegelaten (voor updates, NTP, ...) | Servers moeten patches kunnen downloaden |

### Technische regels (VyOS)

```vyos
# Firewall policy: Internet → DMZ (ingress op eth0, richting eth1)
set firewall ipv4 forward filter rule 10 action 'accept'
set firewall ipv4 forward filter rule 10 state established 'enable'
set firewall ipv4 forward filter rule 10 state related 'enable'

set firewall ipv4 forward filter rule 20 action 'accept'
set firewall ipv4 forward filter rule 20 destination address '172.16.10.100'
set firewall ipv4 forward filter rule 20 destination port '80,443'
set firewall ipv4 forward filter rule 20 protocol 'tcp'

set firewall ipv4 forward filter rule 30 action 'accept'
set firewall ipv4 forward filter rule 30 destination address '172.16.10.20'
set firewall ipv4 forward filter rule 30 destination port '1194'
set firewall ipv4 forward filter rule 30 protocol 'udp'

set firewall ipv4 forward filter rule 99 action 'drop'
set firewall ipv4 forward filter rule 99 inbound-interface name 'eth0'
set firewall ipv4 forward filter rule 99 outbound-interface name 'eth1'

# IPv6 — zelfde logica
set firewall ipv6 forward filter rule 10 action 'accept'
set firewall ipv6 forward filter rule 10 state established 'enable'
set firewall ipv6 forward filter rule 10 state related 'enable'

set firewall ipv6 forward filter rule 20 action 'accept'
set firewall ipv6 forward filter rule 20 destination address 'fd00:ac10:a::100'
set firewall ipv6 forward filter rule 20 destination port '80,443'
set firewall ipv6 forward filter rule 20 protocol 'tcp'

set firewall ipv6 forward filter rule 30 action 'accept'
set firewall ipv6 forward filter rule 30 destination address 'fd00:ac10:a::20'
set firewall ipv6 forward filter rule 30 destination port '1194'
set firewall ipv6 forward filter rule 30 protocol 'udp'

set firewall ipv6 forward filter rule 99 action 'drop'
set firewall ipv6 forward filter rule 99 inbound-interface name 'eth0'
set firewall ipv6 forward filter rule 99 outbound-interface name 'eth1'
```

---

## 4.2 DMZ ↔ LAN

### In woorden

| # | Richting | Wat | Waarom |
|---|----------|-----|--------|
| 1 | LAN → DMZ | HTTP/HTTPS toegelaten naar web.ktn.local | LAN-gebruikers mogen de interne website bezoeken |
| 2 | LAN → DMZ | Gevestigde verbindingen doorgelaten | Antwoorden komen terug |
| 3 | DMZ → LAN | Al het andere geblokkeerd | Als een DMZ-server gehackt wordt, mag die de LAN niet bereiken |
| 4 | DMZ → LAN | Enkel DNS-antwoorden (53) naar SRV-DNS doorgelaten | DMZ-servers kunnen interne namen opvragen |

### Technische regels (VyOS)

```vyos
# LAN → DMZ: toegelaten
set firewall ipv4 forward filter rule 110 action 'accept'
set firewall ipv4 forward filter rule 110 inbound-interface name 'eth2'
set firewall ipv4 forward filter rule 110 outbound-interface name 'eth1'
set firewall ipv4 forward filter rule 110 destination port '80,443'
set firewall ipv4 forward filter rule 110 protocol 'tcp'

set firewall ipv4 forward filter rule 111 action 'accept'
set firewall ipv4 forward filter rule 111 state established 'enable'
set firewall ipv4 forward filter rule 111 state related 'enable'

# DMZ → LAN: enkel DNS toegelaten
set firewall ipv4 forward filter rule 120 action 'accept'
set firewall ipv4 forward filter rule 120 inbound-interface name 'eth1'
set firewall ipv4 forward filter rule 120 outbound-interface name 'eth2'
set firewall ipv4 forward filter rule 120 destination address '10.10.0.10'
set firewall ipv4 forward filter rule 120 destination port '53'
set firewall ipv4 forward filter rule 120 protocol 'tcp_udp'

# DMZ → LAN: rest geblokkeerd
set firewall ipv4 forward filter rule 199 action 'drop'
set firewall ipv4 forward filter rule 199 inbound-interface name 'eth1'
set firewall ipv4 forward filter rule 199 outbound-interface name 'eth2'

# IPv6 — zelfde logica
set firewall ipv6 forward filter rule 110 action 'accept'
set firewall ipv6 forward filter rule 110 inbound-interface name 'eth2'
set firewall ipv6 forward filter rule 110 outbound-interface name 'eth1'
set firewall ipv6 forward filter rule 110 destination port '80,443'
set firewall ipv6 forward filter rule 110 protocol 'tcp'

set firewall ipv6 forward filter rule 120 action 'accept'
set firewall ipv6 forward filter rule 120 inbound-interface name 'eth1'
set firewall ipv6 forward filter rule 120 outbound-interface name 'eth2'
set firewall ipv6 forward filter rule 120 destination address 'fd00:a:a::10'
set firewall ipv6 forward filter rule 120 destination port '53'
set firewall ipv6 forward filter rule 120 protocol 'tcp_udp'

set firewall ipv6 forward filter rule 199 action 'drop'
set firewall ipv6 forward filter rule 199 inbound-interface name 'eth1'
set firewall ipv6 forward filter rule 199 outbound-interface name 'eth2'
```

---

## 4.3 Verificatie

```bash
# Op VyOS: toon alle actieve firewall regels
show firewall

# Test: ping van LAN-client naar DMZ-server (moet werken via HTTP, niet via ping tenzij ICMP toegelaten)
# Test: ping van DMZ-server naar LAN-client (moet geblokkeerd worden)
```
