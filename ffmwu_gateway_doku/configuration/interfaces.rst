.. _interfaces:

Netzwerk Interfaces
===================

/etc/network/interfaces

Wir richten uns hier je eine bridge pro Mesh-Wolke, die wir versorgen wollen, ein.

Auf diese bridges binden sich die Dienste (wie DHCP, DNS, NTP, etc..).
Das hoch/runterfahren der VPNs oder das (neu-)starten von Diensten kann dadurch unabhängig voneinander geschehen.

.. seealso::
    - :ref:`fastd`
    - :ref:`dhcp`
    - :ref:`routing_tables`
    - :ref:`policyrouting`
    - :ref:`internetexit`

Hier eine bridge mit IPv4 und IPv6 am Beispiel von Donau-Ries::

    auto donBR
    iface donBR inet static
        hwaddress 02:00:0a:0b:90:02
        address 10.11.144.2
        netmask 255.255.240.0
        pre-up          /sbin/brctl addbr $IFACE
        up              /sbin/ip address add fdc4:d762:2143::0a0b:9002/64 dev $IFACE
        post-down       /sbin/brctl delbr $IFACE

Hier eine bridge mit IPv4 und IPv6 am Beispiel von Wiesbaden::

    auto wiBR
    iface wiBR inet static
        hwaddress 02:42:0a:38:00:XX
        address 10.56.0.X
        netmask 255.255.192.0
        pre-up          /sbin/brctl addbr $IFACE
        up              /sbin/ip address add fd56:b4dc:4b1e::a38:X/64 dev $IFACE
        post-down       /sbin/brctl delbr $IFACE

.. seealso::
    - :ref:`routing_tables`
    - :ref:`policyrouting`
    - :ref:`gateway_schema`
    - :ref:`interface_bezeichnung`

Wir haben uns dazu entschieden jegliche Up- & Down Scripte in der /etc/network/interfaces zu verwalten.
Dies gestaltet alles übersichtlicher.

Scripte für :ref:`fastd`::

    allow-hotplug donVPN
    iface donVPN inet6 manual
        hwaddress 02:00:0a:0b:90:02
        pre-up          /sbin/modprobe batman-adv
        post-up         /usr/sbin/batctl -m donBAT if add $IFACE
        post-up         /sbin/ip link set dev donBAT up

Zum Schluss noch für das B.A.T.M.A.N. Interface::

    allow-hotplug donBAT
    iface donBAT inet6 manual
        pre-up          /sbin/modprobe batman-adv
        post-up         /sbin/brctl addif donBR $IFACE
        post-up         /usr/sbin/batctl -m $IFACE it 10000
        post-up         /usr/sbin/batctl -m $IFACE vm server
        post-up         /usr/sbin/batctl -m $IFACE gw server  96mbit/96mbit
        pre-down        /sbin/brctl delif donBR $IFACE || true

