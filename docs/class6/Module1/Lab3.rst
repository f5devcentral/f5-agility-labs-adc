Lab 3:  Configure MAC Masquerade
--------------------------------

In Lab 3, we will setup MAC masquerading at the traffic-group level, allowing a "floating MAC" to be shared across the traffic-group, and validate BIG-IP configuration synchronization.  


Lab Tasks:
==========

* Task 1: Configure MAC Masquerade
* Task 2: Verify an Active / Standby "In Sync" State

Task 1:  Configure MAC Masquerade
=====================================

In Task 1, we will configure & setup MAC Masquerade.

To optimize the flow of traffic during failover events, you can configure a MAC masquerade address for any defined traffic-group on the BIG-IP system. A MAC masquerade address is a unique, floating MAC address that you create. You can assign one MAC masquerade address to each traffic-group on a BIG-IP device. 

.. note:: MAC Masauerade is NOT a requirement, or a fit for every architecture; it enhances our BIG-IP behavior for architectures that have issues with failover & receiving ARP updates (i.e. timing, flooding, policing, etc.). It can enhance failover timing under certain circumstances. This lab showcases where / why you would configure it.

BIG-IP's default failover mechanism is based on Gratuitous ARP (GARP).
In case of a failover event, the BIG-IP has to send a gratuitous ARP for every floating IP and service IP address (i.e. Virtual Server IP address [VIP]; SNAT address; etc.)
The GARP contains the physical MAC address of the new primary BIG-IP.
With gratuitous ARP, the device that takes over sends GARP packets, which asks all hosts on the LAN segment to update their ARP table. 
After the hosts update their ARP table with the MAC address of the new primary BIG-IP, they send all traffic to the now active BIG-IP.

Sometimes, hosts like Firewalls or routers, do not update their ARP table when they receive a gratuitous ARP.
In this case, the firewall or router will keep sending traffic to the old MAC address, which leads to service intererruption.

This issue can be addressed with MAC masquerade.

With MAC masquerade configured, BIG-IP devices will use a configurable MAC masquerade address as source MAC for packets leaving the BIG-IP.
In case of a failover, the MAC address will not change.
The new active BIG-IP will start using the MAC masquerade MAC address.
Now, there is no need to update the hosts ARP table. 

The MAC address used for MAC masquerade is freely configurable. 
A best practices guide on how to choose a unique MAC masquerade MAC address is described in Knowledge Article `K3523 <https://support.f5.com/csp/article/K3523>`_

For more information on MAC masquerade see Knowledge Article `K13502 <https://support.f5.com/csp/article/K13502>`_

In Virtualized environments, there are some configuration caveats to be aware of; please review the **Notes** section in Article `K13502: Configuring MAC masquerade (11.x - 16.x) <https://support.f5.com/csp/article/K13502>`_

First, we need to obtain a Unique MAC address to use for our MAC Masquerade.  We will leverage one of our Virtual Interfaces MACs; we'll flip the 1st MAC HEX value to "02."

.. note:: For additional details on creating a unique L2 MAC Address, please see Article `K3523: Choosing a unique MAC address for MAC masquerade <https://support.f5.com/csp/article/K3523>`_

1.  On the **ACTIVE** BIG-IP, **Navigate to**: Network > Interfaces, and copy the 1.1 MAC address to your "copy/paste" machine buffer:
   
    .. image:: ../images/image116.png

2.  Now, **Navigate to**: Device Management > Traffic Groups > click the traffic-group-1 hyperlink:
   
    .. image:: ../images/image117.png

3.  In the MAC Masquerade Address Field, paste the previously saved MAC Address:
   
    .. image:: ../images/image118.png

    Replace the "52" with "02" and click the **Save** button:

    .. image:: ../images/image119.png


Task 2: Verify an Active / Standby "In Sync" State
==================================================

In Task 2, we will perform a BIG-IP device synchronization.  This will validate our HA DSC configuration, along with sharing this unique MAC masquerade configuration.

#. On each BIG-IP, review the current state.

   -  BIG-IP-A:

      .. image:: ../images/image183.png

   -  BIG-IP-B:

      .. image:: ../images/image182.png


#. On the **ACTIVE** BIG-IP, click the **Changes Pending** hyperlink, and proceed with the recommended synchronization action:

.. image:: ../images/image184.png

#. Validate your BIG-IPs are In Sync:

   -  BIG-IP-A:

      .. image:: ../images/image186.png

   -  BIG-IP-B:

      .. image:: ../images/image185.png

Lab Summary
===========

In this lab, you configured MAC Masquerade.  This feature can enhance failover timing with upstream devcies.
After completion of these lab tasks, you should have a basic Active/Standby High-Availability (HA) BIG-IP configuration, with Local Traffic Manager (LTM) objects (Floating Self IPs), that are synchronized between the BIG-IPs.  These configuration objects will assist with the following labs.

This completes Lab 3.