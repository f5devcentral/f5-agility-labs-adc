Lab 1: West Data Center Configuration
=====================================

In this lab you will be configuring a BIG-IP HA cluster and setting it to advertise available virtual servers through the BGP protocol.

Configure the BIG-IP HA Cluster in the West Data Center
-------------------------------------------------------

From the jumphost ssh to the W_A_BIGIP-13

.. code-block:: none

        root@jumphost:~# ssh -l root 192.168.1.3

.. code-block:: none

	tmsh modify sys ntp servers add { pool.ntp.org }
	tmsh mv cm device bigip1 W_A_BIGIP-13.local
	tmsh modify cm device W_A_BIGIP-13.local configsync-ip 192.168.255.10
	tmsh modify cm device W_A_BIGIP-13.local  unicast-address { { ip 192.168.255.10 } }
	
From the jumphost ssh to the W_B_BIGIP-13

.. code-block:: none

	root@jumphost:~# ssh -l root 192.168.1.6

.. code-block:: none

	tmsh modify sys ntp servers add { pool.ntp.org }
	tmsh mv cm device bigip1 W_B_BIGIP-13.local
	tmsh modify cm device W_B_BIGIP-13.local configsync-ip 192.168.255.11
	tmsh modify cm device W_B_BIGIP-13.local  unicast-address { { ip 192.168.255.11 } }

Create the trust domain and sync-failover group from one of the two BIGIP's:
	
From W_B_BIGIP-13
	
.. code-block:: none

	tmsh modify cm trust-domain /Common/Root ca-devices add { 192.168.255.10 } name W_A_BIGIP-13.local username admin password admin

	tmsh create cm device-group sync-failover type sync-failover auto-sync enabled save-on-auto-sync true devices add { W_A_BIGIP-13.local W_B_BIGIP-13.local }
		
Run the initial configuration sync from one of the two BIGIP's:

.. code-block:: none

	tmsh run cm config-sync to-group sync-failover
	
	[root@W_B_BIGIP-13:Active:Changes Pending] config # 
	[root@W_B_BIGIP-13:Active:Awaiting Initial Sync] config # 
	[root@W_B_BIGIP-13:Active:Awaiting Initial Sync] config # 
	[root@W_B_BIGIP-13:Active:In Sync] config # 
	[root@W_B_BIGIP-13:Active:In Sync] config # 
	
Save the configuration on **both** BIG-IP devices
	
.. code-block:: none

	tmsh save sys config partitions all

Create an application configuration for a virtual server and a pool member
--------------------------------------------------------------------------

Create the following virtual server and pool member


.. code-block:: none

	tmsh create ltm pool pool1 members add { 10.3.99.200:80 } monitor tcp_half_open

	tmsh create ltm virtual vip1 destination 10.99.99.100:80 source-address-translation { type automap } pool pool1 profiles add { tcp http }

.. Note:: The configuration should now be auto-sync'd to the other device and auto-saved.

.. ATTENTION:: The virtual server is not available.

.. code-block:: none

	[root@W_A_BIGIP-13:Standby:In Sync] config # tmsh show ltm virtual
	------------------------------------------------------------------
	Ltm::Virtual Server: vip1      
	------------------------------------------------------------------
	Status                         
	  Availability     : offline   
	  State            : enabled   
	  Reason           : The children pool member(s) are down
	  CMP              : enabled   
	  CMP Mode         : all-cpus  
	  Destination      : 10.99.99.100:80

**The pool members are unavailable.**
	
.. code-block:: none

	[root@W_A_BIGIP-13:Standby:In Sync] config # tmsh show ltm pool
	---------------------------------------------------------------------
	Ltm::Pool: pool1                       
	---------------------------------------------------------------------
	Status                                 
	  Availability : offline               
	  State        : enabled               
	  Reason       : The children pool member(s) are down
	  Monitor      : tcp_half_open         
	  Minimum Active Members : 0           
	  Current Active Members : 0           
	       Available Members : 0           
  	           Total Members : 1               
	          Total Requests : 0           
	        Current Sessions : 0


You need to create a static route to the web server located in the core network on both BIG-IPs.

.. code-block:: none

	tmsh create net route webservers network 10.3.99.0/24 gw 10.1.20.254

.. Note:: This only needs to be configured on one device as seen below

.. code-block:: none

	[root@W_B_BIGIP-13:Active:In Sync] config # tmsh create net route webservers network 10.3.99.0/24 gw 10.1.20.254
	[root@W_B_BIGIP-13:Active:Not All Devices Synced] config # 
	[root@W_B_BIGIP-13:Active:In Sync] config # 

Your virtual server should now show available.

.. code-block:: none

	tmsh show ltm virtual

	------------------------------------------------------------------
	Ltm::Virtual Server: vip1      
	------------------------------------------------------------------
	Status                         
	  Availability     : available 
	  State            : enabled   
	  Reason           : The virtual server is available
	  CMP              : enabled   
	  CMP Mode         : all-cpus  
	  Destination      : 10.99.99.100:80
			
