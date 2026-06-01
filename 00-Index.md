# Threaded Case Study — Netwerken 2 ISB

**Vak:** Netwerken 2 ISB  
**Groepsleden:** Noa Robles Levy | Quinten Morreel  
**Deadline indiening:** 09/06/2026 om 20:00  
**Presentatie:** 10/06/2026 om 10:30 in GR501 (max 5 min, geen demo)  
**Tools:** VirtualBox + VyOS + Debian/Ubuntu Server (NIET Packet Tracer voor demo)  
**X-Factor:** IPv6 dual-stack (ULA intern, 2001:db8::/64 WAN)

---

> Nieuw hier? Start met [[00-Uitleg]] — uitleg over de vault, het schema en alle elementen.

## Secties

| # | Sectie | Status |
|---|--------|--------|
| 1 | [[01-Bedrijf-en-Netwerkontwerp]] | ⬜ |
| 2 | [[02-Adressering]] | 🔄 |
| 3 | [[03-LAN-Servers-DHCP-DNS]] | 🔄 |
| 4 | [[04-Firewall]] | 🔄 |
| 5 | [[05-Router-Redundantie]] | ✅ |
| 6 | [[06-Server-Redundantie]] | 🔄 |
| 7 | [[07-VPN]] | ✅ |
| 8 | [[08-X-Factor]] | 🔄 |
| 9 | [[09-Bijlagen]] | 🔄 |
| 10 | [[10-Timesheet]] | 🔄 |
| 11 | [[11-Referenties]] | ✅ |

> Status: ⬜ nog te doen · 🔄 bezig · ✅ klaar

---

## Demo-checklist

- [x] Demo redundante router/gateway (VyOS VRRP) — 08/05
- [x] Demo redundante servers (HAProxy + Keepalived) — Noa
- [x] Demo VPN-oplossing (client → OpenVPN → intern netwerk) — 21/05

## Presentatie-checklist (10/06, max 5 min)

- [ ] Bedrijf + core business uitleggen
- [ ] Globaal netwerkschema tonen
- [ ] Redundante oplossingen toelichten (VRRP + HAProxy)
- [ ] VPN-oplossing toelichten
- [ ] X-Factor toelichten (IPv6 dual-stack)
- [ ] **PowerPoint maken en uploaden naar repo**
