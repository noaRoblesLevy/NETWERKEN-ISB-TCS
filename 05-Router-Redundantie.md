# 5. Router-redundantie

[[00-Index|← Index]]

> **Niet in Packet Tracer** — gebruik VirtualBox + VyOS  
> Doel: als één router uitvalt, blijven clients bereikbaar via de andere.

---

## 5.1 Gekozen oplossing

- [x] **VRRP** (Virtual Router Redundancy Protocol) — gedeeld Virtual IP als gateway voor clients

**Motivatie:** VRRP is een open standaard (RFC 5798), native ondersteund in VyOS. Eén VIP fungeert als gateway — clients merken niets van een failover.

---

## 5.2 Demo setup

```
[Client VM] ──── [VyOS-A (master)]  ← VRRP group 10, priority 200
                 [VyOS-B (backup)]  ← VRRP group 10, priority 100
                       ↕
               Virtual IP = gateway van client
```

| Rol | IPv4 | IPv6 |
|-----|------|------|
| VyOS-A (master) | 10.10.0.2/24 | fd00:a:a::2/64 |
| VyOS-B (backup) | 10.10.0.3/24 | fd00:a:a::3/64 |
| **VRRP VIP (gateway)** | **10.10.0.1/24** | **fd00:a:a::1/64** |

---

## 5.3 Configuratie VyOS-A (master)

```vyos
# Interface
set interfaces ethernet eth2 address '10.10.0.2/24'
set interfaces ethernet eth2 address 'fd00:a:a::2/64'

# IPv6 Router Advertisements (SLAAC voor clients)
set interfaces ethernet eth2 ipv6 router-advert send-advert true
set interfaces ethernet eth2 ipv6 router-advert prefix fd00:a:a::/64

# VRRP IPv4
set high-availability vrrp group LAN vrid 10
set high-availability vrrp group LAN interface eth2
set high-availability vrrp group LAN priority 200
set high-availability vrrp group LAN virtual-address '10.10.0.1/24'

# VRRP IPv6
set high-availability vrrp group LAN6 vrid 16
set high-availability vrrp group LAN6 interface eth2
set high-availability vrrp group LAN6 priority 200
set high-availability vrrp group LAN6 virtual-address 'fd00:a:a::1/64'
```

---

## 5.4 Configuratie VyOS-B (backup)

```vyos
# Interface
set interfaces ethernet eth2 address '10.10.0.3/24'
set interfaces ethernet eth2 address 'fd00:a:a::3/64'

# VRRP IPv4
set high-availability vrrp group LAN vrid 10
set high-availability vrrp group LAN interface eth2
set high-availability vrrp group LAN priority 100
set high-availability vrrp group LAN virtual-address '10.10.0.1/24'

# VRRP IPv6
set high-availability vrrp group LAN6 vrid 16
set high-availability vrrp group LAN6 interface eth2
set high-availability vrrp group LAN6 priority 100
set high-availability vrrp group LAN6 virtual-address 'fd00:a:a::1/64'
```

---

## 5.5 Demo procedure

1. Start VyOS-A en VyOS-B, verifieer VRRP status: `show vrrp`
2. Start client VM — controleer gateway: `ip route` → moet 10.10.0.1 zijn
3. Ping continu naar VIP: `ping 10.10.0.1` en `ping6 fd00:a:a::1`
4. **Stop VyOS-A** (master uitschakelen)
5. Verifieer dat VyOS-B de VIP overneemt: `show vrrp` op VyOS-B
6. Ping blijft actief — client merkt niets (maximaal 1–2 pakketten verlies)
7. Herstart VyOS-A → wordt opnieuw master (hogere priority)