.. Note:: From the jump host you can now try to reach the website

Either open a web browser and browse to http://10.99.99.100 or from the jump host CLI, type:
    
.. code-block:: none

	curl http://10.99.99.100

Why is it not available?
                 
Configure the route advertisement on the BIG-IP cluster
-------------------------------------------------------

On both W_A_BIGIP and W_B_BIGIP configure the following:
	
.. code-block:: none

	tmsh modify net route-domain 0 routing-protocol add { BGP }

.. Note::
	The following just needs to be applied on just one device

.. code-block:: none

	tmsh modify ltm virtual-address 10.99.99.100 route-advertisement all 

.. Note::

    route-advertisement
        Specifies the route advertisement setting for the virtual address.
        The default value is disabled.

        enabled or selective
                Route is advertised when virtual-address is available.

        disabled
                Route is not advertised.

        always
                Route is advertised regardless of the availability status.
    
        any  Route is advertised when any of the contributing virtual
                server is available.

        all  Route is advertised when all of the contributing virtual
                server is available.

Because the previous commands are also base configuration you should save the configuration on both devices.

.. code-block:: none

	tmsh save sys config

Now you can begin to configure the routing protocol in ZebOS. To reach the ZebOS CLI, from the BIG-IP BASH shell.

.. code-block:: none

        [root@W_B_BIGIP-13:Active:In Sync] config # imish
        [root@W_B_BIGIP-13:Active:In Sync] config # enable
        W_B_BIGIP-13.local[0]#show ip route
        Codes: K - kernel, C - connected, S - static, R - RIP, B - BGP
                O - OSPF, IA - OSPF inter area
                N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
                E1 - OSPF external type 1, E2 - OSPF external type 2
                i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
                * - candidate default
        C       10.1.10.0/24 is directly connected, external
        C       10.1.20.0/24 is directly connected, internal
        K       10.3.99.0/24 [0/0] via 10.1.20.254, internal
        K       10.99.99.100/32 [0/0] is directly connected, tmm
        C       127.0.0.1/32 is directly connected, lo
        C       127.1.1.254/32 is directly connected, tmm
        C       192.168.255.0/24 is directly connected, ha
        
        Gateway of last resort is not set

.. code-block:: none

        [root@W_A_BIGIP-13:Active:In Sync] config # imish
        [root@W_A_BIGIP-13:Active:In Sync] config # enable
        W_A_BIGIP-13.local[0]>sh ip route
        Codes: K - kernel, C - connected, S - static, R - RIP, B - BGP
                O - OSPF, IA - OSPF inter area
                N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
                E1 - OSPF external type 1, E2 - OSPF external type 2
                i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
                * - candidate default
        C       10.1.10.0/24 is directly connected, external
        C       10.1.20.0/24 is directly connected, internal
        K       10.3.99.0/24 [0/0] via 10.1.20.254, internal
        C       127.0.0.1/32 is directly connected, lo
        C       127.1.1.254/32 is directly connected, tmm
        C       192.168.255.0/24 is directly connected, ha
        
        Gateway of last resort is not set

.. Note::

	The two kernel routes designated with a 'K'. These are routes provided to ZebOS from tmm. The first route you may recall is 10.3.99.0/24. This is the static route you configured via tmsh. The second route, 10.99.99.100/32 is the virtual server.

.. Note::

	Why does the kernel route for 10.99.99.0/32 only show up in one of the devices?

Force a failover event to occur on the 'Active' device and validate.

.. code-block:: none

        W_B_BIGIP-13.local[0]#quit
	
.. code-block:: none

        tmsh run sys failover standby

.. Note::

        This needs to be run from whichever device in your HA Pair is currently active. Once you 'quit' imish, it can be determined from the bash shell prompt.

        [root@W_A_BIGIP-13:Active:In Sync]
        [root@W_B_BIGIP-13:Active:In Sync]

.. code-block:: none

        [root@W_A_BIGIP-13:Active:In Sync] config # imish
        [root@W_B_BIGIP-13:Active:In Sync] config # enable
        W_B_BIGIP-13.local[0]#sh ip route
        Codes: K - kernel, C - connected, S - static, R - RIP, B - BGP
                O - OSPF, IA - OSPF inter area
                N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
                E1 - OSPF external type 1, E2 - OSPF external type 2
                i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
                * - candidate default
        C       10.1.10.0/24 is directly connected, external
        C       10.1.20.0/24 is directly connected, internal
        K       10.3.99.0/24 [0/0] via 10.1.20.254, internal
        C       127.0.0.1/32 is directly connected, lo
        C       127.1.1.254/32 is directly connected, tmm
        C       192.168.255.0/24 is directly connected, ha
        
        Gateway of last resort is not set

