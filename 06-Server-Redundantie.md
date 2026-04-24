# 6. Server-redundantie (DMZ)

[[00-Index|← Index]]

> **Niet in Packet Tracer** — gebruik GNS3  
> De dienst moet *dynamisch* en *high available* zijn.

---

## 6.1 Gekozen dienst en OS

**Dienst:** (HTTP / Mail / File / ...)  
**OS:** (Linux / Windows / ...)  
**Reden keuze:**  

---

## 6.2 Gekozen HA-oplossing

- [ ] Virtual IP + Load balancer
- [ ] Sync + shared storage

```
Server 1 ──┐
           ├── Virtual IP / Load balancer ── Clients
Server 2 ──┘
     ↕
   Sync
```

---

## 6.3 Configuratie

### Server 1

```bash
# Configuratie / script
```

### Server 2

```bash
# Configuratie / script
```

### Load balancer / Virtual IP

```bash
# Configuratie
```

---

## 6.4 Demo procedure

1. 
2. Stop de dienst op Server 1
3. Verifieer dat verkeer automatisch naar Server 2 gaat
4. Start Server 1 terug op en verifieer herstel
