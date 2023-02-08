# Proxy recap   x

## Was ist Squid?
Squid is a Unix-based proxy server that caches Internet content closer to a requestor than its original point of origin.

## Was kann Squid?
It reduces bandwidth and improves response times by caching and reusing frequently-requested web pages

## Plane einen Proxy mit Squid für ein LAN (Dienste, Ziele). Starte dabei z.B. von der Situation: Ein Hotel stellt Internet über ein Kabel zur Verfügung, wir haben aber mindestens 2 Laptops und einen Switch. Verwende einen RPi als Proxy.
### targets:
- allow guests of hotel to use webpages
- have a fast connection
- have a secure connection

### services
- firewall for seciurity
- squid for cashing and improved speeds

## Installiere den Dienst und teste ihn. Hast du Zugriff?

### installation

```
sudo apt update 
sudo apt install squid -y
```
### enable and start

```
sudo systemctl enable squid
sudo systemctl start squid
```

### test 

```
telnet localhost 3128
```

## Erlaube den Zugriff für genau einen PC.

The defoult configuration allows acces to only one ip (localhost), you can replace it by breating an acs element for a new ip in the network and allow it in the **/etc/squid/squid.conf** file

```
acs newip src 11.22.33.44
http_acces allow newip
```

## Erlaube den Zugriff für ein gesamtes Netz.

uncomment **http_acces allow localnet** at mor or les 16% of the  **/etc/squid/squid.conf** document.

## Definiere den Zugriff für die Zeitspanne von 10.25 Uhr bis 10.40 Uhr und von 14.30 Uhr bis 16.30 Uhr.
to set time
weekdays:
- S. - Sunday
- M - Monday
- T - Tuesday
- W - Wednesday
- H - Thursday
- F - Friday
- A - Saturday

```
acl timeframe_morning time M T W H F A S 10:25-10:40
acl timeframe_afternoon time M T W H F A S 14:30-16:30

http_access allow localnet timeframe_morning
http_access allow localnet timeframe_afternoon

http_access deny
```

## Wähle weitere Einstellungsmöglichkeiten mit Squid und konfiguriere sie.

### Cache-Management:

```
# Cache-Größe festlegen (in MB)
cache_mem 128 MB

# maximale Lebensdauer gecachter Inhalte festlegen (in Sekunden)
maximum_object_size_in_memory 512 KB

# Regeln für das Löschen von Inhalten aus dem Cache festlegen
cache_replacement_policy heap GDSF
```

### Authentifizierung:

```
# Authentifizierung aktivieren und eine spezifische Authentifizierungsmethode festlegen
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
acl authenticated_users proxy_auth REQUIRED
http_access allow authenticated_users
```

### Zugriffskontrolle:

```
# Zugriff auf bestimmte Websites beschränken
acl blocked_sites dstdomain "/etc/squid/blocked_sites.txt"
http_access deny blocked_sites

# Zugriff auf bestimmte IP-Adressen erlauben
acl allowed_ips src "/etc/squid/allowed_ips.txt"
http_access allow allowed_ips
```

### Logging:

```
# Log-Level festlegen
log_level 3

# Log-Format festlegen
log_format combined %>a %ui %un [%tl] "%rm %ru HTTP/%rv" %Hs %<st "%{Referer}>h" "%{User-Agent}>h" %Ss:%Sh

# Log-Datei festlegen
access_log /var/log/squid/access.log combined
```

### Netzwerkeinstellungen:

```
# Transparent-Proxy konfigurieren
http_port 3128 transparent

# Reverse-Proxy konfigurieren
cache_peer example.com parent 80 0 no-query default

# Interception-Proxy konfigurieren
http_port 8080 intercept
```


## Wie kannst Du Blacklists einstellen?

```
acl blacklist dstdomain "/etc/squid/blacklist.txt"
http_access deny all blacklist
```
/blacklist.txt

```
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