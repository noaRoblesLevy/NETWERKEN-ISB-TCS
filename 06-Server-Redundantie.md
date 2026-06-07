# 6. Server-redundantie (DMZ)

[[00-Index|← Index]]

> **Niet in Packet Tracer** — gebruik VirtualBox + Ubuntu Server 22.04  
> De dienst moet *dynamisch* en *high available* zijn.

---

## 6.1 Gekozen dienst en oplossing

**Dienst:** HTTP (Apache webserver)  
**OS:** Ubuntu Server 22.04  
**HA-oplossing:** HAProxy (load balancer) + Keepalived (VIP/VRRP) + lsyncd (content sync)

```
[Client] ──── [HAProxy VIP: 172.16.10.100]
                    ├── [WEB1: 172.16.10.10]  (Apache + HAProxy + Keepalived MASTER)
                    └── [WEB2: 172.16.10.11]  (Apache + HAProxy + Keepalived BACKUP)
                          ↕ lsyncd (real-time sync van /var/www/html)
```

---

## 6.2 IP-adressen demo

| Rol | IPv4 | Zone |
|-----|------|------|
| WEB1 | 172.16.10.10/24 | Internal Network `demo-web` |
| WEB2 | 172.16.10.11/24 | Internal Network `demo-web` |
| **HAProxy VIP** | **172.16.10.100/24** | Virtueel (Keepalived) |
| Client | 172.16.10.200/24 | Internal Network `demo-web` |

---

## 6.3 Configuratie Apache (WEB1 en WEB2)

```bash
sudo apt install apache2 -y
sudo systemctl enable apache2

# Apache verplaatsen naar poort 8080 (HAProxy gebruikt poort 80)
sudo nano /etc/apache2/ports.conf
# Verander: Listen 80 → Listen 8080

sudo nano /etc/apache2/sites-enabled/000-default.conf
# Verander: <VirtualHost *:80> → <VirtualHost *:8080>

sudo systemctl restart apache2
```

---

## 6.4 Configuratie HAProxy (`/etc/haproxy/haproxy.cfg`)

Identiek op WEB1 en WEB2:

```haproxy
global
    log /dev/log local0

defaults
    mode http
    timeout connect 5s
    timeout client 30s
    timeout server 30s

frontend web_front
    bind *:80
    default_backend web_back

backend web_back
    balance roundrobin
    option httpchk GET /
    server web1 172.16.10.10:8080 check
    server web2 172.16.10.11:8080 check
```

```bash
sudo systemctl enable haproxy && sudo systemctl restart haproxy
```

---

## 6.5 Configuratie Keepalived

### WEB1 (`/etc/keepalived/keepalived.conf`) — MASTER

```
vrrp_instance WEB_VIP {
    state MASTER
    interface enp0s3
    virtual_router_id 51
    priority 200
    advert_int 1
    virtual_ipaddress {
        172.16.10.100/24
    }
}
```

### WEB2 — BACKUP

```
vrrp_instance WEB_VIP {
    state BACKUP
    interface enp0s3
    virtual_router_id 51
    priority 100
    advert_int 1
    virtual_ipaddress {
        172.16.10.100/24
    }
}
```

```bash
sudo systemctl enable keepalived && sudo systemctl restart keepalived
```

---

## 6.6 Content synchronisatie met lsyncd

Om te garanderen dat WEB1 en WEB2 altijd dezelfde webinhoud tonen, wordt **lsyncd** gebruikt. lsyncd combineert `inotify` (kernel file watching) met `rsync`: zodra een bestand wijzigt op WEB1, wordt het onmiddellijk naar WEB2 gepusht via SSH.

### Setup SSH-sleutel (WEB1 → WEB2)

```bash
# Op WEB1 als root
sudo ssh-keygen -t ed25519 -N "" -f /root/.ssh/id_ed25519
sudo ssh-copy-id web2@172.16.10.11

# WEB2 geeft schrijfrechten aan web2-gebruiker
sudo chown -R web2:web2 /var/www/html   # op WEB2
```

### lsyncd configuratie (`/etc/lsyncd/lsyncd.conf.lua`) — op WEB1

```lua
settings {
    logfile = "/var/log/lsyncd.log",
    statusFile = "/var/log/lsyncd-status.log",
}

sync {
    default.rsyncssh,
    source = "/var/www/html",
    host = "web2@172.16.10.11",
    targetdir = "/var/www/html",
    rsync = {
        archive = true,
        compress = true,
    },
}
```

```bash
sudo systemctl enable lsyncd && sudo systemctl restart lsyncd
```

---

## 6.7 Demo procedure

1. Start WEB1, WEB2 en CLIENT
2. Verifieer VIP op WEB1: `ip addr show enp0s3` → `172.16.10.100` zichtbaar
3. Verifieer load balancing via client:
   ```bash
   for i in $(seq 1 6); do curl http://172.16.10.100; done
   ```
4. **Content sync aantonen:** wijzig inhoud op WEB1 en check WEB2:
   ```bash
   echo "<h1>KTN Website - Updated</h1>" | sudo tee /var/www/html/index.html
   # Na 2-3 seconden op WEB2:
   cat /var/www/html/index.html   # → zelfde inhoud
   ```
5. **Failover aantonen:** stop Apache op WEB1:
   ```bash
   sudo systemctl stop apache2   # op WEB1
   ```
6. Client blijft werken via WEB2 — geen onderbreking
7. Herstart Apache op WEB1 → load balancing hervat automatisch

---

## 6.8 Demo resultaten

| Test | Resultaat |
|------|-----------|
| VIP aanwezig op WEB1 (MASTER) | `172.16.10.100` zichtbaar op enp0s3 ✅ |
| Load balancing (round robin) | Requests wisselen af tussen WEB1 en WEB2 ✅ |
| Failover bij uitval WEB1 | HAProxy stuurt alles naar WEB2, geen foutmelding ✅ |
| Failback bij herstel WEB1 | Load balancing hervat automatisch ✅ |
| Content sync (lsyncd) | Wijziging op WEB1 verschijnt binnen 3s op WEB2 ✅ |
| VIP overname door WEB2 | Keepalived geeft VIP aan WEB2 als WEB1 volledig uitvalt ✅ |
