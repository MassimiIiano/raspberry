# VPN 

**Aufgabenstellung:**

Erstellen Sie mit Hilfe der freien Software WireGuard eine verschlüsselte Verbindung über ein virtuelles privates Netzwerk (VPN) zwischen dem Raspberry (Server) und eurem Laptop (Peer). Eine VPN-Verbindung wird einfach durch den Austausch öffentlicher Schlüssel hergestellt, genau wie beim Austausch von SSH-Schlüsseln. Zuerst muss das Softwarepaket wireguard auf dem beiden Maschinen installliert werden. Konfigurieren Sie dann die Netzwerkinterfaces, die Schlüsselpaare, die Firewallregeln und die Konfigurationsdateien angemessen. Verwende dazu die bereitgestellten Kommandozeilenprogramme wg und wg-quick. Automatisieren Sie schlussendlich das Erstellen des VPN-Tunnels mit einem systemd-Dienst.

`sudo apt update && sudo apt upgrade`

`sudo apt install wireguard wg wg-quick`

in /etc/wireguard/wg0.conf

RPi
```bash
[Interface]
PrivateKey = 2PlkhUVQD9WOJW4Qzkd6oNDtjbeLd1GfV/A/6WjCp08=
Address = 172.16.1.1/24
ListenPort = 51820

[Peer]
PublicKey = FD8UEXOlXvv7QqX2kax+BhGVjboKfb90829RzMGpZxo=
AllowedIPs = 172.16.1.2/32
```

Laptop
```bash
[Interface]
PrivateKey = OJJGy7q8mGcBrtTq7Y7omGpYLUQmf/XHZ33TClOxhFM= 
Address = 172.16.1.2/24
ListenPort = 21841

[Peer]
PublicKey = toZTxyZamtoUlC9VQBIg2nXa0g/XXOjATMcqfje4hB0=
Endpoint = 10.216.220.213:51820
AllowedIPs = 0.0.0.0/0
```

use `wg-quick up wg0` on both RPi and Laptop
