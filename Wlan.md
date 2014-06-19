## Oase Mesh

Doku aus dem letzem Jahr:

Also Prinzipiell schauts jetzt so aus:

Clients: `10.10.0.0 - 10.10.253.254 (~/16 [255.255.0.0])`
Management Netz: `10.10.254.0/24`
Firewall:  `10.11.0.254`
           `rückroute auf 10.10.0.0/16 [10.11.0.1 (Border-Mesh-GW)]`
Transportnetz:  '10.11.0.0./24'
* [GW](#GW)  `10.11.0.1` und `10.10.254.254`
* [N1](#N1)  `10.10.254.1`
* [AP1](#AP1)  `10.10.254.101`


'

Der Border-Mesh-Router ist das Batman-Adv Gateway.. dieser hat die
10.10.254.254 und routet gegen die Firewall auf einem Transportnetz (10.11.0.0/24)

hat also die 10.11.0.1 und schickt den Verkehr an 10.11.0.254 (Die
Firewall)

DHCP-Requests aus dem Wifi schlagen auf dem Border-Mesh-GW auf und
werden an die Firewall mittels dhcp-fwd geforwarded.. (Config anbei)

Ich hab die Firewall hier im Testsetup mit einem OpenWrt-Router
gestellt... Deswegen hier die Beispiel-Config für den DNSMASQ wenn
DHCP-Requests von einem DHCP Relay-Agent kommen... Wie das auf der pf
geht muss stefan rausfinden... Sollte ja aber kein Act sein. 
Die Firewall braucht unterdessen eine Rückroute nach 10.10.0.0/16 auf die
10.11.0.1 (Border-Mesh-GW) - klar

Nochmal zur anmerkung.. Es ist 8.8.8.8 auf den Kisten als DNS-Server
eingetragen....

UND: Nicht verwechseln:

Bei den WR841N ist das LAN eth1 und WAN ist eth0
Bei den WR741ND ist LAN eth0 und WAN eth1

:P

Clients sollten später einen internen DNS-Server gedrückt bekommen...
Dazu Config vom wirklichen DHCP-Server anpassen



Wenn ihr die Kisten aufstellt bitte nochmal den Kanal provisionieren..
Im moment ist da irgendwas eingestellt...

Also:

uci set wireless.@wifi-device[0].channel=[5,8,11]
uci commit wireless


##################################

DHCP-Relay auf Firewall mit DNSMASQ

Hier wird die Range gelegt.. mit set:hot wird hier noch ein schlüssel
definiert mit dem die dhcp-options gesetzt werden können...

Das einfach in die /etc/dnsmasq.conf reinschieben

##################################





dhcp-range=set:hot,10.10.0.0,10.10.253.254,255.255.0.0,120h

server=/hot/10.10.254.254
dhcp-option=tag:hot,option:router,10.10.254.254
dhcp-option=tag:hot,option:dns-server,8.8.8.8



- --------------------------------------------------------


##################################
<a name="GW"/>
### GW - Border-Mesh-GW

Nun die Config vom Border-Mesh-GW.. Ich hatte ja schon angemerkt das
hier ein generisches 12.09 aus dem OpenWrt-Download-Mirror benutzt
wird..

Also die Kiste flashen, ans Netz hängen und folgenden Init fahren:

* Batman-Adv installieren
* Network

	* Mesh-Interface anlegen (Wifi)
		- MTU 1528 !!!
	-> Batman-Adv hat einen größeren Header..

	* LAN-Interface konfigurieren
		- IP setzen
		- Subnet setzen
		- bat0 auf lan bridgen

	* WAN-Interface für Uplink
		- IP setzen
		- Subnet setzen
		- Broadcast setzen
		- Gateway setzen
		- DNS setzen

* Wireless

	* Mesh-Wifi-SSID -> mesh
	* Mesh-Wifi-BSSID -> 02:BA:FF:EE:BA:BE
	* Wifi-HWMode -> 11ng
(http://wiki.openwrt.org/doc/uci/wireless#common.options)
	* Wifi-Channel -> 1
	* Wifi-Mode -> adhoc
	* Network -> mesh


* DHCP
	* Disable


* System
	* Hostname

* Batman-Adv
	* Interfaces -> mesh
	* aggregated_ogms -> 1
	* ap_isolation -> 0
	* bonding -> 0
	* fragmentation -> 1
	* gw_mode -> server			<-------
	* log_level -> 0
	* orig_interval -> 1000
	* vis_mode -> server			<-------
	* bridge_loop_avoidance -> 1


Danach Reboot...

Disable NAT
- -------------



Weil ich die uci-Regeln für die Firewall so kompliziert finde hier eine
ganz einfach Lösung das NAT auszuschalten:

Das WAN-Interface in die LAN-Zone werfen und Forward ACCEPTEN.. Fertig


DHCP-Forwarder
- --------------

Fr den DHCP-Forwarder steht unten eine config bereit. Die muss einfach
in /etc/dhcp-fwd abgeschmissen werden..


##################################




opkg update
opkg install kmod-batman-adv dhcp-fwd

uci set network.mesh='interface'
uci set network.mesh.proto='none'
uci set network.mesh.mtu='1528'
uci set network.lan.proto='static'
uci set network.lan.ipaddr='10.10.254.254'
uci set network.lan.netmask='255.255.255.0'
uci set network.lan.ifname='bat0 eth1'
uci set network.wan.ifname='eth0'
uci set network.wan.proto='static'
uci set network.wan.ipaddr='10.11.0.254'
uci set network.wan.netmask='255.255.255.0'
uci set network.wan.gateway='10.11.0.1'
uci set network.wan.dns='8.8.8.8'
uci commit network

uci set wireless.@wifi-device[0].disabled=0
uci set wireless.@wifi-device[0].hwmode=11ng
uci set wireless.@wifi-device[0].channel=1
uci set wireless.@wifi-iface[0].mode=adhoc
uci set wireless.@wifi-iface[0].ssid=mesh
uci set wireless.@wifi-iface[0].network=mesh
uci set wireless.@wifi-iface[0].bssid=02:BA:FF:EE:BA:BE
uci commit wireless

uci set dhcp.@dhcp[0].ignore=1
uci commit dhcp

uci set system.@system[0].hostname=gw1
uci commit system

uci set batman-adv.@mesh[0].interfaces=mesh
uci set batman-adv.@mesh[0].aggregated_ogms=1
uci set batman-adv.@mesh[0].ap_isolation=0
uci set batman-adv.@mesh[0].bonding=0
uci set batman-adv.@mesh[0].fragmentation=1
uci set batman-adv.@mesh[0].gw_mode=server
uci set batman-adv.@mesh[0].log_level=0
uci set batman-adv.@mesh[0].orig_interval=1000
uci set batman-adv.@mesh[0].vis_mode=server
uci set batman-adv.@mesh[0].bridge_loop_avoidance=1
uci commit batman-adv



Jetzt NAT übers WEB-IF ausschalten (siehe oben)


DHCP-Forwarder
- --------------

/etc/dhcp-fwd.conf:


user            0
group           0

~#chroot         /var/run/dhcp-fwd

logfile         /var/dhcp-fwd.log
loglevel        0

pidfile         /var/run/dhcp-fwd.pid

if      br-lan  true    false   true
if      eth0    false   true    true

server  ip      10.11.0.1




- --------------------------------------------------------



##################################
<a name="N1"/>
### N1 - Infrastruktur-Node


Verfält sich fast genauso, hier nur die Unterschiede:


* Network

	* WAN-Interface für Batman-Adv
		- Interface eth0 reinsetzen
		- MTU 1528 !!!
		- keine IP - keine route

	* LAN-Interface
		- gateway setzen
		- dns setzen

* Wireless

	* Mesh-Wifi-SSID -> mesh
	* Mesh-Wifi-BSSID -> 02:BA:FF:EE:BA:BE
	* Wifi-HWMode -> 11ng
(http://wiki.openwrt.org/doc/uci/wireless#common.options)
	* Wifi-Channel -> 1
	* Wifi-Mode -> adhoc
	* Network -> mesh

* System
	* Hostname

* Batman-Adv
	* Interfaces -> mesh eth0
	* gw_mode -> client
	* vis_mode -> client



Danach Reboot...


##################################


opkg update
opkg install kmod-batman-adv

uci set network.mesh='interface'
uci set network.mesh.proto='none'
uci set network.mesh.mtu='1528'
uci set network.lan.proto='static'
uci set network.lan.ipaddr='10.10.254.1'
uci set network.lan.netmask='255.255.255.0'
uci set network.lan.gateway='10.10.254.254'
uci set network.lan.dns='8.8.8.8'
uci set network.lan.ifname='bat0 eth1'
uci set network.wan.proto='none'
uci set network.wan.mtu='1528'
uci set network.wan.ifname='eth0'
uci commit network

uci set wireless.@wifi-device[0].disabled=0
uci set wireless.@wifi-device[0].hwmode=11ng
uci set wireless.@wifi-device[0].channel=1
uci set wireless.@wifi-iface[0].mode=adhoc
uci set wireless.@wifi-iface[0].ssid=mesh
uci set wireless.@wifi-iface[0].network=mesh
uci set wireless.@wifi-iface[0].bssid=02:BA:FF:EE:BA:BE
uci commit wireless

uci set dhcp.@dhcp[0].ignore=1
uci commit dhcp

uci set system.@system[0].hostname=n1
uci commit system

uci set batman-adv.@mesh[0].interfaces='mesh eth0'
uci set batman-adv.@mesh[0].aggregated_ogms=1
uci set batman-adv.@mesh[0].ap_isolation=0
uci set batman-adv.@mesh[0].bonding=0
uci set batman-adv.@mesh[0].fragmentation=1
uci set batman-adv.@mesh[0].gw_mode=client
uci set batman-adv.@mesh[0].log_level=0
uci set batman-adv.@mesh[0].orig_interval=1000
uci set batman-adv.@mesh[0].vis_mode=client
uci set batman-adv.@mesh[0].bridge_loop_avoidance=1
uci commit batman-adv




##################################
<a name="AP1"/>
### AP1 - Access-Node


Verfält sich fast genauso, hier nur die Unterschiede:


* Network

	* WAN-Interface für Batman-Adv
		- Interface eth1 reinsetzen !!!!!
		- MTU 1528 !!!
		- keine IP - keine route

	* LAN-Interface
		- Interface ist hier eth0 !!!!
		- gateway setzen
		- dns setzen

* Wireless

	* Wifi-SSID -> Oase
	* Wifi-Channel -> 5,8,11
	* Wifi-Mode -> AP
	* Network -> lan

* System
	* Hostname

* Batman-Adv
	* Interfaces -> mesh eth1
	* gw_mode -> client
	* vis_mode -> client



Danach Reboot...


##################################



opkg update
opkg install kmod-batman-adv

uci set network.mesh=interface
uci set network.mesh.proto=none
uci set network.mesh.mtu=1528
uci set network.mesh.ifname=eth1
uci set network.wan.proto=none
uci set network.wan.mtu=1528
uci set network.wan.ifname='eth0'
uci set network.lan.proto=static
uci set network.lan.ipaddr='10.10.254.101'
uci set network.lan.netmask='255.255.255.0'
uci set network.lan.gateway='10.10.254.254'
uci set network.lan.dns='8.8.8.8'
uci set network.lan.ifname='bat0 eth0'
uci commit network

uci set wireless.@wifi-device[0].disabled=0
uci set wireless.@wifi-device[0].hwmode=11ng
uci set wireless.@wifi-device[0].channel=2
uci set wireless.@wifi-iface[0].mode=ap
uci set wireless.@wifi-iface[0].network=lan
uci set wireless.@wifi-iface[0].ssid=oase
uci commit wireless

uci set dhcp.@dhcp[0].ignore=1
uci commit dhcp

uci set system.@system[0].hostname=ap1
uci commit system

uci set batman-adv.@mesh[0].interfaces=eth1
uci set batman-adv.@mesh[0].aggregated_ogms=1
uci set batman-adv.@mesh[0].ap_isolation=0
uci set batman-adv.@mesh[0].bonding=0
uci set batman-adv.@mesh[0].fragmentation=1
uci set batman-adv.@mesh[0].gw_mode=client
uci set batman-adv.@mesh[0].log_level=0
uci set batman-adv.@mesh[0].orig_interval=1000
uci set batman-adv.@mesh[0].vis_mode=client
uci set batman-adv.@mesh[0].bridge_loop_avoidance=1
uci commit batman-adv

- ------------------------------------------------




Automatisierung der Visualisierung und das Frontent als tar.gz kommt
noch kommt noch...



Ich mach jetzt noch die bat-hosts für die Kisten damit das nicht so
aussieht:

root@ap1:~# batctl o
[B.A.T.M.A.N. adv 2012.3.0, MainIF/MAC: eth1/f8:d1:11:55:93:8b (bat0)]

Originator      last-seen (#/255)           Nexthop [outgoingIF]:
Potential nexthops ...
a0:f3:c1:64:56:6f    0.150s   (255) a0:f3:c1:64:56:6f [      eth1]:
a0:f3:c1:64:56:6f (255)
a0:f3:c1:64:57:17    0.810s   (225) a0:f3:c1:64:56:6f [      eth1]:
a0:f3:c1:64:56:6f (225)
a0:f3:c1:64:57:5b    0.210s   (225) a0:f3:c1:64:56:6f [      eth1]:
a0:f3:c1:64:56:6f (225)

f8:d1:11:23:3f:db    0.950s   (188) a0:f3:c1:64:56:6f [      eth1]:
a0:f3:c1:64:56:6f (188)
a0:f3:c1:64:57:50    0.280s   (220) a0:f3:c1:64:56:6f [      eth1]:
a0:f3:c1:64:56:6f (220)
a0:f3:c1:64:55:bc    1.280s   (217) a0:f3:c1:64:56:6f [      eth1]:
a0:f3:c1:64:56:6f (217)
f8:d1:11:55:99:89    0.530s   (196) a0:f3:c1:64:56:6f [      eth1]:
a0:f3:c1:64:56:6f (196)
a0:f3:c1:64:57:0b    0.210s   (221) a0:f3:c1:64:56:6f [      eth1]:
a0:f3:c1:64:56:6f (221)
a0:f3:c1:64:55:bb    0.210s   (221) a0:f3:c1:64:56:6f [      eth1]:
a0:f3:c1:64:56:6f (221)
f8:d1:11:55:99:3f    0.530s   (198) a0:f3:c1:64:56:6f [      eth1]:
a0:f3:c1:64:56:6f (198)
f8:d1:11:55:94:73    0.530s   (198) a0:f3:c1:64:56:6f [      eth1]:
a0:f3:c1:64:56:6f (198)
a0:f3:c1:64:57:18   46.740s   (223) a0:f3:c1:64:56:6f [      eth1]:
a0:f3:c1:64:56:6f (223)

root@ap1:~#

