# RPi-

## Aufgabenstellung Öffentliches Interface
Jede Gruppe schließt ihren RPi an das LAN des Systeme-Labors (= RPi-Netz) an und vergibt diesem "öffentlichen" Interface die IP 10.0.0.x/24 (wobei x der ID der Gruppe bzw. der RPI-Nummer entspricht; die Gruppen gehen von 1 bis 7). Der Webserver jedes RPis soll über diese öffentliche IP erreichbar sein.

1. Um eine statische IP-Addresse zu bekommen muss man folgende Datei bearbeiten:

<h5 a><strong><code>/etc/network/interfaces</code></strong></h5>

```bash
auto eth0
iface eth0 inet static
    address 10.0.0.6
    netmask 255.255.255.0
    network 10.0.0.0
    broadcast 10.0.0.255
```

Dabei muss geschaut werden, dass der networking.service läuft und der dhcpcd.service nicht die IP-Adressen überschreibt. Mit dem Command `ip a` kann man nachschauen, ob die IP-Adresse richtig vergeben wurde und ob das Interface auch aktiv ist.  

Es ist jedoch wichtig zu beachten, dass diese Methode für neuere Versionen von Debian/Raspbian (ab Version 9 Stretch) nicht mehr empfohlen wird. Stattdessen wird das Programm dhcpcd verwendet, um eine statische IP-Adresse zu konfigurieren. Weitere Informationen dazu finden Sie in der offiziellen Dokumentation von Raspbian: https://www.raspberrypi.org/documentation/configuration/tcpip/, dies ist auch in der verwendeten Version möglich. 

Dabei muss man folgende Datei konfigurieren:
<h5 a><strong><code>/etc/dhcpcd.conf</code></strong></h5>

```bash
interface eth0
static ip_address=10.0.0.6/24
...
```

2. Dann kann mit ping ein anderer Raspberry gepingt werden und man sieht, dass das Interface eth0 aktiv ist und man sich im RPi-Netz befindet.

Zusätzlich sollte geschaut werden, ob die Firewalleinstellungen von iptables nicht das RPI-net verweigert, denn wenn iptables das RPI-net verweigert kann ein anderer Raspberry im RPI-net nicht den eigenen Raspberry erreichen. Man kann mit `sudo iptables -F && sudo iptables -P INPUT ACCEPT` für die jetzige Session am Raspberry den ganzen Eingang aktzeptieren. Dann kann man auch testen, ob ein anderer Raspberry den eigenen erreichen kann, aber man sollte die Firewalleinstellungen selbst so anpassen, wie sie gewollt sein, damit auch das Interface so funktioniert, wie es funktionieren soll - und das sicherheitsstechnisch in Ordnung. Der vorherig genannte Command ist also nur für Testzwecke in der jeweiligen Session gut. Um das sicher und über Sessions unabhängig zu konfigurieren, kann man, wenn man den folgenden iptables Befehl zu der Konfiguration hinzufügt:
`sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT` Diese Regel öffnet den Port 80 für eingehende TCP-Verbindungen.

Es sollte jedoch beachtet werden, dass neuere Versionen von Raspbian standardmäßig die Firewall ufw verwenden. Daher ist es ratsam, ufw zu konfigurieren, um sicherzustellen, dass der Webserver des RPi über das öffentliche Interface erreichbar ist. Weitere Informationen dazu finden Sie in der offiziellen Dokumentation von Raspbian: https://www.raspberrypi.org/documentation/configuration/security-firewall.md, diese Konfiguration war in dieser Version jedoch nicht notwendig.

Schließlich sollte beachtet werden, dass es für den Betrieb eines Webservers auf einem RPi empfehlenswert ist, SSL (Secure Sockets Layer) zu aktivieren, um die Verbindung zwischen dem Client und dem Server zu verschlüsseln. Dies kann durch die Installation von certbot und die Konfiguration von nginx als Reverse Proxy erreicht werden. Weitere Informationen dazu finden Sie in der offiziellen Dokumentation von Raspbian: https://www.raspberrypi.org/documentation/remote-access/web-server/nginx.md, dies wurde jedoch nicht vervollständigt.

3. Dann muss der Apache konfiguriert werden, damit er auch über die öffentliche IP-Adresse erreichbar ist. Hierzu muss die entsprechende Konfigurationsdatei bearbeitet werden:

<h5 a><strong><code>/etc/apache2/apache2.conf</code></strong></h5>

```bash
Listen 10.0.0.6
```

4. Der Apache muss dann gestartet werden mit: `sudo systemctl start apache2`. Läuft der Apache schon, kann man das mit `sudo systemctl status apache2` kontrollieren.

## Aufgabenstellung Privates Netz
Der RPi erhält dann per WLAN-AccessPoint (oder USB/Ethernet Adapter) ein zweites, "privates" Interface, auf dem ein Netz aus dem Bereich 172.16.0.0/12 oder 192.168.0.0/16 konfiguriert wird.
Der RPi selbst und ein angeschlossener Laptop erhalten jeweils eine IP in diesem Netz. Dieses private Netz soll nicht von außen erreichbar sein.

