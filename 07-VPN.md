  # 7. VPN-oplossing

  [[00-Index|← Index]]

  > **Niet in Packet Tracer** — gebruik VirtualBox + Debian Linux
  > Oplossing voor **meerdere clients** (NIET point-to-point)
  > Clients moeten het interne netwerk bereiken via een beveiligde tunnel

  ---

  ## 7.1 Gekozen VPN-technologie

  - [x] **OpenVPN**

  **Motivatie:** OpenVPN is battle-tested, ondersteunt meerdere
  gelijktijdige clients,
  werkt op UDP/TCP, en heeft clients voor alle besturingssystemen.
  Authenticatie
  gebeurt via PKI-certificaten (geen wachtwoorden).

  ---

  ## 7.2 Architectuur

  De VPN-server (SRV-VPN) staat in de DMZ en is bereikbaar vanaf het
  internet
  via poort 1194 UDP. Verbonden clients krijgen een tunnel-IP uit de
  10.8.0.0/24
  pool en kunnen zo het interne netwerk bereiken.

  [Externe client]
         |
    OpenVPN tunnel (UDP 1194)
         |
  [SRV-VPN: debian2 — 10.10.0.20]
         |
  [Intern netwerk: 10.10.0.0/24]

  | Rol | IP |
  |-----|----|
  | SRV-VPN (server) | 10.10.0.20/24 |
  | VPN tunnel pool (clients) | 10.8.0.0/24 |
  | Client tunnel-IP (voorbeeld) | 10.8.0.6 |

  ---

  ## 7.3 PKI — Certificaten via Easy-RSA

  bash
  make-cadir ~/openvpn-ca
  cd ~/openvpn-ca
  ./easyrsa init-pki
  ./easyrsa build-ca nopass          # CA aanmaken → Common Name: KTN-CA
  ./easyrsa gen-req server nopass    # Serversleutel
  ./easyrsa sign-req server server   # Servercertificaat ondertekenen
  ./easyrsa gen-dh                   # Diffie-Hellman (2048-bit)
  ./easyrsa gen-req client1 nopass   # Clientsleutel
  ./easyrsa sign-req client client1  # Clientcertificaat ondertekenen

  ┌─────────────────────────┬───────────────────────────┐
  │         Bestand         │           Doel            │
  ├─────────────────────────┼───────────────────────────┤
  │ pki/ca.crt              │ CA-certificaat            │
  ├─────────────────────────┼───────────────────────────┤
  │ pki/issued/server.crt   │ Servercertificaat         │
  ├─────────────────────────┼───────────────────────────┤
  │ pki/private/server.key  │ Serversleutel             │
  ├─────────────────────────┼───────────────────────────┤
  │ pki/dh.pem              │ Diffie-Hellman parameters │
  ├─────────────────────────┼───────────────────────────┤
  │ pki/issued/client1.crt  │ Clientcertificaat         │
  ├─────────────────────────┼───────────────────────────┤
  │ pki/private/client1.key │ Clientsleutel             │
  └─────────────────────────┴───────────────────────────┘

  ---
  7.4 Server-configuratie (/etc/openvpn/server.conf)

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

  Server starten:
  sudo systemctl start openvpn@server
  sudo systemctl enable openvpn@server

  ---
  7.5 Client-configuratie (client1-full.ovpn)

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
  ... (CA certificaat)
  </ca>
  <cert>
  ... (clientcertificaat)
  </cert>
  <key>
  ... (clientsleutel)
  </key>

  Client-software: OpenVPN GUI (Windows)

  ---
  7.6 Demo resultaten

  ┌─────────────────────────────────────┬───────────────────────────────┐
  │                Test                 │           Resultaat           │
  ├─────────────────────────────────────┼───────────────────────────────┤
  │ OpenVPN server status               │ active (running) ✅           │
  ├─────────────────────────────────────┼───────────────────────────────┤
  │ Client verbinding (Windows PC)      │ Verbonden, tunnel IP 10.8.0.6 │
  │                                     │  ✅                           │
  ├─────────────────────────────────────┼───────────────────────────────┤
  │ Ping 10.8.0.1 (server tunnel-IP)    │ 0% packet loss ✅             │
  ├─────────────────────────────────────┼───────────────────────────────┤
  │ Intern netwerk bereikbaar           │ Via VPN tunnel ✅             │
  │ (10.10.0.20)                        │                               │
  ├─────────────────────────────────────┼───────────────────────────────┤
  │ Ping zonder VPN verbinding          │ Request timed out ✅          │
  └─────────────────────────────────────┴───────────────────────────────┘

  ---
