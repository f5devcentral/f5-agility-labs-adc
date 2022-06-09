Intro to: BIG-IP HA - Do it the Proper Way
==========================================

Time and time again, we have seen customers setting up a basic HA setup.
Customer expectations were, that in case of a failover, the event would be seamless. 
They were surprised that the failover had production traffic impacts, and were cautious what to expect in the future.

In this lab, we will create a BIG-IP Active/Standby pair with best practices.
The result is a failover configuration that allows minimal impact in case of a failover event.

In order to achieve this, we will address the following BIG-IP topics:

1. Architecture
2. Network setup
3. Layer 2 Link Failure Detection
4. HA Basics
5. MAC Masquerade
6. High-Avalability (HA) Groups 
7. Gateway Pool
8. Connection Mirroring 
9. Persistence Mirroring

Architecture
------------

Our lab Architecture looks like this:

.. image:: ./images/image90.png

Network Setup
-------------

BIG-IP TMOS (Traffic Managment Operating System) classical network configuration steps are:

1. Create VLANS, and assigns VLANs to physical interfaces. 
2. Create Self IPs, and assign IP interfaces to VLANS.

Each Self IP interface has a configuration object called Port lockdown. Port lockdown determines which BIG-IP System service (like Web UI, API, SSH Access, etc.) the BIG-IP will allow on that IP interface. 

For a best practice HA setup, the BIG-IPs will have three type of IP interfaces: 

* Data interfaces - multiple possible
* HA interface
* Managment interface

Data interfaces
+++++++++++++++

The data interfaces are used to handle the data traffic (Virtual servers). The BIG-IP should not be manageable from these interfaces. The only BIG-IP system service allowed will be the heartbeat for failover, which runs on UDP port 1026.
Therefore the Portlockdown setting for the data interfaces is "custom" and UDP Port 1026.

HA interface
++++++++++++

The HA IP interface will be used for HA information, like connection mirroring, HA status updates, config sync and others.

For a secure HA setup, it is recommended that the ConfigSync & Mirroring information is **NOT** sent over a data interface/VLAN.
Therefore, customer(s) should always use a dedicated HA VLAN for ConfigSync & Mirroring configuration.

The HA VLAN self-IP should have port lockdown set to default. The self-IP should be in a IPv4 /29 network CIDR range.

It is recommended that the HA VLAN runs on a dedicated physical interface. The link speed of the HA interface should match the data interface link speed

If possible connect the HA interfaces of both BIG-IP devices direct. If a direct link is not possible, connect them to switches, but use dedicated VLANs.

Management interface
++++++++++++++++++++

The management interface is used to manage the BIG-IP. 
It should be in a dedicated subnet and not be exposed to the public or unauthorized users.


Layer 2 Link Failure Detection
------------------------------

BIG-IP TMOS has no exposure to the physical link state. It does not know if a link is up or down. 

In the past this limitation was handled by a feature called "VLAN Failsafe". 
VLAN Failsafe monitored the traffic on a specific VLAN and acted if no traffic was received. This method took between 10 and 40 seconds to detect a physical link failure.

A better way to failover on Layer 2 link failure is the use of Trunks and High availability (HA) groups.

Why Trunks?
+++++++++++

A trunk can be used for Link Aggregation or channeling of multiple physical interfaces in one bigger pipe.
At the same time a trunk can have only a single interface applied. 

BIG-IP TMOS can see the number of interfaces in a trunk. So we will use this ability to track the link status if a physical interface is up or down. 

Based on this, the best practice network configuration steps of a BIG-IP are:

1. Create Trunks and assign physical interfaces to Trunks 
2. Create VLANS and assigns VLANs to trunks. 
3. Create SelfIP and assigns IP interfaces to VLANS. 

**Every physical interface will be part of a trunk.** 

If your architecture uses one physical interface for external and one physical interface for internal traffic, then create two Trunks with one physical interface in each Trunk.

So the trunk object allows BIG-IP TMOS to failover if the one interface in the trunk goes down.

.. note:: **Remember**: Always create a Trunk first. Do not assign VLANs to physical interfaces, assign them to Trunks.

Physical interfaces that use a multiple VLANs 
+++++++++++++++++++++++++++++++++++++++++++++

If a physical interface uses 802.1Q VLAN tagging, then it is enough to monitor a single VLAN from that trunk for interface Failover.  


HA Basics
---------

With the basic HA setup we will create the BIG-IP HA Cluster. Once the HA cluster is created we will add additional configuration in this lab.

The steps for a basic HA setup are:

1. Create device trust
2. Configure config sync
3. Configure failover network
4. Configure mirroring
5. Configure device group
6. Configure MAC masquerade - where applicable


Device Trust
++++++++++++

Create the device trust.

Config Sync
+++++++++++

Set the IP address on the HA VLAN to be config sync address. 

Failover network
++++++++++++++++

We will use Failover **Unicast** Configuration only. Do not use Failover Multicast configuration.

Add all data interfaces and the management interface. **Do not** add HA interface IP address. 

Mirroring
+++++++++

Set the HA VLAN IP address as "Primary Local Mirror Address". Mirroring traffic volume can be significant. 
Use dedicated interfaces with link speed that is equal or higher to the data interfaces.

Device Group
++++++++++++

Create a device group with a name of your choice. Include both BIG-IPs and set the Sync type to "Manual with Full Sync"

