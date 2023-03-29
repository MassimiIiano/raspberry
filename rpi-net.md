# RPi-

## Aufgabenstellung Öffentliches Interface
Jede Gruppe schließt ihren RPi an das LAN des Systeme-Labors (= RPi-Netz) an und vergibt diesem "öffentlichen" Interface die IP 10.0.0.x/24 (wobei x der ID der Gruppe bzw. der RPI-Nummer entspricht; die Gruppen gehen von 1 bis 7). Der Webserver jedes RPis soll über diese öffentliche IP erreichbar sein.

1. Um eine statische IP-Addresse zu bekommen muss man folgende Datei bearbeiten:

<h5 a><strong><code>/etc/network/interfaces</code></strong></h5>

```
auto eth0
iface eth0 inet static
    address 10.0.0.6/24
    netmask 255.255.255.0
    network 10.0.0.0
    broadcast 10.0.0.255
```

Dabei muss geschaut werden, dass der networking.service läuft und der dhcpcd.service nicht die IP-Adressen überschreibt. Mit dem Command `ip a` kann man nachschauen, ob die IP-Adresse richtig vergeben wurde und ob das Interface auch aktiv ist.  

2. Dann kann mit ping ein anderer Raspberry gepingt werden und man sieht, dass das Interface eth0 aktiv ist und man sich im RPi-Netz befindet.

Zusätzlich sollte geschaut werden, ob die Firewalleinstellungen von iptables nicht das RPI-net verweigert, denn wenn iptables das RPI-net verweigert kann ein anderer Raspberry im RPI-net nicht den eigenen Raspberry erreichen. Man kann mit `sudo iptables -F && sudo iptables -P INPUT ACCEPT` für die jetzige Session am Raspberry den ganzen Eingang aktzeptieren. Dann kann man auch testen, ob ein anderer Raspberry den eigenen erreichen kann, aber man sollte die Firewalleinstellungen selbst so anpassen, wie sie gewollt sein, damit auch das Interface so funktioniert, wie es funktionieren soll - und das sicherheitsstechnisch in Ordnung. Der vorherig genannte Command ist also nur für Testzwecke in der jeweiligen Session gut.

## Aufgabenstellung Privates Netz
Der RPi erhält dann per WLAN-AccessPoint (oder USB/Ethernet Adapter) ein zweites, "privates" Interface, auf dem ein Netz aus dem Bereich 172.16.0.0/12 oder 192.168.0.0/16 konfiguriert wird.
Der RPi selbst und ein angeschlossener Laptop erhalten jeweils eine IP in diesem Netz. Dieses private Netz soll nicht von außen erreichbar sein.

1. hostapd mittels `sudo apt install hostapd` installieren.
2. Um ein privates Netz auf dem Interface wlan0 laufen zu lassen muss folgende Datei bearbeitet werden: 

<h5 a><strong><code>/etc/network/interfaces</code></strong></h5>

```
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

```
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

5. Damit das private Netz nicht von außen erreichbar ist (also vom Netz auf eth0 - dem "Internet") muss folgende iptables-Regel definiert werden:

`sudo iptables -A FORWARD -i eth0 -o wlan0 -j DROP`

Somit wird alles, was von eth0 nach wlan0 weitergeleitet werden soll gedroppt. 

## Aufgabenstellung Routing
Konfiguriert den RPi so, dass er für den Laptop als Gateway zum RPi-Netz arbeitet. Versucht nun vom Laptop aus auf die Webserver (RPi) der anderen Gruppen zuzugreifen - ihr werdet sehen, dass dies trotz des Gateways nicht möglich ist. Findet heraus warum, und dokumentiert!

?? Überprüfen:

1. Zuerst muss man sicherstellen, dass die IPv4-Weiterleitung aktiviert ist. Das kann man tun, indem man den folgenden Befehl ausführt:

`echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward`

Man sollte außerdem sicherstellen, dass keine iptables-Regeln den Verkehr von wlan0 zu eth0 blockieren.

2. Auf dem Gerät, das mit wlan0 verbunden ist, muss man dann die IP-Adresse des Raspberry Pi auf wlan0 als Standard-Gateway festlegen. Dies kann in den Netzwerkeinstellungen des Geräts erfolgen.

Sobald diese Schritte abgeschlossen sind, sollte das Gerät auf wlan0 in der Lage sein, über den Raspberry Pi auf das Netzwerk auf eth0 zuzugreifen.

## PAT (Masuerading)
Konfiguriert als erste Lösung für das obige Problem PAT (Masquerading). Testet und dokumentiert.

??

## NAT 
Über NAT (nicht PAT) wird nun den privaten IPs (RPi und Laptop) jeweils eine weitere öffentliche IP "zugeteilt". Diese IPs haben den Prefix y.0.0.0/24 (y = 10 + ID der Gruppe).
Versucht nun erneut, vom internen Netz aus die Webseite der anderen Gruppen zu erreichen. Wieder werdet ihr merken, dass es nicht auf Anhieb funktioniert - denkt an die Rückroute der Pakete! Testet und dokumentiert. Versucht, sowohl statisches als auch dynamisches NAT zu konfigurieren!

??

## Spezieller Dienst
Konfiguriert nun in eurem privaten Netz (auf dem privaten Interface des RPi oder auf dem Laptop) einen weiteren Dienst (z.B. FTP). Dieser Dienst soll ausschließlich über eure öffentliche Gruppen-IP (y.0.0.0/24) erreichbar sein.

??