.. code-block:: none

        [root@W_A_BIGIP-13:Active:In Sync] config # imish
        [root@W_B_BIGIP-13:Active:In Sync] config # enable
        W_A_BIGIP-13.local[0]#show ip route
        Codes: K - kernel, C - connected, S - static, R - RIP, B - BGP
                O - OSPF, IA - OSPF inter area
                N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
                E1 - OSPF external type 1, E2 - OSPF external type 2
                i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
                * - candidate default
        
        C       10.1.10.0/24 is directly connected, external
        C       10.1.20.0/24 is directly connected, internal
        K       10.3.99.0/24 [0/0] via 10.1.20.254, internal
        K       10.99.99.100/32 [0/0] is directly connected, tmm
        C       127.0.0.1/32 is directly connected, lo
        C       127.1.1.254/32 is directly connected, tmm
        C       192.168.255.0/24 is directly connected, ha
        
        Gateway of last resort is not set

Configure the iBGP session with the West CPE devices W_CPE_A_CSR1k and W_CPE_A_CSR1k. The CPE configuration is already done for you so you only need to configure the two BIG-IP devices.

.. code-block:: none

        [root@W_A_BIGIP-13:Active:In Sync] config # imish
        W_A_BIGIP-13.local[0]>enable
        W_A_BIGIP-13.local[0]#config t
        W_A_BIGIP-13.local[0](config)#router bgp 65101
        W_A_BIGIP-13.local[0](config-router)# neighbor 10.1.10.3 remote-as 65101
        W_A_BIGIP-13.local[0](config-router)# neighbor 10.1.10.3 description W_CPE_A
        W_A_BIGIP-13.local[0](config-router)# neighbor 10.1.10.4 remote-as 65101
        W_A_BIGIP-13.local[0](config-router)# neighbor 10.1.10.4 description W_CPE_B
        W_A_BIGIP-13.local[0](config-router)# neighbor 10.1.10.6 remote-as 65101
        W_A_BIGIP-13.local[0](config-router)# neighbor 10.1.10.6 description W_B_BIGIP-13
        W_A_BIGIP-13.local[0](config-router)# redistribute kernel
        W_A_BIGIP-13.local[0](config)#end
        W_A_BIGIP-13.local[0]#wr
        Building configuration...
        [OK]
        W_A_BIGIP-13.local[0]#

.. code-block:: none

        [root@W_B_BIGIP-13:Active:In Sync] config # imish
        W_B_BIGIP-13.local[0]>enable
        W_B_BIGIP-13.local[0]#config t
        W_B_BIGIP-13.local[0](config)#router bgp 65101
        W_B_BIGIP-13.local[0](config-router)# neighbor 10.1.10.3 remote-as 65101
        W_B_BIGIP-13.local[0](config-router)# neighbor 10.1.10.3 description W_CPE_A
        W_B_BIGIP-13.local[0](config-router)# neighbor 10.1.10.4 remote-as 65101
        W_B_BIGIP-13.local[0](config-router)# neighbor 10.1.10.4 description W_CPE_B
        W_B_BIGIP-13.local[0](config-router)# neighbor 10.1.10.5 remote-as 65101
        W_B_BIGIP-13.local[0](config-router)# neighbor 10.1.10.5 description W_A_BIGIP-13
        W_B_BIGIP-13.local[0](config-router)#redistribute kernel
        W_B_BIGIP-13.local[0](config-router)#end
        W_B_BIGIP-13.local[0]#wr
        Building configuration...
        [OK]
        W_B_BIGIP-13.local[0]#
			
.. Note::  BGP has not come up between the BIG-IP's.  Why is that?

.. code-block:: none

        W_A_BIGIP-13.local[0]#sh ip bgp summary
        BGP router identifier 192.168.255.10, local AS number 65101
        BGP table version is 3
        4 BGP AS-PATH entries
        0 BGP community entries
        
        Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
        10.1.10.3       4 65101      41      32        3    0    0 00:00:26        6
        10.1.10.4       4 65101      40      30        3    0    0 00:00:26        6
        10.1.10.6       4 65101       7       7        0    0    0 00:01:04 Active   
			
Configure port lockdown on BIG-IP's to allow BGP via external interface on both BIG-IP:

.. code-block:: none

    tmsh list net self external allow-service 
        net self external {
            allow-service none
        }
.. code-block:: none

	tmsh modify net self external allow-service add { tcp:179 } 

.. code-block:: none

    tmsh list net self external allow-service 
        net self external {
            allow-service {
                tcp:bgp
            }
        }
					
Validate BGP is now established between both BIGIP's

.. code-block:: none

        W_B_BIGIP-13.local[0]#sh ip bgp summary
        BGP router identifier 192.168.255.11, local AS number 65101
        BGP table version is 5
        4 BGP AS-PATH entries
        0 BGP community entries
        
        Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
        10.1.10.3       4 65101      32      23        5    0    0 00:03:16        6
        10.1.10.4       4 65101      32      23        5    0    0 00:03:16        6
        10.1.10.5       4 65101      12      11        5    0    0 00:00:07        2
        
        Total number of neighbors 3

