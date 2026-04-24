# 5. Router-redundantie

[[00-Index|← Index]]

> **Niet in Packet Tracer** — gebruik GNS3 + VyOS  
> Doel: als één router uitvalt, blijven clients bereikbaar via de andere.

---

## 5.1 Gekozen oplossing

- [ ] Virtual IP (VRRP) — gateway voor clients
- [ ] Load balancer

**Motivatie keuze:**

---

## 5.2 Minimum setup

```
Router A (master) ──┐
                    ├── Virtual IP (client gateway)
Router B (backup) ──┘
```

**Virtual IP:**  
**Router A IP:**  
**Router B IP:**  

---

## 5.3 Configuratie

### Router A

```
# VyOS VRRP configuratie
```

### Router B

```
# VyOS VRRP configuratie
```

---

## 5.4 Demo procedure

1. 
2. Stop Router A
3. Verifieer dat Virtual IP overgenomen wordt door Router B
4. Ping/verbinding blijft actief
