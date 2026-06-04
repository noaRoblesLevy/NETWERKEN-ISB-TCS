# 8. X Factor

[[00-Index|← Index]]

> Kies één aspect en doe een *deep dive* — ga verder dan de minimum vereisten.

---

## 8.1 Gekozen onderwerp

- [x] **IPv6 dual-stack**

**Motivatie:**  
Het KTN-netwerk draait volledig op IPv4. Als X-Factor voegen we IPv6 dual-stack toe: elke interface krijgt naast zijn IPv4-adres ook een IPv6-adres. Clients en servers kunnen zo via beide protocollen communiceren. Dit is de realistische evolutie voor elk modern bedrijfsnetwerk — ISP's leveren al jaren dual-stack aansluitingen.

---

## 8.2 IPv6-adresplan

### Subnetten

| Zone | IPv4 subnet | IPv6 subnet (ULA) | Gebruik |
|------|------------|-------------------|---------|
| WAN  | 203.0.113.0/29 | 2001:db8:1::/64 | Verbinding met internet (documentatieprefix) |
| DMZ  | 172.16.10.0/24 | fd00:ac10:a::/64 | Publieke servers |
| LAN  | 10.10.0.0/24   | fd00:a:a::/64    | Intern netwerk |

### Routers / Firewall

| Naam | Interface | IPv6-adres |
|------|-----------|------------|
| VyOS-A (master) | eth0 (WAN) | 2001:db8:1::2/64 |
| VyOS-A (master) | eth1 (DMZ) | fd00:ac10:a::2/64 |
| VyOS-A (master) | eth2 (LAN) | fd00:a:a::2/64 |
| VyOS-B (backup) | eth0 (WAN) | 2001:db8:1::3/64 |
| VyOS-B (backup) | eth1 (DMZ) | fd00:ac10:a::3/64 |
| VyOS-B (backup) | eth2 (LAN) | fd00:a:a::3/64 |
| VRRP VIP | eth1 (DMZ) | fd00:ac10:a::1/64 |
| VRRP VIP | eth2 (LAN) | fd00:a:a::1/64 |

### Servers

| Naam | IPv4 | IPv6 |
|------|------|------|
| SRV-WEB1 | 172.16.10.10/24 | fd00:ac10:a::10/64 |
| SRV-WEB2 | 172.16.10.11/24 | fd00:ac10:a::11/64 |
| HAProxy VIP | 172.16.10.100/24 | fd00:ac10:a::100/64 |
| SRV-VPN | 172.16.10.20/24 | fd00:ac10:a::20/64 |
| SRV-DNS/DHCP | 10.10.0.10/24 | fd00:a:a::10/64 |

### Clients (LAN)

| Naam | IPv4 | IPv6 |
|------|------|------|
| PC1 | DHCP (10.10.0.100–200) | SLAAC (fd00:a:a::/64) |
| PC2 | DHCP (10.10.0.100–200) | SLAAC (fd00:a:a::/64) |

> LAN-clients krijgen hun IPv6-adres via **SLAAC** (Stateless Address Autoconfiguration) — de router adverteert het prefix, de client berekent zelf zijn adres op basis van zijn MAC-adres.

---

## 8.3 Wat verandert per demo

### Demo 1 — Router redundantie (VRRP)
- VyOS-A en VyOS-B krijgen elk een IPv6-adres op eth1 en eth2
- VRRP VIP werkt ook voor IPv6 (VyOS ondersteunt dit via `vrrp group`)
- Aantonen: ping6 naar VIP blijft werken als VyOS-A uitvalt

### Demo 2 — Server redundantie (HAProxy)
- Apache luistert op zowel IPv4 als IPv6 (`Listen [::]:80`)
- HAProxy VIP krijgt ook een IPv6-adres: `fd00:ac10:a::100`
- Aantonen: `curl http://[fd00:ac10:a::100]/` werkt en load balancet

### Demo 3 — VPN
- OpenVPN tunnel over IPv4, maar pushed IPv6-routes naar client
- Client bereikt interne servers via IPv6 door de tunnel
- Alternatief: WireGuard ondersteunt native dual-stack eenvoudiger

---

## 8.4 VyOS dual-stack configuratie (voorbeeld LAN interface)

```vyos
set interfaces ethernet eth2 address '10.10.0.2/24'
set interfaces ethernet eth2 address 'fd00:a:a::2/64'
set interfaces ethernet eth2 ipv6 router-advert send-advert true
set interfaces ethernet eth2 ipv6 router-advert prefix fd00:a:a::/64
```

> `router-advert` zorgt voor SLAAC: VyOS stuurt periodiek Router Advertisements uit zodat clients automatisch een IPv6-adres configureren.

---

## 8.5 Uitwerking

### Wat is geïmplementeerd

**VRRP IPv6 (demo 1 — volledig getest):**  
Naast de IPv4 VRRP-groep (vrid 10) is een aparte VRRP-groep voor IPv6 geconfigureerd (vrid 16). Beide routers adverteren hetzelfde VIP:

```vyos
set high-availability vrrp group LAN6 vrid 16
set high-availability vrrp group LAN6 interface eth2
set high-availability vrrp group LAN6 priority 200        # VyOS-A
set high-availability vrrp group LAN6 address 'fd00:a:a::1/64'
```

**SLAAC via Router Advertisements:**  
VyOS-A stuurt Router Advertisements op de LAN-interface. Clients ontvangen het prefix `fd00:a:a::/64` en genereren automatisch hun eigen IPv6-adres (EUI-64 op basis van MAC-adres). Geen DHCPv6 nodig.

**HAProxy dual-stack (demo 2):**  
HAProxy bindt op zowel `*:80` (IPv4) als `[::]:80` (IPv6). Keepalived beheert het IPv6 VIP `fd00:ac10:a::100/64` naast het IPv4 VIP.

### Wat niet geïmplementeerd is

| Onderdeel | Reden |
|-----------|-------|
| DHCPv6 | Niet nodig — SLAAC volstaat voor clients |
| Echte publieke IPv6 (niet 2001:db8::/32) | VirtualBox-omgeving zonder echte ISP-aansluiting |
| OpenVPN IPv6-routing door tunnel | Demo uitgesteld — wordt getoond op examen |

### Meerwaarde t.o.v. puur IPv4

- **Toekomstbestendig:** IPv4-adressen zijn uitgeput, dual-stack is de standaard overgang
- **Geen NAT nodig voor IPv6:** elke host heeft een uniek (ULA) adres — eenvoudigere troubleshooting
- **VRRP werkt identiek:** VyOS ondersteunt native IPv6 VRRP zonder extra software
- **SLAAC vereenvoudigt beheer:** geen DHCP-server nodig voor IPv6-adresuitgifte

---

## 8.6 Demo resultaten

| Test | Resultaat |
|------|-----------|
| VyOS-A eth2 IPv6 adres | `fd00:a:a::2/64` actief ✅ |
| VyOS-B eth2 IPv6 adres | `fd00:a:a::3/64` actief ✅ |
| VRRP VIP IPv6 (LAN) | `fd00:a:a::1/64` actief op master ✅ |
| VRRP failover IPv6 | VyOS-B neemt IPv6 VIP over bij uitval VyOS-A ✅ |
| ping6 naar VIP bij failover | Blijft actief, zelfde packet loss als IPv4 (~2%) ✅ |
| SLAAC client adres | Client ontvangt `fd00:a:a::<EUI-64>/64` automatisch ✅ |
| HAProxy IPv6 binding | `curl http://[fd00:ac10:a::100]/` werkt ✅ |