.. Note:: iBGP requires that you configure a full mesh. But in this simplified scenario you do not need to configure a full mesh for iBGP because there are only four routers and you've got the BIG-IP HA Cluster and your outbound gateway. Your topology may vary and you may want to configure a full mesh if there are other devices. Additionally in a larger topology you may want to configure eBGP between the CPE and the BIG-IP clusters. This is explored in more depth when you configure the East data center.
	
.. Note:: The redistribute kernel command is the secret sauce that allows tmm routes from the BIG-IP to be advertised to BGP peers.

Validate that the website is accessible from the jumphost

.. code-block:: none

        root@jumphost:~# curl 10.99.99.100
        <html><body><h1>It works!</h1>
        <p>This is the default web page for this server.</p>
        <p>The web server software is running but no content has been added, yet.</p>
        </body></html>

Validate your configuration on both of the CPE routers
	
You can telnet to either of the CPE devices

.. code-block:: none

        csr1000v-W_CPE_A#sh ip route 
        Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
                D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
                N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
                E1 - OSPF external type 1, E2 - OSPF external type 2
                i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
                ia - IS-IS inter area, * - candidate default, U - per-user static route
                o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
                a - application route
                + - replicated route, % - next hop override, p - overrides from PfR

        Gateway of last resort is not set

                1.0.0.0/32 is subnetted, 2 subnets
        C        1.1.1.1 is directly connected, Loopback100
        O        1.1.1.2 [110/2] via 10.1.10.4, 02:49:46, GigabitEthernet4
                3.0.0.0/32 is subnetted, 1 subnets
        B        3.3.3.3 [20/0] via 172.16.1.3, 00:05:44
                10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
        C        10.1.10.0/24 is directly connected, GigabitEthernet4
        L        10.1.10.3/32 is directly connected, GigabitEthernet4
        B        10.99.99.100/32 [200/0] via 10.1.10.5, 00:00:07
                172.16.0.0/16 is variably subnetted, 6 subnets, 2 masks
        C        172.16.1.0/24 is directly connected, GigabitEthernet2
        L        172.16.1.4/32 is directly connected, GigabitEthernet2
        B        172.16.2.0/24 [20/0] via 172.16.1.3, 00:35:33
        C        172.16.3.0/24 is directly connected, GigabitEthernet3
        L        172.16.3.4/32 is directly connected, GigabitEthernet3
        B        172.16.6.0/24 [20/0] via 172.16.1.3, 00:49:06
	
.. Note:: The BGP route should be pointing to the 'Active' device in the HA cluster.

Initiate a failover event to determine how the route update happens.

From whichever BIG-IP device is active:

.. code-block:: none

	tmsh run sys failover standby

.. code-block:: none

        csr1000v-W_CPE_A#sh ip route
        Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
                D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
                N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
                E1 - OSPF external type 1, E2 - OSPF external type 2
                i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
                ia - IS-IS inter area, * - candidate default, U - per-user static route
                o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
                a - application route
                + - replicated route, % - next hop override, p - overrides from PfR
        
        Gateway of last resort is not set
        
                1.0.0.0/32 is subnetted, 2 subnets
        C        1.1.1.1 is directly connected, Loopback100
        O        1.1.1.2 [110/2] via 10.1.10.4, 02:51:26, GigabitEthernet4
                3.0.0.0/32 is subnetted, 1 subnets
        B        3.3.3.3 [20/0] via 172.16.1.3, 00:07:24
                10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
        C        10.1.10.0/24 is directly connected, GigabitEthernet4
        L        10.1.10.3/32 is directly connected, GigabitEthernet4
        B        10.99.99.100/32 [200/0] via 10.1.10.6, 00:00:02
                172.16.0.0/16 is variably subnetted, 6 subnets, 2 masks
        C        172.16.1.0/24 is directly connected, GigabitEthernet2
        L        172.16.1.4/32 is directly connected, GigabitEthernet2
        B        172.16.2.0/24 [20/0] via 172.16.1.3, 00:37:13
        C        172.16.3.0/24 is directly connected, GigabitEthernet3
        L        172.16.3.4/32 is directly connected, GigabitEthernet3
        B        172.16.6.0/24 [20/0] via 172.16.1.3, 00:50:46

.. Note:: If you were watching the update time you'll notice it takes a bit for the next-hop address to change. This can be modified with timers, or you can use a floating self-ip between an HA pair of BIG-IP's to minimize the update time because the Next-hop entry for the floating address never needs to be updated.

Validate that the website is accessible from the jumphost

.. code-block:: none

        root@jumphost:~# curl 10.99.99.100
        <html><body><h1>It works!</h1>
        <p>This is the default web page for this server.</p>
        <p>The web server software is running but no content has been added, yet.</p>
        </body></html>

Next we will configure a floating self-ip address to minimize the need for the next-hop to change. On one of the West Data Center BIG-IP devices. Combined with mac-masquerading and connection mirroring this is the least disruptive failover configuration.

.. code-block:: none

		tmsh create net self float-bgp address 10.1.10.7/24 vlan external traffic-group traffic-group-1

You can either wait until the route update occurs or you can reset the BGP neighbors with the clear ip bgp * command