1. hostapd mittels `sudo apt install hostapd` installieren.
2. Um ein privates Netz auf dem Interface wlan0 laufen zu lassen muss folgende Datei bearbeitet werden: 

<h5 a><strong><code>/etc/network/interfaces</code></strong></h5>

```bash
auto wlan0
iface wlan0 inet static
    address 192.168.1.1
    netmask 255.255.255.0
    network 192.168.1.0
    broadcast 192.168.1.255
```

Hier kann man wiederum mit `ip a` schauen, ob das Interface richtig gesetzt wurde. Genau wie die Firewalleinstellungen und die Prozesse, welche eventuell interferieren müssen überprüft werden (genau wie bei der vorherigen Übung).

3. Dann muss die Konfigurationsdatei von hostapd bearbeitet werden:

<h5 a><strong><code>/etc/hostapd/hostapd.conf</code></strong></h5>

```bash
interface=wlan0 # das interface
driver=nl80211 
ssid=Raspberry_6
hw_mode=g
channel=11 # kann geändert werden, falls andere channels belegt sind
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2 # kann auch auf wpa3 gesetzt werden
wpa_passphrase=... # das passwort in klartext und ohne anführungszeichen: wpa_passphrase=my_super_secret_password
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP   
```

4. Dann muss man mit `sudo /sbin/hostapd /etc/hostapd/hostapd.conf` den hostapd ausführen. 

Wenn die Applikation ohne Fehler läuft, dann kann man sich mit einem Gerät, zum Beispiel dem Laptop aber auch mit dem Smartphone sich mit dem Raspberry verbinden. Man muss aber wieder aufpassen und eine statische Konfiguration auszuwählen, um sich auch mit dem Raspberry verbinden zu können, da hostapd nicht einen DHCP-Server enthält (zumindest die verwendete Version (2.9), neuere Versionen eventuell haben einen DHCP-Server). Hat man sich dann verbunden, kann man innerhalb des Netzes 192.168.1.1 agieren. So kann man auch zwei Geäte mit dem AP verbinden und sich gegenseitig pingen.

5. Möchte man jedoch trotzdem einen eigenen DHCP-Server einrichten, so kann man das folgendermaßen tun:
    1. Man muss das Programm dnsmasq installieren: `sudo apt install dnsmasq`
    2. Die Konfigurationsdatei `/etc/dnsmasq.conf` bearbeiten:
            ```bash
            interface=wlan0
            dhcp-range=192.168.1.100,192.168.1.200,255.255.255.0,12h
            ```
        Somit gibt es jetzt auf dem Interface einen DHCP-Server, um Adressen im Bereich von 192.168.1.100 bis 192.168.1.200 mit einer Lease-Zeit von 12 Stunden zu vergeben.
    3. Dann muss dnsmasq gestartet werden mit: `sudo systemctl start dnsmasq`

6. Damit das private Netz nicht von außen erreichbar ist (also vom Netz auf eth0 - dem "Internet") muss folgende iptables-Regel definiert werden:

`sudo iptables -A FORWARD -i eth0 -o wlan0 -j DROP`

Somit wird alles, was von eth0 nach wlan0 weitergeleitet werden soll gedroppt. 

## Aufgabenstellung Routing
Konfiguriert den RPi so, dass er für den Laptop als Gateway zum RPi-Netz arbeitet. Versucht nun vom Laptop aus auf die Webserver (RPi) der anderen Gruppen zuzugreifen - ihr werdet sehen, dass dies trotz des Gateways nicht möglich ist. Findet heraus warum, und dokumentiert!

## PAT (Masuerading)
Konfiguriert als erste Lösung für das obige Problem PAT (Masquerading). Testet und dokumentiert.

1. Zuerst muss man sicherstellen, dass die IPv4-Weiterleitung aktiviert ist. Das kann man tun, indem man den folgenden Befehl ausführt:

`echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward`

Man sollte außerdem sicherstellen, dass keine iptables-Regeln den Verkehr von wlan0 zu eth0 blockieren.

2. Auf dem Gerät, das mit wlan0 verbunden ist, muss man dann die IP-Adresse des Raspberry Pi auf wlan0 als Standard-Gateway festlegen. Dies kann in den Netzwerkeinstellungen des Geräts erfolgen.

Sobald diese Schritte abgeschlossen sind, sollte das Gerät auf wlan0 in der Lage sein, über den Raspberry Pi auf das Netzwerk auf eth0 zuzugreifen.

Ohne NAT (Network Address Translation) oder PAT (Port Address Translation) können Geräte in einem privaten Netzwerk nicht direkt mit Geräten in einem anderen Netzwerk kommunizieren. Dies liegt daran, dass private IP-Adressen nicht im Internet geroutet werden können. Um die Kommunikation zwischen den beiden Netzwerken zu ermöglichen, muss der Verkehr von einem privaten Netzwerk zu einem anderen über ein Gateway geleitet werden, das NAT oder PAT verwendet.

