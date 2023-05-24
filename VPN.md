# Dokumentation Laborübung WireGuard

## Index:

<!-- TODO make index -->

## Aufgabenstellung:

Erstellen Sie mit Hilfe der freien Software WireGuard eine verschlüsselte Verbindung über ein virtuelles privates Netzwerk (VPN) zwischen dem Raspberry (Server) und eurem Laptop (Peer). Eine VPN-Verbindung wird einfach durch den Austausch öffentlicher Schlüssel hergestellt, genau wie beim Austausch von SSH-Schlüsseln. Zuerst muss das Softwarepaket wireguard auf dem beiden Maschinen installliert werden. Konfigurieren Sie dann die Netzwerkinterfaces, die Schlüsselpaare, die Firewallregeln und die Konfigurationsdateien angemessen. Verwende dazu die bereitgestellten Kommandozeilenprogramme wg und wg-quick. Automatisieren Sie schlussendlich das Erstellen des VPN-Tunnels mit einem systemd-Dienst.

## Verwendete Werkzeuge:

- RaspberryPi
- laptop (with linux distro)
- wireguard package
- Text editor (eg. vim, nano)

## Grundlagen Theorie:

## Versuchsdurchführung:

<!-- TODO write a summary of the versuchsdurchfuhrung -->

### 1. Installation von WireGuard:

Man installiert WireGuard auf `beiden` Maschinen

```bash
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install wireguard
```

### 2. Schlüssel Generieren:

man fürt folgenden Befehl auf `beiden` Machienen aus, um die `Schlüsselpaare` zu generieren

```bash
wg genkey | sudo tee /etc/wireguard/priv.key | wg pubkey | sudo tee /etc/wireguard/pub.key
```

Dieser Befehl generiert ein neues `Schlüsselpaar`, speichert den privaten Schlüssel in `/etc/wireguard/priv.key` und den öffentlichen Schlüssel in `/etc/wireguard/pub.key` ab.

### 3. Konfiguration der Netzwerkinterfaces:

#### RaspberryPi konfiguration `/etc/wireguard/wg0.conf`
```bash
[Interface]
Address = 10.216.220.213/24                                 # RPi Address
ListenPort = 51820                                          # wireguard standart Port
PrivateKey = 2PlkhUVQD9WOJW4Qzkd6oNDtjbeLd1GfV/A/6WjCp08=   # RPi Private Key

[Peer]
PublicKey = FD8UEXOlXvv7QqX2kax+BhGVjboKfb90829RzMGpZxo=    # Laptop Public Key
AllowedIPs = 172.16.1.3/24                                  # Laptop IP-Adresse
```
Zu beachten im `[interface]` Ausschnitt:

- `Address` ist die IP-Adresse, die dem WireGuard-Interface auf diesem Gerät zugewiesen ist. In diesem Fall hat das Raspberry Pi die Adresse 10.216.220.213 innerhalb des VPN-Netzwerks. 
- `ListenPort` ist der Port, auf dem das WireGuard-Interface auf eingehende Verbindungen hört. Der Standardport für WireGuard ist 51820.
- `PrivateKey` ist der private Schlüssel des Raspberry Pi in diesem VPN.

Zu beachten im `[peer]` Ausschnitt:

- `PublicKey` ist der öffentliche Schlüssel des Peers, in diesem Fall des Laptops
- `AllowedIPs` ist eine Liste von IP-Adressen oder Netzwerkbereichen, die über den VPN-Tunnel an diesen Peer weitergeleitet werden dürfen.

####  Laptop konfiguration `/etc/wireguard/wg0.conf`

```bash
[Interface]
Address =  172.16.1.3/24                                    # Laptop IP-Adresse
PrivateKey = gKD/oBwpPxWquoaw1kEwYypxVZ5N5Wu/EK/TsUJoFmw=   # Laptop Private Key

[Peer]
PublicKey = toZTxyZamtoUlC9VQBIg2nXa0g/XXOjATMcqfje4hB0=    # RPi Public Key
AllowedIPs = 10.216.220.213/24                              # RPi IP-Ardesse   
Endpoint = 10.216.220.213:51820
```

### 4. Konfiguration der Firewallregeln

#### Firewall Regeln `/etc/sysctl.conf`:
```bash
# Auskommentieren
# net.ipv4.ip_forward 
```

Kommentiere die Zeile `net.ipv4.ip_forward` im file `sysctl.conf` aus. Anschließsend lade die veränderten `sysctl` regeln durch den befehl `sudo sysctl -p`.

#### Datenverkehr über WireGuard erlauben
```bash
# Erlaubt eingehende Verbindungen, die bereits etabliert sind oder sich auf eine bestehende Verbindung beziehen.
sudo iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

# Erlaubt weitergeleitete Verbindungen, die bereits etabliert sind oder sich auf eine bestehende Verbindung beziehen.
sudo iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

# Erlaubt neue eingehende UDP-Verbindungen auf Port 51820.
sudo iptables -A INPUT -p udp -m udp --dport 51820 -m conntrack --ctstate NEW -j ACCEPT

#  Erlaubt neue weitergeleitete Verbindungen, die sowohl auf dem WireGuard-Interface wg0 eingehen als auch von dort ausgehen.
sudo iptables -A FORWARD -i wg0 -o wg0 -m conntrack --ctstate NEW -j ACCEPT

# Ersetzt die Quell-IP-Adresse von Paketen, die aus dem angegebenen Subnetz kommen und über das eth0-Interface ausgehen, durch die IP-Adresse des eth0-Interface (dies wird als "Masquerading" bezeichnet und ist eine Form der NAT - Network Address Translation).
sudo iptables -t nat -A POSTROUTING -s 
```

### 5. Automatisation

Um sicherzustellen, dass der VPN-Tunnel bei jedem Start des Systems automatisch aufgebaut wird, kann man einen systemd-Dienst erstellen. Auf beiden Maschinen führen Sie die folgenden Befehle aus:

```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```


## Fazit

<!-- TODO fazit -->
<!-- 




## Konfigurationsdateien übertragen

`sudo wg-quick up wg0`

`sudo wg-quick down wg0`

## Automatisierung mit systemd-Dienst

`sudo vim /etc/systemd/system/wg-quick@wg0.service`

```bash
[Unit]
Description=WireGuard VPN tunnel via wg-quick(8) for %i
After=network-online.target nss-lookup.target
Wants=network-online.target nss-lookup.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/wg-quick up %i
ExecStop=/usr/bin/wg-quick down %i

[Install]
WantedBy=multi-user.target
```

`sudo systemctl start wg-quick@wg0.service`

`sudo systemctl status wg-quick@wg0.service`

für start nach neustart
`sudo systemctl enable wg-quick@wg0.service`

## Testen der VPN-Verbindung

`sudo wg`

`sudo wg show`

`sudo wg showconf wg0`

`sudo wg-quick down wg0`

`sudo wg-quick up wg0`

## Referenzen
- https://www.wireguard.com/
- https://www.wireguard.com/quickstart/ -->