.. code-block:: none

        [root@W_B_BIGIP-13:Active:In Sync] config # imish
        W_B_BIGIP-13.local[0]>enable
        W_B_BIGIP-13.local[0]#clear ip bgp *

.. code-block:: none

	csr1000v-W_CPE_A#sh ip route
        Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
                D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
                N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
                E1 - OSPF external type 1, E2 - OSPF external type 2
                i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
                ia - IS-IS inter area, * - candidate default, U - per-user static route
                o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
                a - application route
                + - replicated route, % - next hop override, p - overrides from PfR
        
        Gateway of last resort is not set
        
                1.0.0.0/32 is subnetted, 2 subnets
        C        1.1.1.1 is directly connected, Loopback100
        O        1.1.1.2 [110/2] via 10.1.10.4, 00:31:23, GigabitEthernet4
                2.0.0.0/32 is subnetted, 1 subnets
        B        2.2.2.2 [20/0] via 172.16.1.3, 00:30:43
                10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
        C        10.1.10.0/24 is directly connected, GigabitEthernet4
        L        10.1.10.3/32 is directly connected, GigabitEthernet4
        B        10.99.99.100/32 [200/0] via 10.1.10.7, 00:03:06
                172.16.0.0/16 is variably subnetted, 6 subnets, 2 masks
        C        172.16.1.0/24 is directly connected, GigabitEthernet2
        L        172.16.1.4/32 is directly connected, GigabitEthernet2
        B        172.16.2.0/24 [20/0] via 172.16.1.3, 00:30:44
        C        172.16.3.0/24 is directly connected, GigabitEthernet3
        L        172.16.3.4/32 is directly connected, GigabitEthernet3
        B        172.16.6.0/24 [20/0] via 172.16.1.3, 00:30:44

On the CPE devices look at the result of the show ip bgp command

.. code-block:: none

        csr1000v-W_CPE_A#sh ip bgp
        BGP table version is 10, local router ID is 1.1.1.1
        Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
                        r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
                        x best-external, a additional-path, c RIB-compressed, 
                        t secondary path, 
        Origin codes: i - IGP, e - EGP, ? - incomplete
        RPKI validation codes: V valid, I invalid, N Not found
        
                Network          Next Hop            Metric LocPrf Weight Path
            *>   1.1.1.1/32       0.0.0.0                  0         32768 i
            r>i  1.1.1.2/32       1.1.1.2                  0    100      0 i
            * i  2.2.2.2/32       172.16.2.3               0    100      0 65001 65002 65201 i
            *>                    172.16.1.3                             0 65001 65002 65201 i
            * i  10.3.99.0/24     10.1.20.254                   100      0 ?
            *>i  10.99.99.100/32  10.1.10.7                     100      0 ?
            r i  172.16.1.0/24    172.16.2.3               0    100      0 65001 ?
            r>                    172.16.1.3               0             0 65001 ?
            * i  172.16.2.0/24    172.16.2.3               0    100      0 65001 ?
            *>                    172.16.1.3               0             0 65001 ?
            * i  172.16.6.0/24    172.16.2.3               0    100      0 65001 65002 ?
            *>                    172.16.1.3                             0 65001 65002 ?
        csr1000v-W_CPE_A#

.. Note:: Compare the show ip route output in the previous step to the show ip bgp output. What looks out of place here? Why is it there? Why does it not show up in the routing table?
	


Create an aggregate route since many service providers will not accept anything less than a /24
-----------------------------------------------------------------------------------------------

On both of the West BIG-IPs configure ZebOS to only advertise an aggregate route for the virtual server network and filter out the core network advertisement.

.. code-block:: none

        [root@W_A_BIGIP-13:Active:In Sync] config # imish
        W_A_BIGIP-13.local[0]>enable
        W_A_BIGIP-13.local[0]#config t
        W_A_BIGIP-13.local[0](config)#ip prefix-list PFX_ALLOW_VIPS seq 5 permit 10.99.99.0/24
        W_A_BIGIP-13.local[0](config)#route-map RESTRICT_ADVERTISE permit 10
        W_A_BIGIP-13.local[0](config-route-map)# match ip address prefix-list PFX_ALLOW_VIPS
        W_A_BIGIP-13.local[0](config)#router bgp 65101
        W_A_BIGIP-13.local[0](config-router)# aggregate-address 10.99.99.0/24
        W_A_BIGIP-13.local[0](config-router)# neighbor 10.1.10.3 route-map RESTRICT_ADVERTISE out
        W_A_BIGIP-13.local[0](config-router)# neighbor 10.1.10.4 route-map RESTRICT_ADVERTISE out
        W_A_BIGIP-13.local[0](config-router)#end
        W_A_BIGIP-13.local[0]#wr
        Building configuration...
        [OK]
        W_A_BIGIP-13.local[0]#

