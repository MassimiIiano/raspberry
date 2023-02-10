# Proxy recap

## Was ist Squid?
Squid is an open-source proxy server software that is used for managing network traffic. It acts as an intermediary between client devices and the internet. Squid provides a range of features and functionalities that allow administrators to control and manage internet access for the devices on their network, improve network performance, enhance security, and monitor network traffic and usage patterns. 

## Was kann Squid?
- Intermediary between client devices and the internet
- Control and management of internet access for network devices
- Caching of frequently accessed websites for improved network performance
- Restrictions on internet usage, such as limiting access to certain websites or times
- Security features, such as a firewall, to prevent malicious traffic
- Flexible configuration and ability to integrate with other network services
- Effective for managing network traffic in small and large networks.

## Plane einen Proxy mit Squid für ein LAN (Dienste, Ziele). Starte dabei z.B. von der Situation: Ein Hotel stellt Internet über ein Kabel zur Verfügung, wir haben aber mindestens 2 Laptops und einen Switch. Verwende einen RPi als Proxy.

### Services:
- Squid proxy server
- DHCP server (for assigning IP addresses to devices in the LAN)
- Firewall (for security purposes)
- Network address translation (NAT)

### Goals:
- Provide internet access to multiple devices in the LAN
- Control and manage the internet usage of devices in the LAN
- Secure the LAN by filtering unwanted or malicious traffic
- Optimize network performance by caching frequently accessed websites
- Monitor network traffic and usage patterns

### Approach
- Set up the Raspberry Pi as the central device in the LAN and install the necessary services such as Squid, DHCP, firewall, and NAT.
- Configure the Squid proxy server to listen on a specific port and allow incoming connections from the LAN.
- Set up the firewall to restrict incoming and outgoing traffic based on predefined rules.
- Configure the DHCP server to assign IP addresses to devices in the LAN dynamically.
- Enable NAT to allow devices in the LAN to access the internet via the Raspberry Pi.
- Test the setup to ensure that the devices in the LAN can access the internet via the Squid proxy.
- Monitor the network traffic and usage patterns to fine-tune the configuration and ensure that the goals are being met.

## Installiere den Dienst und teste ihn. Hast du Zugriff?

### Installation

```bash
sudo apt update 
sudo apt install squid -y
```
### Enabling and Starting

```bash
sudo systemctl enable squid
sudo systemctl start squid
```

### Testing 

```bash
telnet localhost 3128
```

The service functions as intended and a different PC can use it as proxy and without any further configuration has access to the internet, just like it should be.

## Erlaube den Zugriff für genau einen PC.

The default configuration allows access to only one IP (localhost), you can replace it by creating an acs-element for a new IP in the network and allow it in the **/etc/squid/squid.conf** file

```bash
acs newip src 11.22.33.44
http_acces allow newip
```

It is also possible to not only specify one IP but also an IP-pool or network.

## Erlaube den Zugriff für ein gesamtes Netz.

Uncomment **http_acces allow localnet** at about 16% of the file (using **more** or **less**) in the  **/etc/squid/squid.conf** file.

You can also specify other networks and or IP-pools for access for other networks (also outside the local network).

## Definiere den Zugriff für die Zeitspanne von 10.25 Uhr bis 10.40 Uhr und von 14.30 Uhr bis 16.30 Uhr.
To set time
Weekdays:
- S - Sunday
- M - Monday
- T - Tuesday
- W - Wednesday
- H - Thursday
- F - Friday
- A - Saturday

```bash
acl timeframe_morning time M T W H F A S 10:25-10:40
acl timeframe_afternoon time M T W H F A S 14:30-16:30

http_access allow localnet timeframe_morning
http_access allow localnet timeframe_afternoon

http_access deny
```

## Wähle weitere Einstellungsmöglichkeiten mit Squid und konfiguriere sie.

### Cache-Management:

```bash
# Cache-Größe festlegen (in MB)
cache_mem 128 MB

# maximale Lebensdauer gecachter Inhalte festlegen (in Sekunden)
maximum_object_size_in_memory 512 KB

# Regeln für das Löschen von Inhalten aus dem Cache festlegen
cache_replacement_policy heap GDSF
```

### Authentifizierung:

```bash
# Authentifizierung aktivieren und eine spezifische Authentifizierungsmethode festlegen
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
acl authenticated_users proxy_auth REQUIRED
http_access allow authenticated_users
```

### Zugriffskontrolle:

```bash
# Zugriff auf bestimmte Websites beschränken
acl blocked_sites dstdomain "/etc/squid/blocked_sites.txt"
http_access deny blocked_sites

# Zugriff auf bestimmte IP-Adressen erlauben
acl allowed_ips src "/etc/squid/allowed_ips.txt"
http_access allow allowed_ips
```

### Logging:

```bash
# Log-Level festlegen
log_level 3

# Log-Format festlegen
log_format combined %>a %ui %un [%tl] "%rm %ru HTTP/%rv" %Hs %<st "%{Referer}>h" "%{User-Agent}>h" %Ss:%Sh

# Log-Datei festlegen
access_log /var/log/squid/access.log combined
```

### Netzwerkeinstellungen:

```bash
# Transparent-Proxy konfigurieren
http_port 3128 transparent

# Reverse-Proxy konfigurieren
cache_peer example.com parent 80 0 no-query default

# Interception-Proxy konfigurieren
http_port 8080 intercept
```

## Wie kannst Du Blacklists einstellen?

```bash
acl blacklist dstdomain "/etc/squid/blacklist.txt"
http_access deny all blacklist
```

blacklist.txt
```bash
"www.google.com"
"www.facebook.com"
```

## Was kann Squidguard?

- SquidGuard can be used to control the access of websites by users.
- It functions as a plugin for the Squid proxy server
- Squid can have blacklists, but they have to be manually configured.
- SquidGuard offers a centralized interface for defining the blacklists and integrates with Squid to enforce filtering rules.
- SquidGuard includes additional features such as authentication and logging not available in Squid alone.
- SquidGuard provides a more complete solution for URL filtering and access control.
