# 3. LAN-servers: DHCP en DNS

[[00-Index|← Index]]

---

## 3.1 DHCP

> Hoe krijgen clients een IP-adres?

**DHCP-server:** SRV-DNS/DHCP — 10.10.0.10 (Ubuntu Server + dnsmasq)  
**IP-range clients:** 10.10.0.100 – 10.10.0.200  
**Gateway:** 10.10.0.1 (VRRP VIP — VyOS-A of VyOS-B)  
**DNS:** 10.10.0.10 (zichzelf)  
**Lease time:** 12 uur  

> **IPv6:** clients krijgen hun IPv6-adres via **SLAAC**, niet via DHCPv6. VyOS stuurt Router Advertisements met prefix `fd00:a:a::/64` — de client berekent zelf zijn adres.

### Configuratie (`/etc/dnsmasq.conf`)

```ini
# DHCP voor LAN
interface=eth0
dhcp-range=10.10.0.100,10.10.0.200,12h
dhcp-option=3,10.10.0.1        # default gateway
dhcp-option=6,10.10.0.10       # DNS-server

# Vaste IP's voor servers (optioneel, op basis van MAC)
# dhcp-host=aa:bb:cc:dd:ee:ff,10.10.0.50,SRV-extra
```

---

## 3.2 DNS

> Hoe kan een client verbinding maken met het internet en interne namen oplossen?

**DNS-server:** SRV-DNS/DHCP — 10.10.0.10 (dnsmasq)  
**Interne zone:** `ktn.local`  
**Forwarder (naar internet):** 1.1.1.1 (Cloudflare) / 8.8.8.8 (Google)  

### Interne hostnamen

| Naam | IPv4 | IPv6 |
|------|------|------|
| web1.ktn.local | 172.16.10.10 | fd00:ac10:a::10 |
| web2.ktn.local | 172.16.10.11 | fd00:ac10:a::11 |
| web.ktn.local (VIP) | 172.16.10.100 | fd00:ac10:a::100 |
| vpn.ktn.local | 172.16.10.20 | fd00:ac10:a::20 |
| dns.ktn.local | 10.10.0.10 | fd00:a:a::10 |

### Configuratie (`/etc/dnsmasq.conf` — aanvulling)

```ini
# DNS forwarder
server=1.1.1.1
server=8.8.8.8

# Interne zone ktn.local — IPv4
address=/web1.ktn.local/172.16.10.10
address=/web2.ktn.local/172.16.10.11
address=/web.ktn.local/172.16.10.100
address=/vpn.ktn.local/172.16.10.20
address=/dns.ktn.local/10.10.0.10

# Interne zone ktn.local — IPv6
address=/web1.ktn.local/fd00:ac10:a::10
address=/web2.ktn.local/fd00:ac10:a::11
address=/web.ktn.local/fd00:ac10:a::100
address=/vpn.ktn.local/fd00:ac10:a::20
address=/dns.ktn.local/fd00:a:a::10

# Lokale domeinnaam voor DHCP-clients
domain=ktn.local
local=/ktn.local/
```

---

## 3.3 Internetverbinding clients

> Hoe raken clients aan het internet?

Clients sturen al hun verkeer naar de **VRRP VIP (10.10.0.1)** als default gateway. VyOS doet vervolgens **NAT/Masquerade**: het vervangt het privé-IP van de client door het publieke WAN-IP (203.0.113.2 of .3) voor pakketjes richting internet.

```
PC1 (10.10.0.100) → gateway 10.10.0.1 (VyOS) → NAT → 203.0.113.2 → Internet
```

### NAT configuratie op VyOS

```vyos
set nat source rule 10 outbound-interface name 'eth0'
set nat source rule 10 source address '10.10.0.0/24'
set nat source rule 10 translation address masquerade

set nat source rule 20 outbound-interface name 'eth0'
set nat source rule 20 source address '172.16.10.0/24'
set nat source rule 20 translation address masquerade
```

> IPv6: in een productieomgeving zou de ISP een publiek /48 prefix delegeren zodat NAT niet nodig is. In het lab gebruiken we ULA-adressen die niet naar buiten routen — internet blijft via IPv4 NAT gaan.