.. code-block:: none

        [root@W_A_BIGIP-13:Active:In Sync] config # imish
        W_A_BIGIP-13.local[0]>enable
        W_B_BIGIP-13.local[0]# config t
        W_B_BIGIP-13.local[0](config)#ip prefix-list PFX_ALLOW_VIPS seq 5 permit 10.99.99.0/24
        W_B_BIGIP-13.local[0](config)#route-map RESTRICT_ADVERTISE permit 10
        W_B_BIGIP-13.local[0](config-route-map)# match ip address prefix-list PFX_ALLOW_VIPS
        W_B_BIGIP-13.local[0](config)#router bgp 65101
        W_B_BIGIP-13.local[0](config-router)# aggregate-address 10.99.99.0/24
        W_B_BIGIP-13.local[0](config-router)# neighbor 10.1.10.3 route-map RESTRICT_ADVERTISE out
        W_B_BIGIP-13.local[0](config-router)# neighbor 10.1.10.4 route-map RESTRICT_ADVERTISE out
        W_B_BIGIP-13.local[0](config-router)#end
        W_B_BIGIP-13.local[0]#wr
        Building configuration...
        [OK]
        W_B_BIGIP-13.local[0]#

Validate on either of the CPE routers. You can either wait until the route update occurs or you can reset the BGP neighbors with the clear ip bgp * command
	
Clear ip bgp * on the active BIG-IP such that /32 is no longer advertised.

.. code-block:: none

        [root@W_B_BIGIP-13:Active:In Sync] config # imish
        W_B_BIGIP-13.local[0]>enable
         W_B_BIGIP-13.local[0]#clear ip bgp *

.. code-block:: none

        csr1000v-W_CPE_A#sh ip route
        Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
                D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
                N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
                E1 - OSPF external type 1, E2 - OSPF external type 2
                i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
                ia - IS-IS inter area, * - candidate default, U - per-user static route
                o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
                a - application route
                + - replicated route, % - next hop override, p - overrides from PfR

        Gateway of last resort is not set
        
                1.0.0.0/32 is subnetted, 2 subnets
        C        1.1.1.1 is directly connected, Loopback100
        O        1.1.1.2 [110/2] via 10.1.10.4, 03:04:37, GigabitEthernet4
                2.0.0.0/32 is subnetted, 1 subnets
        B        2.2.2.2 [20/0] via 172.16.1.3, 00:06:50
                10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
        C        10.1.10.0/24 is directly connected, GigabitEthernet4
        L        10.1.10.3/32 is directly connected, GigabitEthernet4
        B        10.99.99.0/24 [200/0] via 10.1.10.7, 00:06:50
                172.16.0.0/16 is variably subnetted, 6 subnets, 2 masks
        C        172.16.1.0/24 is directly connected, GigabitEthernet2
        L        172.16.1.4/32 is directly connected, GigabitEthernet2
        B        172.16.2.0/24 [20/0] via 172.16.1.3, 00:06:50
        C        172.16.3.0/24 is directly connected, GigabitEthernet3
        L        172.16.3.4/32 is directly connected, GigabitEthernet3
        B        172.16.6.0/24 [20/0] via 172.16.1.3, 00:06:50

Validate that the website is accessible from the jumphost.

.. code-block:: none

        root@jumphost:~# curl 10.99.99.100
        <html><body><h1>It works!</h1>
        <p>This is the default web page for this server.</p>
        <p>The web server software is running but no content has been added, yet.</p>
        </body></html>

Add another virtual server and test the behavior of failover of the aggregate route.
----------------------------------------------------------------------------------------------------

Add another virtual server within the VIP subnet.

.. code-block:: none

        tmsh create ltm pool pool2 members add { 10.3.99.200:8081 } monitor tcp_half_open
        tmsh create ltm virtual vip2 destination 10.99.99.101:80 source-address-translation { type automap } pool pool2 profiles add { tcp http } 
	
Validate that the website is accessible from the jumphost

.. code-block:: none

        root@jumphost:~# curl 10.99.99.101
        <html><body><h1>It works!</h1>
        <p>This is the default web page for this server.</p>
        <p>The web server software is running but no content has been added, yet.</p>
        </body></html>

Disable the first pool you created.

.. code-block:: none

        tmsh modify ltm pool pool1 members modify { 10.3.99.200:http { state user-down } }

        tmsh show ltm virtual vip1
        ------------------------------------------------------------------
        Ltm::Virtual Server: vip1      
        ------------------------------------------------------------------
        Status                         
                Availability     : offline   
                State            : enabled   
                Reason           : The children pool member(s) are down
                CMP              : enabled   
                CMP Mode         : all-cpus  
                Destination      : 10.99.99.100:80

Validate that the website is still accessible from the jumphost

.. code-block:: none

        root@jumphost:~# curl 10.99.99.101
        curl: (7) Failed to connect to 10.99.99.101 port 80: No route to host

Now look at the routing table on the CPE virtual server.