The setting "Manual with full sync" is best practice for the first sync. It can be changed later.

MAC masquerade
--------------

BIG-IP's default failover mechanism is based on gratuitous ARP.
BIG-IP has to send a gratuitous ARP for every floating IP and service IP address like virtual server IP address and SNAT address.
The gratuitous ARP contains the physical MAC address of the new primary BIG-IP.
With gratuitous ARP, the device that takes over sends gratuitous ARP packets, which asks all hosts on the LAN segment to update their ARP table. 
After the hosts updated their ARP table with the MAC address of the new primary BIG-IP, they send all traffic to the active BIG-IP.

Sometimes hosts like Firewalls or routers do not update their ARP table when they receive a gratuitous ARP.
In this case the firewall or router will keep sending traffic to the old MAC address, which leads to service intererruption.

This issue can be addressed with MAC masquerade.

With MAC masquerade configured, BIG-IP devices will use the MAC masquerade address as source MAC for packets leaving BIG-IP.
In case of a failover, the MAC address will not change.
The new primary BIG-IP will start using the MAC masquerade MAC address.
Now there is no need to update the hosts ARP table. 

The MAC address used for MAC masquerade is free configurable. 
A best practices guide how to choose the MAC masquerade MAC address is described in K-Article K3523. https://support.f5.com/csp/article/K3523

For more information on MAC masquerade see K-Article K13502
https://support.f5.com/csp/article/K13502


Architectures that are recomended for MAC masquerade:
+++++++++++++++++++++++++++++++++++++++++++++++++++++

Traditional Datacenter deployments with BIG-IP applicances, Viprion or Velos Chassis

Architectures that are **NOT** recommended for MAC masquerade:
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

BIG-IP Virtual Edition deployments on VMWare ESXi.

Per default ESXi does not support multiple MAC addresses on a single NIC. 
To enable MAC masquerade the ESXi port group has to have promiscuous mode enabled. 
Check VMWare documentation on promiscuous mode.

Architectures that do not support MAC masquerade:
+++++++++++++++++++++++++++++++++++++++++++++++++

BIG-IP VE deployments in public cloud providers like AWS, Azure and GCP

High-Availability (HA) Groups
-----------------------------

An HA group is a high availability feature that allows you to specify a set of configuration objects such as trunks, pools, and VIPRION clusters that may be used to raise failover for redundant BIG-IP systems. 
When you associate an HA group with a traffic group instance on a specific device, the BIG-IP system calculates a health score for that device for the associated traffic group, based on trunk, pool, or cluster availability. 
This HA health score determines the device that should be active at any given time and then triggers failover if necessary. 

Best practices on HA groups:
https://support.f5.com/csp/article/K16947

GW Pool
-------

We will use a pool object to facilitate failover for a HA Group. 
The pool member for this pool is the default gateway of the BIG-IP.

We will use a custom gateway_icmp monitor with a short timeout (e.g. 4 sec) and a short interval (1 second).

These settigns will allow BIG-IP to failover within 4 seconds if it cannot reach its default gateway.

Alernative to the default gateway, there can be other IP endpoints within the network that can be monitored within this pool.
As long as there are pool members available, BIG-IP will assume the network layer is reachable and not use this as a failover trigger.

Connection Mirroring
--------------------

BIG-IP TMOS is a statefull proxy. 
It performs traffic forwarding and most of the time changes source/destination packet information like IP addresses and/or ports.
BIG-IP keeps track of these changes in it's connection table. 

If the connection information is not mirrored to the standby device, then all existing traffic flow during the failover will be interrupted.
Clients will have to establish a new connection, which is perceived as service disruption. 

The importance of connection mirroring is depending on the protocol. 
HTTP based protocols can carry their persistent information at higher layers, so that the individual connection might not be problematic if it has to be rebuild.
Other protocols like FTP are more sensitive, because a large file download might have to restart after a failover.

Per default the connection information is not mirrored to the standby BIG-IP. This has to be done for each Virtual server. 
This allows customers to decide which Virtual IP to mirror and which not.

Performance Impact of Connection Mirroring
++++++++++++++++++++++++++++++++++++++++++

Enabling connection mirroring can have a performance impact in terms of higher CPU load.
The performance impact depends on the type of application traffic and on the Virtual server type.

Performance Layer 4 virtual servers copy the connection setup and connection close packets to the standby device.

Standard virtual server copy the complete data stream to the standby device.

Recommended interface speed for connection mirroring
++++++++++++++++++++++++++++++++++++++++++++++++++++

Since the connection mirroring traffic can be significant, it is recommended that the interface speed for the VLAN used for connection mirroring is the same speed as the external interface.

e.g. is the external interface speed is 10 Gbit/s then the connection mirroring interface should be 10 Gbit/s as well. 

More information on connection mirroring
https://support.f5.com/csp/article/K84303332


Persistance Mirroring
---------------------

Applications can require that once a client is load balanced to a specific pool member, subsequent requests will be sent to the same pool member.
It is important that the persistence information is copied to the standby device, to prevent the same client to be sent to another pool member after the failover.
Persistence mirroring is used that.

Persistence information is only sent when a new persistence record is created or an old record expired. It has less performance impact on BIG-IP than connection mirroring.

.. toctree::
   :maxdepth: 1
   :glob:

   labinfo
   Module*/Lab*