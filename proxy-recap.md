# Proxy recap

## Was ist Squid?
Squid is a Unix-based proxy server that caches Internet content closer to a requestor than its original point of origin.

## Was kann Squid?
It reduces bandwidth and improves response times by caching and reusing frequently-requested web pages

## Plane einen Proxy mit Squid für ein LAN (Dienste, Ziele). Starte dabei z.B. von der Situation: Ein Hotel stellt Internet über ein Kabel zur Verfügung, wir haben aber mindestens 2 Laptops und einen Switch. Verwende einen RPi als Proxy.
#### targets:
- allow guests of hotel to use webpages
- have a fast connection
- have a secure connection

### services
- firewall for seciurity
- squid for cashing and improved speeds

## Installiere den Dienst und teste ihn. Hast du Zugriff?
#### installation
> wpa_supplicant makes chrush networking adn apt update dosn't work;
> try to write script in init.d;
> for that i have to install chkconfig;
> that still dosn't work;
> write a service for systemctl to write defoult rules;
> as of now the service is stll failing, but at least i can install squid;
```
sudo apt update 
sudo apt install squid -y
```
#### enable and start
```
sudo systemctl enable squid
sudo systemctl start squid
```

#### test 
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
## Wähle weitere Einstellungsmöglichkeiten mit Squid und konfiguriere sie.
## Wie kannst Du Blacklists einstellen?
## Was kann Squidguard?