.. code-block:: none

	csr1000v-W_CPE_A#sh ip route
        Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
                D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
                N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
                E1 - OSPF external type 1, E2 - OSPF external type 2
                i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
                ia - IS-IS inter area, * - candidate default, U - per-user static route
                o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
                a - application route
                + - replicated route, % - next hop override, p - overrides from PfR
        
        Gateway of last resort is not set
        
                1.0.0.0/32 is subnetted, 2 subnets
        C        1.1.1.1 is directly connected, Loopback100
        O        1.1.1.2 [110/2] via 10.1.10.4, 00:57:32, GigabitEthernet4
                2.0.0.0/32 is subnetted, 1 subnets
        B        2.2.2.2 [20/0] via 172.16.1.3, 00:15:52
                10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
        C        10.1.10.0/24 is directly connected, GigabitEthernet4
        L        10.1.10.3/32 is directly connected, GigabitEthernet4
                172.16.0.0/16 is variably subnetted, 6 subnets, 2 masks
        C        172.16.1.0/24 is directly connected, GigabitEthernet2
        L        172.16.1.4/32 is directly connected, GigabitEthernet2
        B        172.16.2.0/24 [20/0] via 172.16.1.3, 00:56:56
        C        172.16.3.0/24 is directly connected, GigabitEthernet3
        L        172.16.3.4/32 is directly connected, GigabitEthernet3
        B        172.16.6.0/24 [20/0] via 172.16.1.3, 00:56:56

On the active BIG-IP look at the routing table.

.. code-block:: none

        W_B_BIGIP-13.local[0]#sh ip route
        
        Codes: K - kernel, C - connected, S - static, R - RIP, B - BGP
                O - OSPF, IA - OSPF inter area
                N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
                E1 - OSPF external type 1, E2 - OSPF external type 2
                i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
                * - candidate default
        
        B       1.1.1.1/32 [200/0] via 10.1.10.3, external, 00:57:35
        B       1.1.1.2/32 [200/0] via 10.1.10.4, external, 00:57:35
        C       10.1.10.0/24 is directly connected, external
        C       10.1.20.0/24 is directly connected, internal
        K       10.3.99.0/24 [0/0] via 10.1.20.254, internal
        C       127.0.0.1/32 is directly connected, lo
        C       127.1.1.254/32 is directly connected, tmm
        C       192.168.255.0/24 is directly connected, ha

.. Note:: 

	There are no kernel routes from tmm. e.g. 10.99.99.0/24, 10.99.99.100, or 10.99.99.101.
	
	Why?
	
.. Note:: You need to make sure either all virtual-addresses have advertisement-enabled or a healthy majority of them enabled or the aggregate route will be withdrawn. BGP by default needs to have a route from another routing protocol available in order to advertise a route to a BGP neighbor. In this case because only one of the virtual-addresses has it enabled and it goes down, all other addresses within the /24 are considered unreachable and so the route is withdrawn.

Re-enable the pool and then review the BIG-IP and CPE route tables.

.. code-block:: none

        tmsh modify ltm pool pool1 members modify { 10.3.99.200:http { state user-up } }

.. code-block:: none

        W_B_BIGIP-13.local[0]#sh ip route
        Codes: K - kernel, C - connected, S - static, R - RIP, B - BGP
                O - OSPF, IA - OSPF inter area
                N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
                E1 - OSPF external type 1, E2 - OSPF external type 2
                i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
                * - candidate default
        
        B       1.1.1.1/32 [200/0] via 10.1.10.3, external, 00:58:09
        B       1.1.1.2/32 [200/0] via 10.1.10.4, external, 00:58:09
        C       10.1.10.0/24 is directly connected, external
        C       10.1.20.0/24 is directly connected, internal
        K       10.3.99.0/24 [0/0] via 10.1.20.254, internal
        B       10.99.99.0/24 [200/0] is a summary, Null, 00:00:04
        K       10.99.99.100/32 [0/0] is directly connected, tmm
        C       127.0.0.1/32 is directly connected, lo
        C       127.1.1.254/32 is directly connected, tmm
        C       192.168.255.0/24 is directly connected, ha
        
        Gateway of last resort is not set

.. code-block:: none

        csr1000v-W_CPE_A#sh ip route
        Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
                D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
                N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
                E1 - OSPF external type 1, E2 - OSPF external type 2
                i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
                ia - IS-IS inter area, * - candidate default, U - per-user static route
                o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
                a - application route
                + - replicated route, % - next hop override, p - overrides from PfR
        
        Gateway of last resort is not set
        
                1.0.0.0/32 is subnetted, 2 subnets
        C        1.1.1.1 is directly connected, Loopback100
        O        1.1.1.2 [110/2] via 10.1.10.4, 01:10:54, GigabitEthernet4
                2.0.0.0/32 is subnetted, 1 subnets
        B        2.2.2.2 [20/0] via 172.16.1.3, 00:29:14
                10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
        C        10.1.10.0/24 is directly connected, GigabitEthernet4
        L        10.1.10.3/32 is directly connected, GigabitEthernet4
        B        10.99.99.0/24 [200/0] via 10.1.10.7, 00:09:42
                172.16.0.0/16 is variably subnetted, 6 subnets, 2 masks
        C        172.16.1.0/24 is directly connected, GigabitEthernet2
        L        172.16.1.4/32 is directly connected, GigabitEthernet2
        B        172.16.2.0/24 [20/0] via 172.16.1.3, 01:10:18
        C        172.16.3.0/24 is directly connected, GigabitEthernet3
        L        172.16.3.4/32 is directly connected, GigabitEthernet3
        B        172.16.6.0/24 [20/0] via 172.16.1.3, 01:10:18

