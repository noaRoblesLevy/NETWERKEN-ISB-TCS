# 9. Bijlagen

[[00-Index|← Index]]

> Enkel relevante configuraties en scripts — geen standaard installatiestappen.

---

## 9.1 VyOS-A configuratie (VRRP Master)

```
system {
    host-name VyOS-A
    login {
        user vyos {
            authentication {
                encrypted-password "$6$rounds=656000$..."
            }
            level admin
        }
    }
    service {
        ssh {
            port 22
        }
    }
}
interfaces {
    ethernet eth0 {
        address 10.10.0.2/24
        address fd00:a:a::2/64
        description LAN
        ipv6 {
            router-advert {
                prefix fd00:a:a::/64 {
                }
                send-advert true
            }
        }
    }
    loopback lo {
    }
}
high-availability {
    vrrp {
        group LAN {
            interface eth0
            priority 200
            address 10.10.0.1/24
            vrid 10
        }
        group LAN6 {
            interface eth0
            priority 200
            address fd00:a:a::1/64
            vrid 16
        }
    }
}
```

---

## 9.2 VyOS-B configuratie (VRRP Backup)

```
system {
    host-name VyOS-B
}
interfaces {
    ethernet eth0 {
        address 10.10.0.3/24
        address fd00:a:a::3/64
        description LAN
    }
    loopback lo {
    }
}
high-availability {
    vrrp {
        group LAN {
            interface eth0
            priority 100
            address 10.10.0.1/24
            vrid 10
        }
        group LAN6 {
            interface eth0
            priority 100
            address fd00:a:a::1/64
            vrid 16
        }
    }
}
```

---

## 9.3 OpenVPN server-configuratie (`/etc/openvpn/server.conf`)

```
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
server 10.8.0.0 255.255.255.0
push "route 10.10.0.0 255.255.255.0"
keepalive 10 120
persist-key
persist-tun
status /var/log/openvpn-status.log
verb 3
```

---

## 9.4 OpenVPN client-configuratie (`client1-full.ovpn`)

```
client
dev tun
proto udp
remote 10.10.0.20 1194
resolv-retry infinite
nobind
persist-key
persist-tun
verb 3
<ca>
... (CA-certificaat — gegenereerd via Easy-RSA)
</ca>
<cert>
... (clientcertificaat — gegenereerd via Easy-RSA)
</cert>
<key>
... (clientsleutel — gegenereerd via Easy-RSA)
</key>
```

---

## 9.5 HAProxy configuratie (`/etc/haproxy/haproxy.cfg`)

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

---

## 9.6 Keepalived configuratie

### WEB1 — MASTER (`/etc/keepalived/keepalived.conf`)

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

---

## 9.7 lsyncd configuratie (`/etc/lsyncd/lsyncd.conf.lua`)

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

---

## 9.8 Netwerkschema en Packet Tracer

- Netwerkschema: zie `netwerkschema.drawio` in de repository
- Packet Tracer simulatie: zie `KTN-Netwerk-Architectuur.pkt` in de repository
