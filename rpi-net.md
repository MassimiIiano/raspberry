# RPi-

## Aufgabenstellung Öffentliches Interface
Jede Gruppe schließt ihren RPi an das LAN des Systeme-Labors (= RPi-Netz) an und vergibt diesem "öffentlichen" Interface die IP 10.0.0.x/24 (wobei x der ID der Gruppe bzw. der RPI-Nummer entspricht; die Gruppen gehen von 1 bis 7). Der Webserver jedes RPis soll über diese öffentliche IP erreichbar sein.

1. Um eine statische ip-addresse zu bekommen, da dhcp sonst diese überschreibt:

<h5 a><strong><code>/etc/dhcpcd.conf</code></strong></h5>

```
interface eth0
static ip_address=10.0.0.6/24
```

2. Dann kann mit ping ein anderer Raspberry gepingt werden und man sieht, dass das Interface eth0 aktiv ist und man sich im RPi-Netz befindet.

## Aufgabenstellung Privates Netz
Der RPi erhält dann per WLAN-AccessPoint (oder USB/Ethernet Adapter) ein zweites, "privates" Interface, auf dem ein Netz aus dem Bereich 172.16.0.0/12 oder 192.168.0.0/16 konfiguriert wird.
Der RPi selbst und ein angeschlossener Laptop erhalten jeweils eine IP in diesem Netz. Dieses private Netz soll nicht von außen erreichbar sein.

1. install hostapd `sudo apt install hostapd`
2. set up the interface 

<h5 a><strong><code>/etc/network/interfaces</code></strong></h5>

```
auto wlan0
iface wlan0 inet static
    address 192.168.1.1
    netmask 255.255.255.0
```

3. set up the hostapd

<h5 a><strong><code>/etc/hostapd/hostapd.conf</code></strong></h5>

```py
interface=wlan0
driver=nl80211 
ssid=my_network
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=my_password
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP   
```

brcmfmac