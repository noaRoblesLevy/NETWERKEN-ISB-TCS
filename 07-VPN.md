# 7. VPN-oplossing

[[00-Index|← Index]]

> **Niet in Packet Tracer** — gebruik VyOS of Linux  
> Oplossing voor **meerdere clients** (NIET point-to-point)  
> Clients moeten een interne server bereiken

---

## 7.1 Gekozen VPN-technologie

- [ ] OpenVPN
- [ ] WireGuard
- [ ] IPsec / IKEv2
- [ ] VyOS L2TP

**Motivatie:**

---

## 7.2 Minimum setup

```
[Thuis-client]
     |
  VPN tunnel
     |
[VPN concentrator] ── [interne server]
```

---

## 7.3 Client-instellingen

**Server-adres:**  
**Protocol/poort:**  
**Authenticatie:**  
**Client-software:**  
- Windows: 
- macOS: 
- Linux: 

---

## 7.4 Server-configuratie

```bash
# VPN server configuratie
```

---

## 7.5 Demo procedure (test)

> Client verbindt via VPN en bereikt een DMZ- of LAN-server.

1. 
2. 
3. Verifieer verbinding met interne server (ping / browser)
4. Welke aanpassingen aan routetabel of encryptie waren nodig?
