# Uitleg: wat staat er in deze vault?

[[00-Index|← Index]]

---

## Wat is deze vault?

Een Obsidian vault is een map met Markdown-bestanden. Markdown is een simpele tekstopmaak:
- `**vet**` → **vet**
- `# Titel` → grote titel
- `[[bestandsnaam]]` → klikbare link naar een ander bestand in de vault

Obsidian maakt er een klikbaar kennissysteem van. Je hoeft Obsidian niet te gebruiken — de bestanden zijn gewone `.md` tekstbestanden die je ook in VS Code of Notepad++ kunt openen.

---

## Structuur van de vault

| Bestand | Inhoud |
|---------|--------|
| `00-Index.md` | Startpagina met links + demo-checklist (21/05) |
| `01-Bedrijf-en-Netwerkontwerp.md` | Bedrijfsprofiel + placeholders voor schema's |
| `02-Adressering.md` | IP-adressen van routers / switches / servers |
| `03-LAN-Servers-DHCP-DNS.md` | Hoe clients een IP krijgen, DNS-config |
| `04-Firewall.md` | Firewallregels DMZ↔Internet en DMZ↔LAN |
| `05-Router-Redundantie.md` | VRRP-config VyOS-A en VyOS-B |
| `06-Server-Redundantie.md` | HA-dienst met HAProxy + Keepalived |
| `07-VPN.md` | VPN-oplossing voor meerdere externe clients |
| `08-X-Factor.md` | Deep dive naar één aspect naar keuze |
| `09-Bijlagen.md` | Configs, scripts, screenshots |
| `10-Timesheet.md` | Wie deed wat wanneer |
| `11-Referenties.md` | Bronnen |

---

## Het netwerkschema (`netwerkschema.drawio`)

Een `.drawio` bestand is XML-tekst die draw.io omzet naar een visueel diagram.
Openen via [app.diagrams.net](https://app.diagrams.net) (gratis, geen installatie) of de draw.io desktopapp.

### Topologie in woorden

```
[VPN-client thuis]
        |
        |  VPN tunnel (UDP 1194) — gestippeld groen
        ↓
  [Internet cloud]
       /        \
 [VyOS-A]──────[VyOS-B]
  master          backup
  (blauw)         (rood)
      ↕  VRRP heartbeat  ↕

  [Virtual IPs]  ← de gateway-IPs die clients kennen
      /                       \
 [SW-DMZ]                 [SW-LAN]
  /   |   \                /   |   \
web1 web2 vpn            dns  pc1  pc2
```

### Kleurcode

| Kleur / stijl | Betekenis |
|---------------|-----------|
| **Blauw — VyOS-A** | Actieve master router. Normaal loopt al het verkeer hierover. |
| **Rood — VyOS-B** | Backup router. Neemt automatisch over als VyOS-A uitvalt (VRRP). |
| Volle blauwe lijn | Actieve verbinding (master-pad) |
| Gestippelde rode lijn | Backup-verbinding, normaal inactief |
| **Gele box "Virtual IPs"** | De IP-adressen die clients kennen als gateway. Bij uitval van VyOS-A verschuiven deze automatisch naar VyOS-B. |
| Gele stippellijn "heartbeat" | VyOS-A en VyOS-B sturen elkaar constant een signaal. Stopt het signaal → VyOS-B neemt de VIPs over. |
| **Oranje zone — DMZ** | 172.16.1.0/24. Publiek toegankelijke servers (web, VPN). Geïsoleerd van het LAN. |
| **Groene zone — LAN** | 192.168.10.0/24. Intern netwerk. Niet bereikbaar van buiten. |
| Groene stippellijn | VPN-tunnel van externe client naar SRV-VPN. |

### Wat betekenen de servers?

| Server | IP | Rol |
|--------|----|-----|
| SRV-WEB1 | 172.16.1.10 | Apache webserver, node 1 van de HA-opstelling |
| SRV-WEB2 | 172.16.1.11 | Apache webserver, node 2 van de HA-opstelling |
| VIP 172.16.1.100 | — | Virtueel IP van HAProxy + Keepalived. Clients surfen naar dit adres; HAProxy verdeelt het verkeer over WEB1 en WEB2. Bij uitval van één server gaat alles naar de andere. |
| SRV-VPN | 172.16.1.20 | OpenVPN server. Externe clients verbinden hiermee en krijgen toegang tot het interne netwerk via een versleutelde tunnel. |
| SRV-DNS/DHCP | 192.168.10.10 | Geeft LAN-clients automatisch een IP-adres (DHCP) en lost domeinnamen op (DNS). |
| PC1 / PC2 | DHCP | Testclients in het LAN. Krijgen automatisch een IP van SRV-DNS/DHCP. |

### Waarom twee routers (VRRP)?

VRRP (Virtual Router Redundancy Protocol) laat twee routers één gedeeld IP-adres beheren.
- Normaal is VyOS-A de **master**: hij houdt de Virtual IPs actief.
- VyOS-B stuurt elke seconde een **heartbeat** naar VyOS-A.
- Als de heartbeat wegvalt, wordt VyOS-B automatisch master en neemt de VIPs over.
- Clients merken niets — ze blijven hetzelfde gateway-IP gebruiken.

### Waarom twee webservers (HAProxy + Keepalived)?

- **Keepalived** doet hetzelfde als VRRP maar dan voor servers: het beheert een Virtual IP (172.16.1.100) tussen WEB1 en WEB2.
- **HAProxy** is een load balancer: hij verdeelt inkomende verzoeken over WEB1 en WEB2.
- Als één webserver uitvalt, detecteert HAProxy dit via een health check en stuurt al het verkeer naar de andere.

---

## Gebruikte tools

| Tool | Waarvoor | Gratis? |
|------|----------|---------|
| Obsidian | Vault bekijken en bewerken | Ja |
| draw.io / diagrams.net | Netwerkschema openen en aanpassen | Ja |
| VirtualBox | VMs draaien voor de demo | Ja |
| VyOS | Router/firewall OS in VMs | Ja |
| GitHub | Samenwerken via git | Ja |
