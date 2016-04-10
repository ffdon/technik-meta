.. _interfaces_ffdon_gate02:

Netzwerk Interfaces
===================

/etc/network/interfaces

Nochmal langsam zum gegenprüfen::

    ### gate02.freifunk-donau-ries.de /etc/network/interfaces
            
    # Loopback device:
    auto lo
    iface lo inet loopback
    iface lo inet6 loopback

    # device: eth0
    # hier verbleiben weiter die von Hetzner vergebenen IPs
    # Die Adressen dienen nur der Wartung
    # über diese Adressen läuft kein Freifunk-Datenverkehr
    # und vor allem keine DNS-Anfragen aus dem Freifunk-Netz
    # auch nicht über einen Cache!
    auto  eth0
    iface eth0 inet dhcp
    iface eth0 inet6 static
      address 2a01:4f8:c17:393b::2
      netmask 64
      gateway fe80::1
    
    # bridge: donBR - hierübver läuft der gesamte Freifunk-Datenverkehr der community ffdon
    # deshalb werden hier auch die IPs des AS64874 aus http://wiki.freifunk.net/IP-Netze
    # bzw. unserem Netzplan
    # https://github.com/ffdon/technik-meta/blob/master/ffmwu_gateway_doku/intro/netzplan.rst
    # konfiguriert
    # ach ja die Hardware-Adresse wird errechnet:
    # https://github.com/freifunk-mwu/technik-meta/blob/master/ffmwu_gateway_doku/intro/netzplan.rst
    # https://github.com/ffdon/technik-meta/blob/master/ffmwu_gateway_doku/intro/netzplan.rst
    auto donBR
    iface donBR inet static
        hwaddress 02:00:0a:0b:90:02
        address 10.11.144.2
        netmask 255.255.240.0
        pre-up          /sbin/brctl addbr $IFACE
        up              /sbin/ip address add fdc4:d762:2143::0a0b:9002/64 dev $IFACE
        post-down       /sbin/brctl delbr $IFACE
        
    # FASTd: donVPN für unsere Nodes mit mesh-over-VPN
    allow-hotplug donVPN
    iface donVPN inet6 manual
        hwaddress 02:00:0a:0b:90:02
        pre-up          /sbin/modprobe batman-adv
        post-up         /usr/sbin/batctl -m donBAT if add $IFACE
        post-up         /sbin/ip link set dev donBAT up

    # B.A.T.M.A.N. unser Routing-Daemon 
    allow-hotplug donBAT
    iface donBAT inet6 manual
        pre-up          /sbin/modprobe batman-adv
        post-up         /sbin/brctl addif donBR $IFACE
        post-up         /usr/sbin/batctl -m $IFACE it 10000
        post-up         /usr/sbin/batctl -m $IFACE vm server
        post-up         /usr/sbin/batctl -m $IFACE gw server  96mbit/96mbit
        pre-down        /sbin/brctl delif donBR $IFACE || true

    # GRE-Tunnel zu bb-a.ak.ber - einer der redundanten Tunnel zur Ableitung des Freifunk-Verkehrs zu Freifunk Rheinland
    auto ffrl-a-ak-ber
    iface ffrl-a-ak-ber inet static
        address 100.64.1.197
        netmask 255.255.255.254
    # das ist das Tunnel-Netz, vergeben von FFRL
        pre-up          /sbin/ip tunnel add $IFACE mode gre local 10.11.144.2 remote 185.66.195.0 ttl 255
    # das ist unsere Bridge-IP (local) gefolgt von der FFRL-Backbone-IP (remote)
        post-up         /sbin/ip link set $IFACE mtu 1400
        post-up         /sbin/ip addr add 185.66.193.91/32 dev $IFACE
    # das ist unsere von FFRL zugewiesene öffentliche IPv4 des Gate02.freifunk-donau-ries.de
        post-down       /sbin/ip tunnel del $IFACE
    iface ffrl-a-ak-ber inet6 static
        address 2a03:2260:0:a1::2
        netmask 64
    # das ist unsere von FFRL zugewiesene öffentliche IPv6 des Gate02.freifunk-donau-ries.de

    # GRE-Tunnel zu bb-b.ak.ber - einer der redundanten Tunnel zur Ableitung des Freifunk-Verkehrs zu Freifunk Rheinland
    auto ffrl-b-ak-ber
    iface ffrl-b-ak-ber inet static
        address 100.64.4.201
        netmask 255.255.255.254
        pre-up          /sbin/ip tunnel add $IFACE mode gre local 10.11.144.2 remote 185.66.195.1 ttl 255
        post-up         /sbin/ip link set $IFACE mtu 1400
        post-up         /sbin/ip addr add 185.66.193.91/32 dev $IFACE
        post-down       /sbin/ip tunnel del $IFACE
    iface ffrl-b-ak-ber inet6 static
        address 2a03:2260:0:a3::2
        netmask 64

    # GRE-Tunnel zu bb-a.ix.dus - einer der redundanten Tunnel zur Ableitung des Freifunk-Verkehrs zu Freifunk Rheinland
    auto ffrl-a-ix-dus
    iface ffrl-a-ix-dus inet static
        address 100.64.4.199
        netmask 255.255.255.254
        pre-up          /sbin/ip tunnel add $IFACE mode gre local 10.11.144.2 remote 185.66.193.0 ttl 255
        post-up         /sbin/ip link set $IFACE mtu 1400
        post-up         /sbin/ip addr add 185.66.193.91/32 dev $IFACE
        post-down       /sbin/ip tunnel del $IFACE
    iface ffrl-a-ix-dus inet6 static
        address 2a03:2260:0:a2::2
        netmask 64

    # GRE-Tunnel zu bb-b.ix.dus - einer der redundanten Tunnel zur Ableitung des Freifunk-Verkehrs zu Freifunk Rheinland
    auto ffrl-b-ix-dus
    iface ffrl-b-ix-dus inet static
        address 100.64.4.202
        netmask 255.255.255.254
        pre-up          /sbin/ip tunnel add $IFACE mode gre local 10.11.144.2 remote 185.66.193.1 ttl 255
        post-up         /sbin/ip link set $IFACE mtu 1400
        post-up         /sbin/ip addr add 185.66.193.91/32 dev $IFACE
        post-down       /sbin/ip tunnel del $IFACE
    iface ffrl-b-ix-dus inet6 static
        address 2a03:2260:0:a4::2
        netmask 64
