# 7. VPN-oplossing

[[00-Index|← Index]]

> **Niet in Packet Tracer** — gebruik VirtualBox + Ubuntu Server  
> Oplossing voor **meerdere clients** (NIET point-to-point)  
> Clients moeten een interne server bereiken

---

## 7.1 Gekozen VPN-technologie

- [x] **OpenVPN**

**Motivatie:** OpenVPN is battle-tested, ondersteunt meerdere gelijktijdige clients, werkt op UDP/TCP, en heeft clients voor alle OS'en. Bovendien ondersteunt OpenVPN dual-stack: clients krijgen zowel een IPv4 als IPv6 tunnel-adres.

---

## 7.2 Demo setup

```
[Externe client VM]
        |
   OpenVPN tunnel (UDP 1194)
        |
[SRV-VPN: 172.16.10.20 / fd00:ac10:a::20]
        |
[Interne server (bv. SRV-WEB1: 172.16.10.10)]
```

| Rol | IPv4 | IPv6 |
|-----|------|------|
| SRV-VPN (server) | 172.16.10.20/24 | fd00:ac10:a::20/64 |
| VPN tunnel pool (clients) IPv4 | 10.8.0.0/24 | — |
| VPN tunnel pool (clients) IPv6 | — | fd00:8::/64 |

---

## 7.3 Server-configuratie (`/etc/openvpn/server.conf`)

```bash
port 1194
proto udp
dev tun

# Certificaten (gegenereerd met Easy-RSA)
ca   /etc/openvpn/ca.crt
cert /etc/openvpn/server.crt
key  /etc/openvpn/server.key
dh   /etc/openvpn/dh.pem

# IPv4 tunnel adressen voor clients
server 10.8.0.0 255.255.255.0

# IPv6 dual-stack tunnel
server-ipv6 fd00:8::/64

# Push routes naar clients
push "route 172.16.10.0 255.255.255.0"         # DMZ bereikbaar
push "route-ipv6 fd00:ac10:a::/64"             # DMZ via IPv6

keepalive 10 120
cipher AES-256-GCM
```

---

## 7.4 Client-instellingen

**Server-adres:** 172.16.10.20 (of extern IP in productie)  
**Protocol/poort:** UDP 1194  
**Authenticatie:** certificaat (PKI via Easy-RSA)  
**Client-software:**
- Windows: OpenVPN GUI
- macOS: Tunnelblick
- Linux: `openvpn --config client.ovpn`

---

## 7.5 Demo procedure

1. Start SRV-VPN, verifieer dat OpenVPN luistert: `ss -ulnp | grep 1194`
2. Verbind externe client: `openvpn --config client.ovpn`
3. Verifieer tunnel-IP op client: `ip addr` → moet 10.8.0.x en fd00:8::x tonen
4. Ping interne server via IPv4: `ping 172.16.10.10`
5. Ping interne server via IPv6: `ping6 fd00:ac10:a::10`
6. Open browser: `http://172.16.10.10` én `http://[fd00:ac10:a::10]` → beide werken
7. Verifieer dat zonder VPN (tunnel down) de interne servers **niet** bereikbaar zijn