In Ihrem Fall ist der Raspberry Pi das Gateway zwischen dem lokalen Netzwerk und dem RPiNet. Ohne NAT oder PAT auf dem Raspberry Pi können Geräte im lokalen Netzwerk nicht direkt mit Geräten im RPiNet kommunizieren. Durch die Konfiguration von NAT oder PAT auf dem Raspberry Pi wird der Verkehr von einem privaten Netzwerk zum anderen umgeschrieben, so dass die Kommunikation möglich wird.

Man kann dieses Problem lösen, indem man mit NAT beziehungsweise mit PAT arbeitet.

3. Dazu muss man die iptables-Regeln auf dem Raspberry Pi konfigurierern, um den Verkehr von wlan0 (lokales Netzwerk) zu eth0 (RPiNet) zu NATen (PATen):

```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

Es wird hier also durch `MASQUERADE` dynamisches NAT (beziehungsweise PAT) benutzt.

Es ist wichtig zu beachten, dass die Konfiguration von NAT eine potenzielle Sicherheitslücke darstellen kann, da die internen IPs des Netzwerks gegenüber dem öffentlichen Internet sichtbar sind. Es ist daher ratsam, entsprechende Firewall-Regeln zu implementieren, um unerwünschte Zugriffe zu verhindern.

Nachdem diese Schritte abgeschlossen wurden, sollte der Laptop im lokalen Netzwerk des Raspberry Pi in der Lage sein, über den Raspberry Pi auf das RPiNet zuzugreifen.

## NAT 
Über NAT (nicht PAT) wird nun den privaten IPs (RPi und Laptop) jeweils eine weitere öffentliche IP "zugeteilt". Diese IPs haben den Prefix y.0.0.0/24 (y = 10 + ID der Gruppe).
Versucht nun erneut, vom internen Netz aus die Webseite der anderen Gruppen zu erreichen. Wieder werdet ihr merken, dass es nicht auf Anhieb funktioniert - denkt an die Rückroute der Pakete! Testet und dokumentiert. Versucht, sowohl statisches als auch dynamisches NAT zu konfigurieren!

Dazu muss man wie im obigen Problem auch schon erläutert die iptables-Regeln auf dem Raspberry Pi konfigurieren und NAT hinzufügen. 

Hier ist die Lösung mit statischem NAT:

```bash
sudo iptables -t nat -A PREROUTING -d y.0.0.0/24 -j DNAT --to-destination 192.168.1.x
sudo iptables -t nat -A POSTROUTING -s 192.168.1.x -j SNAT --to-source y.0.0.0
```

Hier ist die Lösung mit dynamischem NAT:

```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Die Rückroute muss ebenfalls konfiguriert werden, damit Antworten von Geräten im RPiNet zurück zum lokalen Netzwerk geleitet werden können. Dies geschieht durch die Konfiguration von iptables-Regeln auf dem Raspberry Pi, die den Verkehr von eth0 (RPiNet) zu wlan0 (lokales Netzwerk) umschreiben.

Dabei ist y 16 (10 + ID der Gruppe) und x die die IP-Adresse des Geräts im lokalen Netzwerk.

## Spezieller Dienst
Konfiguriert nun in eurem privaten Netz (auf dem privaten Interface des RPi oder auf dem Laptop) einen weiteren Dienst (z.B. FTP). Dieser Dienst soll ausschließlich über eure öffentliche Gruppen-IP (y.0.0.0/24) erreichbar sein.

Um einen FTP-Dienst in einem privaten Netzwerk zu konfigurieren und ihn ausschließlich über eine öffentliche Gruppen-IP erreichbar zu machen, kann man die folgenden Schritte ausführen:

1. Installation einens FTP-Servers auf dem Raspberry Pi oder dem Laptop im privaten Netzwerk: 
`sudo apt-get install vsftpd`

2. Konfiguration des FTP-Servers gemäß den Anforderungen. Die Konfigurationsdatei für vsftpd befindet sich unter:  `/etc/vsftpd.conf`

Es muss sichergestellt werden, dass der FTP-Server gestartet ist und ordnungsgemäß funktioniert: 
`sudo systemctl start vsftpd`

3. Dann muss man die iptables-Regeln auf dem Raspberry Pi konfigurieren, um den FTP-Verkehr von der öffentlichen Gruppen-IP (y.0.0.0/24) zur privaten IP-Adresse umzuleiten:

```bash
sudo iptables -t nat -A PREROUTING -p tcp -d y.0.0.0/24 --dport 21 -j DNAT --to-destination 192.168.1.x:21
```

Dabei ist y 10 + ID der Gruppe und x die IP-Adresse des Geräts im privaten Netzwerk, auf dem der FTP-Server ausgeführt wird.

Nachdem diese Schritte abgeschlossen wurden, sollte der FTP-Dienst ausschließlich über die öffentliche Gruppen-IP erreichbar sein.
