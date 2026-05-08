# 6. Server-redundantie (DMZ)

[[00-Index|← Index]]

> **Niet in Packet Tracer** — gebruik VirtualBox + Ubuntu Server  
> De dienst moet *dynamisch* en *high available* zijn.

---

## 6.1 Gekozen dienst en oplossing

**Dienst:** HTTP (Apache webserver)  
**OS:** Ubuntu Server 22.04  
**HA-oplossing:** HAProxy (load balancer) + Keepalived (VIP/VRRP)

```
[Client] ──── [HAProxy VIP: .100 / ::100]
                    ├── [WEB1: .10 / ::10]  (Apache)
                    └── [WEB2: .11 / ::11]  (Apache)
                          ↕ (Keepalived bewaakt)
```

---

## 6.2 IP-adressen demo

| Rol | IPv4 | IPv6 |
|-----|------|------|
| SRV-WEB1 | 172.16.10.10/24 | fd00:ac10:a::10/64 |
| SRV-WEB2 | 172.16.10.11/24 | fd00:ac10:a::11/64 |
| **HAProxy VIP** | **172.16.10.100/24** | **fd00:ac10:a::100/64** |

---

## 6.3 Configuratie Apache (WEB1 en WEB2)

```bash
apt install apache2 -y

# Maak een onderscheidende pagina per server (voor demo)
echo "<h1>WEB1</h1>" > /var/www/html/index.html   # op WEB1
echo "<h1>WEB2</h1>" > /var/www/html/index.html   # op WEB2

# IPv6 luisteren (staat standaard aan in Apache)
# Controleer: /etc/apache2/ports.conf → Listen 80
```

---

## 6.4 Configuratie HAProxy + Keepalived (op WEB1 en WEB2)

### HAProxy (`/etc/haproxy/haproxy.cfg`)

```haproxy
frontend web_front
    bind *:80
    bind [::]:80          # IPv6 dual-stack
    default_backend web_back

backend web_back
    balance roundrobin
    server web1 172.16.10.10:80 check
    server web2 172.16.10.11:80 check
```

### Keepalived (`/etc/keepalived/keepalived.conf`) — op WEB1

```
vrrp_instance WEB_VIP {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 200
    virtual_ipaddress {
        172.16.10.100/24
        fd00:ac10:a::100/64
    }
}
```

> Op WEB2: zelfde config maar `state BACKUP` en `priority 100`.

---

## 6.5 Demo procedure

1. Start WEB1 en WEB2, verifieer Apache: `curl http://172.16.10.10`
2. Verifieer HAProxy VIP: `curl http://172.16.10.100` en `curl http://[fd00:ac10:a::100]`
3. Verifieer load balancing: meerdere requests wisselen af tussen WEB1 en WEB2
4. **Stop Apache op WEB1**: `systemctl stop apache2`
5. HAProxy detecteert dat WEB1 down is (health check), stuurt alles naar WEB2
6. `curl http://172.16.10.100` → altijd WEB2, geen onderbreking
7. Herstart Apache op WEB1 → load balancing hervat automatisch
