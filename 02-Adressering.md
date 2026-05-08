# 2. Adressering en namen — KTN

[[00-Index|← Index]]

---

## 2.1 Subnetten overzicht

| Zone | IPv4 subnet | IPv6 subnet | Gateway IPv4 | Gateway IPv6 | Gebruik |
|------|------------|-------------|--------------|--------------|---------|
| WAN | 203.0.113.0/29 | 2001:db8:1::/64 | 203.0.113.1 (ISP) | 2001:db8:1::1 | Verbinding met internet |
| DMZ | 172.16.10.0/24 | fd00:ac10:a::/64 | 172.16.10.1 (VRRP) | fd00:ac10:a::1 (VRRP) | Publieke servers |
| LAN | 10.10.0.0/24 | fd00:a:a::/64 | 10.10.0.1 (VRRP) | fd00:a:a::1 (VRRP) | Intern netwerk |

> IPv6 intern: **ULA** (Unique Local Addresses, fd00::/8) — privé-equivalent voor IPv6, niet routeerbaar op internet.  
> IPv6 WAN: **2001:db8::/32** — officieel documentatieprefix voor labs en documentatie.

---

## 2.2 Routers / Firewall

| Naam | Interface | IPv4-adres | IPv6-adres | Beschrijving |
|------|-----------|------------|------------|--------------|
| VyOS-A (master) | eth0 | 203.0.113.2/29 | 2001:db8:1::2/64 | WAN |
| VyOS-A (master) | eth1 | 172.16.10.2/24 | fd00:ac10:a::2/64 | DMZ |
| VyOS-A (master) | eth2 | 10.10.0.2/24 | fd00:a:a::2/64 | LAN |
| VyOS-B (backup) | eth0 | 203.0.113.3/29 | 2001:db8:1::3/64 | WAN |
| VyOS-B (backup) | eth1 | 172.16.10.3/24 | fd00:ac10:a::3/64 | DMZ |
| VyOS-B (backup) | eth2 | 10.10.0.3/24 | fd00:a:a::3/64 | LAN |
| **VRRP VIP** | eth1 | **172.16.10.1/24** | **fd00:ac10:a::1/64** | DMZ gateway (gedeeld) |
| **VRRP VIP** | eth2 | **10.10.0.1/24** | **fd00:a:a::1/64** | LAN gateway (gedeeld) |

---

## 2.3 Switches

| Naam | Zone | Beschrijving |
|------|------|--------------|
| SW-DMZ | DMZ | Verbindt VyOS-A, VyOS-B en DMZ-servers |
| SW-LAN | LAN | Verbindt VyOS-A, VyOS-B en LAN-toestellen |

---

## 2.4 Servers

| Naam | IPv4-adres | IPv6-adres | Zone | Rol |
|------|------------|------------|------|-----|
| SRV-WEB1 | 172.16.10.10/24 | fd00:ac10:a::10/64 | DMZ | Apache webserver (HA node 1) |
| SRV-WEB2 | 172.16.10.11/24 | fd00:ac10:a::11/64 | DMZ | Apache webserver (HA node 2) |
| HAProxy VIP | 172.16.10.100/24 | fd00:ac10:a::100/64 | DMZ | Virtueel IP webdienst (HAProxy + Keepalived) |
| SRV-VPN | 172.16.10.20/24 | fd00:ac10:a::20/64 | DMZ | OpenVPN server |
| SRV-DNS/DHCP | 10.10.0.10/24 | fd00:a:a::10/64 | LAN | DNS + DHCP voor LAN-clients |

---

## 2.5 Clients (LAN)

| Naam | IPv4-adres | IPv6-adres | Beschrijving |
|------|------------|------------|--------------|
| PC1 | DHCP (10.10.0.100–200) | SLAAC (fd00:a:a::/64) | Testclient |
| PC2 | DHCP (10.10.0.100–200) | SLAAC (fd00:a:a::/64) | Testclient |

> DHCP-range: 10.10.0.100 – 10.10.0.200  
> Default gateway via DHCP: 10.10.0.1 (VRRP VIP)  
> DNS via DHCP: 10.10.0.10 (SRV-DNS)  
> IPv6: clients ontvangen hun adres via **SLAAC** — VyOS adverteert het prefix `fd00:a:a::/64` via Router Advertisements.