Validate that the websites are accessible from the jumphost

.. code-block:: none

        root@jumphost:~# curl 10.99.99.100
        <html><body><h1>It works!</h1>
        <p>This is the default web page for this server.</p>
        <p>The web server software is running but no content has been added, yet.</p>
        </body></html>

.. code-block:: none

        root@jumphost:~# curl 10.99.99.101
        <html><body><h1>It works!</h1>
        <p>This is the default web page for this server.</p>
        <p>The web server software is running but no content has been added, yet.</p>
        </body></html>
	
Modify the virtual-address for the second virtual server to allow route-advertisement and repeat disabling.

.. code-block:: none

        tmsh modify ltm virtual-address 10.99.99.101 route-advertisement selective 
        tmsh modify ltm pool pool1 members modify { 10.3.99.200:http { state user-down } }

.. code-block:: none

        csr1000v-W_CPE_A#sh ip route
        Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
                D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
                N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
                E1 - OSPF external type 1, E2 - OSPF external type 2
                i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
                ia - IS-IS inter area, * - candidate default, U - per-user static route
                o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
                a - application route
                + - replicated route, % - next hop override, p - overrides from PfR
        
        Gateway of last resort is not set
        
                1.0.0.0/32 is subnetted, 2 subnets
        C        1.1.1.1 is directly connected, Loopback100
        O        1.1.1.2 [110/2] via 10.1.10.4, 01:21:27, GigabitEthernet4
                2.0.0.0/32 is subnetted, 1 subnets
        B        2.2.2.2 [20/0] via 172.16.1.3, 00:39:47
                10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
        C        10.1.10.0/24 is directly connected, GigabitEthernet4
        L        10.1.10.3/32 is directly connected, GigabitEthernet4
        B        10.99.99.0/24 [200/0] via 10.1.10.7, 00:20:15
                172.16.0.0/16 is variably subnetted, 6 subnets, 2 masks
        C        172.16.1.0/24 is directly connected, GigabitEthernet2
        L        172.16.1.4/32 is directly connected, GigabitEthernet2
        B        172.16.2.0/24 [20/0] via 172.16.1.3, 01:20:51
        C        172.16.3.0/24 is directly connected, GigabitEthernet3
        L        172.16.3.4/32 is directly connected, GigabitEthernet3
        B        172.16.6.0/24 [20/0] via 172.16.1.3, 01:20:51

The route now remained when one of the virtual servers went down.

Validate that .101 website remained up while the .100 is not.

.. code-block:: none

        root@jumphost:~# curl 10.99.99.100
        curl: (56) Recv failure: Connection reset by peer

		
.. code-block:: none

        root@jumphost:~# curl 10.99.99.101
        <html><body><h1>It works!</h1>
        <p>This is the default web page for this server.</p>
        <p>The web server software is running but no content has been added, yet.</p>
        </body></html>

You can re-enable the .100 website before moving on to Module 2.

.. code-block:: none

        tmsh modify ltm pool pool1 members modify { 10.3.99.200:http { state user-up } }

Validate that the websites are accessible from the jumphost

.. code-block:: none

        root@jumphost:~# curl 10.99.99.101
        <html><body><h1>It works!</h1>
        <p>This is the default web page for this server.</p>
        <p>The web server software is running but no content has been added, yet.</p>
        </body></html>

        root@jumphost:~# curl 10.99.99.100
        <html><body><h1>It works!</h1>
        <p>This is the default web page for this server.</p>
        <p>The web server software is running but no content has been added, yet.</p>
        </body></html>

Sample configuration
--------------------

.. code-block:: none

        Sample complete configuration from one BIG-IP:
                !
                W_B_BIGIP-13.local[0]#sh run
                !
                no service password-encryption
                !
                router bgp 65101
                    aggregate-address 10.99.99.0/24
                    redistribute kernel
                    neighbor 10.1.10.3 remote-as 65101
                    neighbor 10.1.10.3 route-map RESTRICT_ADVERTISE out
                    neighbor 10.1.10.4 remote-as 65101
                    neighbor 10.1.10.4 route-map RESTRICT_ADVERTISE out
                    neighbor 10.1.10.5 remote-as 65101
                !
                ip prefix-list PFX_ALLOW_VIPS seq 5 permit 10.99.99.0/24
                !
                route-map RESTRICT_ADVERTISE permit 10
                    match ip address prefix-list PFX_ALLOW_VIPS
                !
                line con 0
                    login
                line vty 0 39
                    login
                !
                end

	
.. Note:: This completes Lab 1