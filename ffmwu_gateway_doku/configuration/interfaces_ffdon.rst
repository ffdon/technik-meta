.. _interfaces_ffdon_gate02:

Netzwerk Interfaces
===================

/etc/network/interfaces

Nochmal langsam zum gegenprüfen.

Auf diese bridges binden sich die Dienste (wie DHCP, DNS, NTP, etc..).
Das hoch/runterfahren der VPNs oder das (neu-)starten von Diensten kann dadurch unabhängig voneinander geschehen.

.. seealso::
    - :ref:`fastd`
    - :ref:`dhcp`
    - :ref:`routing_tables`
    - :ref:`policyrouting`
    - :ref:`internetexit`

Hier eine bridge mit IPv4 und IPv6 am Beispiel von Donau-Ries::
