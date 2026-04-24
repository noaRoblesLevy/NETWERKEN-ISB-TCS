# 2. Adressering en namen — KTN

[[00-Index|← Index]]

---

## 2.1 Subnetten overzicht

| Zone | Subnet | Gateway (VRRP VIP) | Gebruik |
|------|--------|--------------------|---------|
| WAN | 203.0.113.0/29 | 203.0.113.1 (ISP) | Verbinding met internet |
| DMZ | 172.16.10.0/24 | 172.16.10.1 | Publieke servers |
| LAN | 10.10.0.0/24 | 10.10.0.1 | Intern netwerk |

---

## 2.2 Routers / Firewall

| Naam | Interface | IP-adres | Beschrijving |
|------|-----------|----------|--------------|
| VyOS-A (master) | eth0 | 203.0.113.2/29 | WAN |
| VyOS-A (master) | eth1 | 172.16.10.2/24 | DMZ |
| VyOS-A (master) | eth2 | 10.10.0.2/24 | LAN |
| VyOS-B (backup) | eth0 | 203.0.113.3/29 | WAN |
| VyOS-B (backup) | eth1 | 172.16.10.3/24 | DMZ |
| VyOS-B (backup) | eth2 | 10.10.0.3/24 | LAN |
| **VRRP VIP** | eth1 | **172.16.10.1/24** | DMZ gateway (gedeeld) |
| **VRRP VIP** | eth2 | **10.10.0.1/24** | LAN gateway (gedeeld) |

---

## 2.3 Switches

| Naam | Zone | Beschrijving |
|------|------|--------------|
| SW-DMZ | DMZ | Verbindt VyOS-A, VyOS-B en DMZ-servers |
| SW-LAN | LAN | Verbindt VyOS-A, VyOS-B en LAN-toestellen |

---

## 2.4 Servers

| Naam | IP-adres | Zone | Rol |
|------|----------|------|-----|
| SRV-WEB1 | 172.16.10.10/24 | DMZ | Apache webserver (HA node 1) |
| SRV-WEB2 | 172.16.10.11/24 | DMZ | Apache webserver (HA node 2) |
| HAProxy VIP | 172.16.10.100/24 | DMZ | Virtueel IP webdienst (HAProxy + Keepalived) |
| SRV-VPN | 172.16.10.20/24 | DMZ | OpenVPN server |
| SRV-DNS/DHCP | 10.10.0.10/24 | LAN | DNS + DHCP voor LAN-clients |

---

## 2.5 Clients (LAN)

| Naam | IP-adres | Beschrijving |
|------|----------|--------------|
| PC1 | DHCP (10.10.0.100–200) | Testclient |
| PC2 | DHCP (10.10.0.100–200) | Testclient |

> DHCP-range: 10.10.0.100 – 10.10.0.200  
> Default gateway via DHCP: 10.10.0.1 (VRRP VIP)  
> DNS via DHCP: 10.10.0.10 (SRV-DNS)
