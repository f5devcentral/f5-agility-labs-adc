Lab 2:  East Data Center Configuration
======================================

In this lab you will be configuring two BIG-IP standalone devices and setting it up to advertise available virtual servers through the BGP protocol.

Configure the BIG-IP East Data Center
------------------------------------------------

From the jumphost ssh to the E_A_BIGIP-13

.. code-block:: none 

    root@jumphost:~# ssh -l root 192.168.1.8

.. code-block:: none 

    tmsh modify sys ntp servers add { pool.ntp.org }

From the jumphost ssh to the E_B_BIGIP-13

.. code-block:: none

    root@jumphost:~# ssh -l root 192.168.1.4

.. code-block:: none

		tmsh modify sys ntp servers add { pool.ntp.org }

Save the configuration on both devices

.. code-block:: none

    tmsh save sys config partitions all

Configure the eBGP session between E_A_BIGIP-13 and the East Core via csr1000v-CORE.
------------------------------------------------------------------------------------

.. Note:: 
    The CORE Router configuration is already done for you so you only need to configure the two EAST BIG-IP devices.


E_A_BIGIP-13:  On E_A_BIGIP configure the following to enable BGP for route-domain 0. 
		
.. code-block:: none

    tmsh modify net route-domain 0 routing-protocol add { BGP }
		
    tmsh save sys config partitions all

E_A_BIGIP-13:  Bring up BGP with the East Core and configure the keepalive interval to 3 seconds and hold time to 9 seconds via neighbor statement instead of using the default values of 30 seconds for keepalive and 180 seconds for hold time:

.. code-block:: none

    [root@E_A_BIGIP-13:Active:In Sync] config # imish -r 0
        E_A_BIGIP-13.local[0]>enable
        E_A_BIGIP-13.local[0]#config t
        E_A_BIGIP-13.local[0](config)#router bgp 65202
        E_A_BIGIP-13.local[0](config-router)# neighbor 10.2.40.4 remote-as 65205
        E_A_BIGIP-13.local[0](config-router)# neighbor 10.2.40.4 description E_CORE
        E_A_BIGIP-13.local[0](config-router)# neighbor 10.2.40.4 timers 3 9
        E_A_BIGIP-13.local[0](config-router)# end
        E_A_BIGIP-13.local[0]#write mem
			
			
E_A_BIGIP-13:  Verify eBGP adjacencies are up between E_A_BIGIP-13 and the East Core router - csr1000v-CORE. 
			
.. code-block:: none

    E_A_BIGIP-13.local[0]#sh ip bgp sum
    BGP router identifier 10.2.40.3, local AS number 65202
    BGP table version is 2
    1 BGP AS-PATH entries
    0 BGP community entries
    
    Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    10.2.40.4       4 65205       5       2        2    0    0 00:00:22        2 
			
E_A_BIGIP-13:  Verify route for the webservice via 10.3.99.0/24 is installed in routing table after eBGP is established between E_A_BIGIP-13 and the East Core router - csr1000v-CORE. 
			
.. code-block:: none

    E_A_BIGIP-13.local[0]#sh ip route | i 10.2.40.4
    B       3.3.3.3/32 [20/0] via 10.2.40.4, internal, 00:01:54
    B       10.3.99.0/24 [20/0] via 10.2.40.4, internal, 00:01:54
    E_A_BIGIP-13.local[0]
			
Configure the eBGP session between E_B_BIGIP-13 and the East Core via csr1000v-CORE.
------------------------------------------------------------------------------------

.. Note:: 
    The CORE Router configuration is already done for you so you only need to configure the two EAST BIG-IP devices.


E_B_BIGIP-13:  On E_B_BIGIP configure the following to enable BGP for route-domain 0. 
		
.. code-block:: none

    tmsh modify net route-domain 0 routing-protocol add { BGP }
		
    tmsh save sys config partitions all

E_B_BIGIP-13:  Bring up BGP with the East Core and configure the keepalive interval to 3 seconds and hold time to 9 seconds via neighbor statement instead of using the default values of 30 seconds for keepalive and 180 seconds for hold time:

.. code-block:: none

    [root@E_B_BIGIP-13:Active:In Sync] config # imish -r 0
        E_B_BIGIP-13.local[0]>enable
        E_B_BIGIP-13.local[0]#config t
        E_B_BIGIP-13.local[0](config)#router bgp 65203
        E_B_BIGIP-13.local[0](config-router)# neighbor 10.2.50.4 remote-as 65205
        E_B_BIGIP-13.local[0](config-router)# neighbor 10.2.50.4 description E_CORE
        E_B_BIGIP-13.local[0](config-router)# neighbor 10.2.50.4 timers 3 9
        E_A_BIGIP-13.local[0](config-router)# end
        E_A_BIGIP-13.local[0]#write mem
				
E_B_BIGIP-13:  Verify eBGP adjacencies are up between E_B_BIGIP-13 and the East Core router - csr1000v-CORE. 
			
.. code-block:: none

    E_B_BIGIP-13.local[0]#sh ip bgp sum
    BGP router identifier 10.2.50.3, local AS number 65203
    BGP table version is 2
    1 BGP AS-PATH entries
    0 BGP community entries
    
    Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    10.2.50.4       4 65205       4       2        1    0    0 00:00:16        2
    
    Total number of neighbors 1
    E_B_BIGIP-13.local[0]#
			
E_B_BIGIP-13:  Verify route for the webservice via 10.3.99.0/24 is installed in routing table after eBGP is established between E_B_BIGIP-13 and the East Core router - csr1000v-CORE. 
			
.. code-block:: none

    E_B_BIGIP-13.local[0]#sh ip route | i 10.2.50.4
    B       3.3.3.3/32 [20/0] via 10.2.50.4, internal, 00:06:52
    B       10.3.99.0/24 [20/0] via 10.2.50.4, internal, 00:06:52
    E_B_BIGIP-13.local[0]#

Validate Webserver Connectivity via Core Network:
-------------------------------------------------

Verify that you can reach the webserver on the core network with icmp ping and curl from both BIG-IPs.

E_A_BIGIP-13:  Ping the webserver @ 10.3.99.200 via the core network from E_A_BIGIP-13.  Note the ping may not work immediately and could take a few seconds.  Also Note the escape sequence for icmp ping on the BIGIP is “CTRL + C”
		
.. code-block:: none

    [root@E_A_BIGIP-13:Active:Standalone] config # ping 10.3.99.200
    PING 10.3.99.200 (10.3.99.200) 56(84) bytes of data.
    64 bytes from 10.3.99.200: icmp_seq=1 ttl=63 time=8.51 ms
    64 bytes from 10.3.99.200: icmp_seq=2 ttl=63 time=8.12 ms
    ^C
    --- 10.3.99.200 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1823ms
    rtt min/avg/max/mdev = 8.121/8.318/8.516/0.217 ms

E_A_BIGIP-13:  Curl the webserver @ 10.3.99.200 via the core network from E_A_BIGIP-13.

.. code-block:: none

    [root@E_A_BIGIP-13:Active:Standalone] config # curl 10.3.99.200
    <html><body><h1>It works!</h1>
    <p>This is the default web page for this server.</p>
    <p>The web server software is running but no content has been added, yet.</p>
    </body></html>
		
E_B_BIGIP-13:  Ping the webserver @ 10.3.99.200 via the core network from E_B_BIGIP-13.  Note the ping may not work immediately and could take a few seconds.  Also Note the escape sequence for icmp ping on the BIGIP is “CTRL + C”
		
.. code-block:: none

    [root@E_B_BIGIP-13:Active:Standalone] config # ping 10.3.99.200
    PING 10.3.99.200 (10.3.99.200) 56(84) bytes of data.
    64 bytes from 10.3.99.200: icmp_seq=1 ttl=63 time=6.06 ms
    64 bytes from 10.3.99.200: icmp_seq=2 ttl=63 time=9.31 ms
    ^C
    --- 10.3.99.200 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1726ms
    rtt min/avg/max/mdev = 6.068/7.692/9.317/1.626 ms

E_B_BIGIP-13:  Curl the webserver @ 10.3.99.200 via the core network from E_B_BIGIP-13.

.. code-block:: none

    [root@E_B_BIGIP-13:Active:Standalone] config # curl 10.3.99.200
    <html><body><h1>It works!</h1>
    <p>This is the default web page for this server.</p>
    <p>The web server software is running but no content has been added, yet.</p>
    </body></html>
    [root@E_B_BIGIP-13:Active:Standalone] config # 
		
			
Create an application configuration for a virtual server and a pool member on E_A_BIGIP-13:
-------------------------------------------------------------------------------------------
			
E_A_BIGIP-13:  Create the following virtual server and pool member on E_A_BIGIP-13:
			
.. code-block:: none

    tmsh create ltm pool pool1 members add { 10.3.99.200:80 } monitor tcp_half_open
    tmsh create ltm virtual vip1 destination 10.99.99.102:80 source-address-translation { type automap } pool pool1 profiles add { tcp http }
    
    tmsh save sys config partitions all
			
E_A_BIGIP-13:  Your virtual server should now show available on E_A_BIGIP-13:
	
.. code-block:: none

    [root@E_A_BIGIP-13:Active:Standalone] config # tmsh show ltm virtual
 
    ------------------------------------------------------------------
    Ltm::Virtual Server: vip1      
    ------------------------------------------------------------------
    Status                         
      Availability     : available 
      State            : enabled   
      Reason           : The virtual server is available
      CMP              : enabled   
      CMP Mode         : all-cpus  
      Destination      : 10.99.99.102:80
                    
Configure the route advertisement on the E_A_BIGIP-13:
------------------------------------------------------
	
E_A_BIGIP-13: Configure the eBGP session on E_A_BIGIP to East CPE_A. The CPE configuration is already done for you so you only need to configure the BIGIP side of session.
	
.. code-block:: none

    [root@E_A_BIGIP-13:Active:In Sync] config # imish -r 0
        E_A_BIGIP-13.local[0]>enable
        E_A_BIGIP-13.local[0]#config t
        E_A_BIGIP-13.local[0](config)#router bgp 65202
        E_A_BIGIP-13.local[0](config-router)# neighbor 10.2.20.4 remote-as 65201
        E_A_BIGIP-13.local[0](config-router)# neighbor 10.2.20.4 description E_CPE_A
        E_A_BIGIP-13.local[0](config-router)# neighbor 10.2.20.4 timers 3 9
        E_A_BIGIP-13.local[0](config-router)# end
        E_A_BIGIP-13.local[0]#write mem
			
E_A_BIGIP-13:  Verify eBGP adjacencies are up between E_A_BIGIP_13 and the East CPE_A. 
			
.. code-block:: none

    E_A_BIGIP-13.local[0]#sh ip bgp sum
    BGP router identifier 10.2.40.3, local AS number 65202
    BGP table version is 3
    5 BGP AS-PATH entries
    0 BGP community entries
    
    Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    10.2.20.4       4 65201      27      21        3    0    0 00:00:53        8
    10.2.40.4       4 65205     157     141        3    0    0 01:07:25        2
			
E_A_BIGIP-13:  On E_A_BIGIP configure the following network statement for 10.99.99.0/24 such that prefix is originated locally:

.. code-block:: none

    [root@E_A_BIGIP-13:Active:Standalone] config # imish -r 0
        E_A_BIGIP-13.local[0]>en
        E_A_BIGIP-13.local[0]#
        E_A_BIGIP-13.local[0]#conf t
        Enter configuration commands, one per line.  End with CNTL/Z.
        E_A_BIGIP-13.local[0](config)#router bgp 65202
        E_A_BIGIP-13.local[0](config-router)#network 10.99.99.0/24
        E_A_BIGIP-13.local[0](config-router)#end
        E_A_BIGIP-13.local[0]#
		
E_A_BIGIP-13:  On E_A_BIGIP verify 10.99.99.0/24 is being locally originated which can be seen with “Local”:
		
.. code-block:: none

    E_A_BIGIP-13.local[0]#sh ip bgp 10.99.99.0/24 | b Local
    
    ...skipping
        Local
        0.0.0.0 from 0.0.0.0 (10.2.40.3)
            Origin IGP, localpref 100, weight 32768, valid, sourced, local, best
            Last update: Mon Jul 16 18:07:54 2018
		
E_A_BIGIP-13:  On E_A_BIGIP verify 10.99.99.0/24 is being advertised outbound to East CPE device via E_CPE_A_CSR1k:

.. code-block:: none
    
    E_A_BIGIP-13.local[0]#sh ip bgp neighbor 10.2.20.4 advertised-routes | i 10.99.99.0/24
    *> 10.99.99.0/24    10.2.20.3                         100      32768 i
    
E_CPE_A_CSR1k:  Verify that E_CPE_A_CSR1k is learning the 10.99.99.0/24 inbound from E_A_BIGIP:

.. Note::
    You can telnet to  the CPE devices using the BGP neighbor IP address from Zebos using root/default for user/pass:
 
Example telnet from E_A_BIGIP-13 to East CPE Device @ 10.2.20.4:
 
.. code-block:: none

    E_A_BIGIP-13.local[0]#telnet 10.2.20.4
    Trying 10.2.20.4...
    Connected to 10.2.20.4.
    Escape character is '^]'.
    
    User Access Verification
    
    Username: root
    Password: 
    csr1000v-E_CPE_A>
    	

E_CPE_A_CSR1k:  Continued...Verify that E_CPE_A_CSR1k is learning the 10.99.99.0/24 inbound from E_A_BIGIP:

.. code-block:: none

    csr1000v-E_CPE_A>show ip bgp vpnv4 vrf internet neighbors 10.2.20.3 routes
    BGP table version is 26, local router ID is 2.2.2.2
    Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
                r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
                x best-external, a additional-path, c RIB-compressed, 
                t secondary path, 
    Origin codes: i - IGP, e - EGP, ? - incomplete
    RPKI validation codes: V valid, I invalid, N Not found
    
        Network          Next Hop            Metric LocPrf Weight Path
    Route Distinguisher: 65201:1000 (default for vrf internet)
    *>   3.3.3.3/32       10.2.20.3       4294967295             0 65202 65205 i
    *>   10.99.99.0/24    10.2.20.3       4294967295             0 65202 i
    
    Total number of prefixes 2 
    csr1000v-E_CPE_A>
		

E_CPE_A_CSR1k: Verify that E_CPE_A_CSR1k is installing 10.99.99.0/24 from E_A_BIGIP:
		
.. code-block:: none

    csr1000v-E_CPE_A>show ip route vrf internet
 
    Routing Table: internet
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
    B        1.1.1.1 [20/4294967294] via 172.16.6.3, 22:12:30
    B        1.1.1.2 [20/4294967294] via 172.16.6.3, 22:12:32
        3.0.0.0/32 is subnetted, 1 subnets
    B        3.3.3.3 [20/4294967294] via 10.2.20.3, 01:27:19
        10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
    C        10.2.20.0/24 is directly connected, GigabitEthernet2
    L        10.2.20.4/32 is directly connected, GigabitEthernet2
    C        10.2.30.0/24 is directly connected, GigabitEthernet3
    L        10.2.30.4/32 is directly connected, GigabitEthernet3
    B        10.99.99.0/24 [20/4294967294] via 10.2.20.3, 01:22:58
        99.0.0.0/24 is subnetted, 1 subnets
    B        99.99.99.0 [20/4294967294] via 172.16.6.3, 22:13:59
        172.16.0.0/16 is variably subnetted, 5 subnets, 2 masks
    B        172.16.1.0/24 [20/4294967294] via 172.16.6.3, 22:12:40
    B        172.16.2.0/24 [20/4294967294] via 172.16.6.3, 22:12:40
    C        172.16.6.0/24 is directly connected, GigabitEthernet5
    L        172.16.6.4/32 is directly connected, GigabitEthernet5
    B        172.16.99.0/24 [20/0] via 172.16.6.3, 22:13:59
    csr1000v-E_CPE_A>

 
E_CPE_A_CSR1k:  Verify that E_CPE_A_CSR1k is installing specific 10.99.99.0/24 from E_A_BIGIP using specific ip route command:  

.. code-block:: none

    csr1000v-E_CPE_A#sh ip route vrf internet 10.99.99.0 255.255.255.0
		
		Routing Table: internet
		Routing entry for 10.99.99.0/24
		  Known via "bgp 65201", distance 20, metric 4294967294
		  Tag 65202, type external
		  Last update from 10.2.20.3 00:00:02 ago
		  Routing Descriptor Blocks:
		  * 10.2.20.3, from 10.2.20.3, 00:00:02 ago
		      Route metric is 4294967294, traffic share count is 1
		      AS Hops 1
		      Route tag 65202
		      MPLS label: none


csr1000v-SP_C: Verify that csr1000v-SP_C is installing 10.99.99.0/24 via East DC because Origin attribute is IGP versus incomplete via for West DC:

You can telnet to csr1000v-SP_C from the jumpbox @ 192.168.1.15 with root/default user/pass:

.. code-block:: none

    ubuntu@jumphost:~$ telnet 192.168.1.15
    Trying 192.168.1.15...
    Connected to 192.168.1.15.
    Escape character is '^]'.
    
    
    User Access Verification
    
    Username: root
    Password: 
    csr1000v-SP_C>


csr1000v-SP_C:  Verify that csr1000v-SP_C is installing 10.99.99.0/24 in BGP table via East DC.  Note the Best path is via AS 65002 988.		 

.. code-block:: none

    csr1000v-SP_C>sh ip bgp 10.99.99.0/24
    BGP routing table entry for 10.99.99.0/24, version 16
    Paths: (2 available, best #1, table default)
        Advertised to update-groups:
            1         
        Refresh Epoch 1
        65002 988
        172.16.99.4 from 172.16.99.4 (172.1.1.2)
            Origin IGP, localpref 100, valid, external, best
            rx pathid: 0, tx pathid: 0x0
        Refresh Epoch 1
        65001 65101, (aggregated by 65101 192.168.255.10)
        172.16.99.3 from 172.16.99.3 (172.1.1.1)
            Origin incomplete, localpref 100, valid, external, atomic-aggregate
            rx pathid: 0, tx pathid: 0

.. Note::
    The BGP peering between E_CPE_A and SP_B leverages AS 988 as seen below.  The following command replaces the local private AS Path with 988 for prefixes originated from East DC to the SP Cloud. 

    .. code-block:: none

        csr1000v-E_CPE_A#sh run | i 988
        neighbor 172.16.6.3 local-as 988 no-prepend replace-as
        csr1000v-E_CPE_A#


csr1000v-SP_C:  Verify that csr1000v-SP_C is installing 10.99.99.0/24 in the ip routing table:

.. code-block:: none

    csr1000v-SP_C>sh ip route 10.99.99.0 255.255.255.0                   
    Routing entry for 10.99.99.0/24
        Known via "bgp 65003", distance 20, metric 0
        Tag 65002, type external
        Last update from 172.16.99.4 00:18:45 ago
        Routing Descriptor Blocks:
        * 172.16.99.4, from 172.16.99.4, 00:18:45 ago
            Route metric is 0, traffic share count is 1
            AS Hops 2
            Route tag 65002
            MPLS label: none

    
.. Note::
    From the jump host you can now try to reach the website via E_A_BIGIP and validate the path is installed via EAST DC.  
		
Either open a web browser and browse to http://10.99.99.102 or from the jumpbox CLI, type:
		    
    *curl http://10.99.99.102*

Jumpbox:  Curl from the jumphost to the virtual server.

.. code-block:: none

    root@jumphost:~# curl 10.99.99.102
		<html><body><h1>It works!</h1>
		<p>This is the default web page for this server.</p>
		<p>The web server software is running but no content has been added, yet.</p>
		</body></html>


Jumpbox:  Traceroute from the jumphost to the virtual server to verify the path it is taking.

.. code-block:: none

    root@jumphost:~# traceroute 10.99.99.102

		traceroute to 10.99.99.102 (10.99.99.102), 30 hops max, 60 byte packets
		 1  192.168.1.15 (192.168.1.15)  7.202 ms  8.251 ms  8.049 ms
		 2  172.16.99.4 (172.16.99.4)  22.485 ms  23.834 ms  36.059 ms
		 3  172.16.6.4 (172.16.6.4)  40.575 ms  40.425 ms  62.741 ms
		 4  10.99.99.102 (10.99.99.102)  64.284 ms  64.026 ms  91.206 ms
		root@jumphost:~# 

		
E_CPE_A_CSR1k:  You can also validate from the CPE with telnet to 10.99.99.102 on port 80.  Note that you can clear the telnet session by executing “clear line vty 0” on the console of the CPE:
		
.. code-block:: none

    csr1000v-E_CPE_A>telnet 10.99.99.102 80 /vrf internet
    Trying 10.99.99.102, 80 ... Open


csr1000v-SP_C:  You can also validate via traceroute to 10.99.99.102 on SP_C:

.. code-block:: none
 
        csr1000v-SP_C>traceroute 10.99.99.102
        Type escape sequence to abort.
        Tracing the route to 10.99.99.102
        VRF info: (vrf in name/id, vrf out name/id)
            1 172.16.99.4 [AS 65001] 7 msec 5 msec 9 msec
            2 172.16.6.4 [AS 65002] 10 msec 10 msec 14 msec
            3 10.99.99.102 [AS 988] 13 msec 13 msec 15 msec
        csr1000v-SP_C>
  		

.. Note::
    Now lets move on and configure BGP on E_B_BIGIP.....

Create an application configuration for a virtual server and a pool member on E_B_BIGIP-13:
-----------------------------------------------------------------------------------------------------------------
			
E_B_BIGIP-13:  Create the following virtual server and pool member on E_B_BIGIP-13:
			
.. code-block:: none

    tmsh create ltm pool pool1 members add { 10.3.99.200:80 } monitor tcp_half_open
    tmsh create ltm virtual vip1 destination 10.99.99.102:80 source-address-translation { type automap } pool pool1 profiles add { tcp http }
    
    tmsh save sys config partitions all


E_B_BIGIP-13:  Your virtual server should now show available on E_B_BIGIP-13:
 
.. code-block:: none

       [root@E_B_BIGIP-13:Active:Standalone] config # tmsh show ltm virtual
        
        ------------------------------------------------------------------
        Ltm::Virtual Server: vip1      
        ------------------------------------------------------------------
        Status                         
        Availability     : available 
        State            : enabled   
        Reason           : The virtual server is available
        CMP              : enabled   
        CMP Mode         : all-cpus  
        Destination      : 10.99.99.102:80


Configure the route advertisement on the E_B_BIGIP-13:
------------------------------------------------------

E_B_BIGIP-13:  Configure the eBGP session on E_B_BIGIP-13 to East CPE_A. The CPE configuration is already done for you so you only need to configure the BIG-IP side of session.
	
.. code-block:: none

    [root@E_B_BIGIP-13:Active:In Sync] config # imish -r 0
        E_B_BIGIP-13.local[0]>enable
        E_B_BIGIP-13.local[0]#config t
        E_B_BIGIP-13.local[0](config)#router bgp 65203
        E_B_BIGIP-13.local[0](config-router)# neighbor 10.2.30.4 remote-as 65201
        E_B_BIGIP-13.local[0](config-router)# neighbor 10.2.30.4 description E_CPE_A
        E_B_BIGIP-13.local[0](config-router)# neighbor 10.2.30.4 timers 3 9
        E_A_BIGIP-13.local[0](config-router)# end
        E_A_BIGIP-13.local[0]#write mem

			
E_B_BIGIP-13: Verify eBGP adjacencies are up between E_B_BIGIP-13 and the East CPE router - E_CPE_A_CSR1k. 
			
.. code-block:: none

    E_B_BIGIP-13.local[0]#sh ip bgp sum
    BGP router identifier 10.2.50.3, local AS number 65203
    BGP table version is 6
    12 BGP AS-PATH entries
    0 BGP community entries
    
    Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
    10.2.30.4       4 65201      17      14        5    0    0 00:00:18        9
    10.2.50.4       4 65205     385     350        6    0    0 02:52:20       10
 
    Total number of neighbors 2
    E_B_BIGIP-13.local[0]#


E_B_BIGIP-13: On E_B_BIGIP configure the following network statement for 10.99.99.0/24 such that prefix is originated locally:
		
.. code-block:: none

    [root@E_B_BIGIP-13:Active:Standalone] config # imish -r 0
    E_B_BIGIP-13.local[0]>en
    E_B_BIGIP-13.local[0]#conf t
    Enter configuration commands, one per line.  End with CNTL/Z.
    E_B_BIGIP-13.local[0](config)#router bgp 65203
    E_B_BIGIP-13.local[0](config-router)#network 10.99.99.0/24
    E_B_BIGIP-13.local[0](config-router)#end
    E_B_BIGIP-13.local[0]#wr
    Building configuration...
    [OK]
    E_B_BIGIP-13.local[0]#

		
E_B_BIGIP-13:  On E_B_BIGIP verify 10.99.99.0/24 is being locally originated which can be seen with “Local”:
		
.. code-block:: none

    E_B_BIGIP-13.local[0]#sh ip bgp 10.99.99.0/24 | b Local
    
    ...skipping
        Local
        0.0.0.0 from 0.0.0.0 (10.2.50.3)
            Origin IGP, localpref 100, weight 32768, valid, sourced, local, best
            Last update: Mon Jul 16 19:29:34 2018
    
        65201 65202
        10.2.30.4 from 10.2.30.4 (2.2.2.2)
            Origin IGP metric 0, localpref 100, valid, external
            Last update: Mon Jul 16 19:02:03 2018
    
        65205 65202
        10.2.50.4 from 10.2.50.4 (3.3.3.3)
            Origin IGP metric 0, localpref 100, valid, external
            Last update: Mon Jul 16 19:02:03 2018

		
E_B_BIGIP-13: On E_B_BIGIP verify 10.99.99.0/24 is being advertised outbound to East CPE device via E_CPE_A_CSR1k:
		
.. code-block:: none

    E_B_BIGIP-13.local[0]#sh ip bgp nei 10.2.30.4 advertised-routes | i 10.99.99.0/24
    *> 10.99.99.0/24    10.2.30.3                         100      32768 i

    
E_CPE_A_CSR1k: Verify that E_CPE_A_CSR1k is learning the 10.99.99.0/24 inbound from E_B_BIGIP:

Note:  You can telnet to  the CPE devices using the BGP neighbor IP address from Zebos using root/default for user/pass:
 
Example telnet from E_B_BIGIP-13 to East CPE Device @ 10.2.30.4:
 
.. code-block:: none

    E_B_BIGIP-13.local[0]#telnet 10.2.30.4
    Trying 10.2.30.4...
    Connected to 10.2.30.4.
    Escape character is '^]'.
    
    
    User Access Verification
    
    Username: root
    Password: 
    csr1000v-E_CPE_A>


E_CPE_A_CSR1k:  Continued...Verify that E_CPE_A_CSR1k is learning the 10.99.99.0/24 inbound from E_B_BIGIP:

.. code-block:: none

    csr1000v-E_CPE_A>show ip bgp vpnv4 vrf internet neighbors 10.2.30.3 routes
    BGP table version is 28, local router ID is 2.2.2.2
    Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
                r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
                x best-external, a additional-path, c RIB-compressed, 
                t secondary path, 
    Origin codes: i - IGP, e - EGP, ? - incomplete
    RPKI validation codes: V valid, I invalid, N Not found
    
        Network          Next Hop            Metric LocPrf Weight Path
    Route Distinguisher: 65201:1000 (default for vrf internet)
    *m   3.3.3.3/32       10.2.30.3       4294967295             0 65203 65205 i
    *m   10.99.99.0/24    10.2.30.3       4294967295             0 65203 i
    
    Total number of prefixes 2 

    
E_CPE_A_CSR1k:  Verify that E_CPE_A_CSR1k is installing 10.99.99.0/24 from E_B_BIGIP using ip route command.  Notice the next hop of E_A_BIGIP @ 10.2.20.3  & E_B_BIGIP @ 10.2.30.3:
		
.. code-block:: none

    csr1000v-E_CPE_A>sh ip route vrf internet
    
    Routing Table: internet
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
    B        1.1.1.1 [20/4294967294] via 172.16.6.3, 23:39:09
    B        1.1.1.2 [20/4294967294] via 172.16.6.3, 23:39:11
        3.0.0.0/32 is subnetted, 1 subnets
    B        3.3.3.3 [20/4294967294] via 10.2.30.3, 00:37:06
                    [20/4294967294] via 10.2.20.3, 00:37:06
        10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
    C        10.2.20.0/24 is directly connected, GigabitEthernet2
    L        10.2.20.4/32 is directly connected, GigabitEthernet2
    C        10.2.30.0/24 is directly connected, GigabitEthernet3
    L        10.2.30.4/32 is directly connected, GigabitEthernet3
    B        10.99.99.0/24 [20/4294967294] via 10.2.30.3, 00:34:33
                        [20/4294967294] via 10.2.20.3, 00:34:33
        99.0.0.0/24 is subnetted, 1 subnets
    B        99.99.99.0 [20/4294967294] via 172.16.6.3, 23:40:38
        172.16.0.0/16 is variably subnetted, 5 subnets, 2 masks
    B        172.16.1.0/24 [20/4294967294] via 172.16.6.3, 23:39:19
    B        172.16.2.0/24 [20/4294967294] via 172.16.6.3, 23:39:19
    C        172.16.6.0/24 is directly connected, GigabitEthernet5
    L        172.16.6.4/32 is directly connected, GigabitEthernet5
    B        172.16.99.0/24 [20/0] via 172.16.6.3, 23:40:38

    
E_CPE_A_CSR1k:  Verify that E_CPE_A_CSR1k is now installing specific 10.99.99.0/24 from E_B_BIGIP using specific ip route command.   Notice the next hop of E_A_BIGIP @ 10.2.20.3  & E_B_BIGIP @ 10.2.30.3:

.. code-block:: none

    csr1000v-E_CPE_A>sh ip route vrf internet 10.99.99.0 255.255.255.0
    
    Routing Table: internet
    Routing entry for 10.99.99.0/24
    Known via "bgp 65201", distance 20, metric 4294967294
    Tag 65202, type external
    Last update from 10.2.20.3 00:39:40 ago
    Routing Descriptor Blocks:
    * 10.2.30.3, from 10.2.30.3, 00:39:40 ago
        Route metric is 4294967294, traffic share count is 1
        AS Hops 1
        Route tag 65202
        MPLS label: none
        10.2.20.3, from 10.2.20.3, 00:39:40 ago
        Route metric is 4294967294, traffic share count is 1
        AS Hops 1
        Route tag 65202
        MPLS label: none
    csr1000v-E_CPE_A>
		

.. Note::
    Congratulations!  You now have eBGP Multipath Loadsharing working within the East DC!  As seen above, this will trigger ECMP for 10.99.99.0/24 on E_CPE_A_CSR1k towards E_A_BIGIP and E_B_BIGIP.  Note that normally the weight, local preference, AS path length, origin, med, etc. would need to be the same for the parallel routes to be installed in the routing table. 
    
    It is worth noting that this behavior various from version to version of IOS.  In this lab, we are using IOS-XE Version 16.3.6.  With this version, the entire AS path needs to be the same for multipath condition to be met. 
    
    How did we work around this?  The following hidden command is used on the CPE to ignore the different AS Path and install the route as multipath if all other conditions are met:

.. code-block:: none

    csr1000v-E_CPE_A#sh run | i as-path
    bgp bestpath as-path multipath-relax
    csr1000v-E_CPE_A#


csr1000v-SP_C:  Verify that nothing changed on csr1000v-SP_C and it is still installing 10.99.99.0/24 via East DC because Origin attribute is IGP versus incomplete for West DC:
		 
csr1000v-SP_C:  Verify that csr1000v-SP_C is installing 10.99.99.0/24 in BGP table via East DC.  Recall the Best path leveraging EAST DC is via AS 65002 988.

.. code-block:: none

    csr1000v-SP_C>sh ip bgp 10.99.99.0/24
    BGP routing table entry for 10.99.99.0/24, version 16
    Paths: (2 available, best #1, table default)
        Advertised to update-groups:
            1         
        Refresh Epoch 1
        65002 988
        172.16.99.4 from 172.16.99.4 (172.1.1.2)
            Origin IGP, localpref 100, valid, external, best
            rx pathid: 0, tx pathid: 0x0
        Refresh Epoch 1
        65001 65101, (aggregated by 65101 192.168.255.10)
        172.16.99.3 from 172.16.99.3 (172.1.1.1)
            Origin incomplete, localpref 100, valid, external, atomic-aggregate
            rx pathid: 0, tx pathid: 0


csr1000v-SP_C:  Verify that csr1000v-SP_C is installing 10.99.99.0/24 in the ip routing table:

.. code-block:: none

    csr1000v-SP_C>sh ip route 10.99.99.0 255.255.255.0                   
    Routing entry for 10.99.99.0/24
        Known via "bgp 65003", distance 20, metric 0
        Tag 65002, type external
        Last update from 172.16.99.4 00:18:45 ago
        Routing Descriptor Blocks:
        * 172.16.99.4, from 172.16.99.4, 00:18:45 ago
            Route metric is 0, traffic share count is 1
            AS Hops 2
            Route tag 65002
            MPLS label: none

    
.. Note::
    As seen above, all traffic from the Jumpbox via SP_C destined to 10.99.99.0/24 is currently via the East DC.  This is because EAST DC wins the tiebreaker as the Origin attribute is IGP versus incomplete for West DC
    
    At this moment, you can only curl to 10.99.99.102 VIP in the EAST DC via the jumpbox.


Place E_A_BIGIP into maintenance mode within the East DC by using BGP AS Path Prepending:
-----------------------------------------------------------------------------------------
		
E_A_BIGIP-13: Create AS-Path-Prepend-OUT route-map on E_A_BIGIP for 10.99.99.0/24 to insert 1 AS Path prepend into the prefix:
		
.. code-block:: none

    [root@E_A_BIGIP-13:Active:Standalone] config # imish -r 0
    E_A_BIGIP-13.local[0]>en
    E_A_BIGIP-13.local[0]#conf t
    Enter configuration commands, one per line.  End with CNTL/Z.
    E_A_BIGIP-13.local[0](config)#
    E_A_BIGIP-13.local[0](config)#ip prefix-list as-path-prepend-prefix seq 10 permit 10.99.99.0/24
    E_A_BIGIP-13.local[0](config)#
    E_A_BIGIP-13.local[0](config)#route-map AS-Path-Prepend-OUT permit 100
    E_A_BIGIP-13.local[0](config-route-map)# match ip address prefix-list as-path-prepend-prefix
    E_A_BIGIP-13.local[0](config-route-map)# set as-path prepend 988
    E_A_BIGIP-13.local[0](config-route-map)#!
    E_A_BIGIP-13.local[0](config-route-map)#route-map AS-Path-Prepend-OUT permit 200
    E_A_BIGIP-13.local[0](config-route-map)#!
    E_A_BIGIP-13.local[0](config-route-map)#router bgp 65202
    E_A_BIGIP-13.local[0](config-router)#nei 10.2.20.4 route-map AS-Path-Prepend-OUT out
    E_A_BIGIP-13.local[0](config-router)#end
    E_A_BIGIP-13.local[0]#
    E_A_BIGIP-13.local[0]#clear ip bgp *
		

E_A_BIGIP-13:  Verify AS-Path-Prepend-OUT has inserted 1 AS Path prepend into the prefix towards CPE @ 10.2.20.4:

.. code-block:: none

    E_A_BIGIP-13.local[0]#sh ip bgp nei 10.2.20.4 advertised-routes | i 10.99.99.0
    *> 10.99.99.0/24    10.2.20.3                         100      32768 988 i
    
		
E_CPE_A_CSR1k:  Verify AS-Path-Prepending inbound on E_CPE_A for 10.99.99.0/24 from E_A_BIGIP:
		
.. code-block:: none

    csr1000v-E_CPE_A>show ip bgp vpnv4 vrf internet neighbors 10.2.20.3 routes
    BGP table version is 56, local router ID is 2.2.2.2
    Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
                    r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
                    x best-external, a additional-path, c RIB-compressed, 
                    t secondary path, 
    Origin codes: i - IGP, e - EGP, ? - incomplete
    RPKI validation codes: V valid, I invalid, N Not found
    
            Network          Next Hop            Metric LocPrf Weight Path
    Route Distinguisher: 65201:1000 (default for vrf internet)
        *>   3.3.3.3/32       10.2.20.3       4294967295             0 65202 65205 i
        *    10.99.99.0/24    10.2.20.3       4294967295             0 65202 988 i

    
E_CPE_A_CSR1k:  Verify that E_A_BIGIP is no longer an installed route or preferred in BGP RIB for 10.99.99.0/24 on E_CPE_A.  You will Note that the next hop for 10.99.99.0/24 is E_B_BIGIP @ 10.2.30.3 and not E_A_BIGIP @ 10.2.20.3.	
	
.. code-block:: none

    csr1000v-E_CPE_A>sh ip route vrf internet
 
    Routing Table: internet
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
    B        1.1.1.1 [20/4294967294] via 172.16.6.3, 1d01h
    B        1.1.1.2 [20/4294967294] via 172.16.6.3, 1d01h
          3.0.0.0/32 is subnetted, 1 subnets
    B        3.3.3.3 [20/4294967294] via 10.2.30.3, 00:05:07
                     [20/4294967294] via 10.2.20.3, 00:05:07
          10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
    C        10.2.20.0/24 is directly connected, GigabitEthernet2
    L        10.2.20.4/32 is directly connected, GigabitEthernet2
    C        10.2.30.0/24 is directly connected, GigabitEthernet3
    L        10.2.30.4/32 is directly connected, GigabitEthernet3
    B        10.99.99.0/24 [20/4294967294] via 10.2.30.3, 00:05:11
          99.0.0.0/24 is subnetted, 1 subnets
    B        99.99.99.0 [20/4294967294] via 172.16.6.3, 1d01h
          172.16.0.0/16 is variably subnetted, 5 subnets, 2 masks
    B        172.16.1.0/24 [20/4294967294] via 172.16.6.3, 1d01h
    B        172.16.2.0/24 [20/4294967294] via 172.16.6.3, 1d01h
    C        172.16.6.0/24 is directly connected, GigabitEthernet5
    L        172.16.6.4/32 is directly connected, GigabitEthernet5
    B        172.16.99.0/24 [20/0] via 172.16.6.3, 1d01h
    csr1000v-E_CPE_A>


.. Note::
    Congratulations! E_A_BIGIP has successfully been placed in maintenance mode within the East DC and is no longer taking any traffic.  This was achieved by inserting the additional AS Path prepend in the previous step eliminating this as a candidate for BGP multipath selection on E_CPE_A.  Let’s continue with additional validation.

E_CPE_A_CSR1k:  Verify that E_CPE_A_CSR1k is now installing specific 10.99.99.0/24 from E_B_BIGIP using specific ip route command.   Notice the next hop of E_B_BIGIP @ 10.2.30.3:

.. code-block:: none

    csr1000v-E_CPE_A>sh ip route vrf internet 10.99.99.0 255.255.255.0
    
    Routing Table: internet
    Routing entry for 10.99.99.0/24
        Known via "bgp 65201", distance 20, metric 4294967294
        Tag 65203, type external
        Last update from 10.2.30.3 00:00:24 ago
        Routing Descriptor Blocks:
        * 10.2.30.3, from 10.2.30.3, 00:00:24 ago
            Route metric is 4294967294, traffic share count is 1
            AS Hops 1
            Route tag 65203
            MPLS label: none
		

E_CPE_A_CSR1k:  Verify that E_CPE_A_CSR1k is now installing specific 10.99.99.0/24 from E_B_BIGIP using specific ip bgp command.   Notice the best path is via E_B_BIGIP @ 10.2.30.3 due to AS Path length:

.. code-block:: none
 
    csr1000v-E_CPE_A>show ip bgp vpnv4 vrf internet 10.99.99.0

    BGP routing table entry for 65201:1000:10.99.99.0/24, version 53
    BGP Bestpath: deterministic-med: aigp-ignore: med
    Paths: (2 available, best #1, table internet)
    Multipath: eiBGP
        Advertised to update-groups:
            3          4         
        Refresh Epoch 1
        65203
        10.2.30.3 (via vrf internet) from 10.2.30.3 (10.2.50.3)
            Origin IGP, metric 4294967295, localpref 100, valid, external, best
            Extended Community: RT:65201:1000
            rx pathid: 0, tx pathid: 0x0
        Refresh Epoch 1
        65202 988
        10.2.20.3 (via vrf internet) from 10.2.20.3 (10.2.40.3)
            Origin IGP, metric 4294967295, localpref 100, valid, external
            Extended Community: RT:65201:1000
            rx pathid: 0, tx pathid: 0

		
csr1000v-SP_C:  Verify path via Virtual Server 10.99.99.102 is still up via East DC @ E_B_BIGIP now that E_A_BIGIP is in maintenance mode within East DC:    

.. code-block:: none

    csr1000v-SP_C>traceroute 10.99.99.102
    Type escape sequence to abort.
    Tracing the route to 10.99.99.102
    VRF info: (vrf in name/id, vrf out name/id)
        1 172.16.99.4 [AS 65001] 7 msec 7 msec 8 msec
        2 172.16.6.4 [AS 65002] 12 msec 14 msec 14 msec
        3 10.99.99.102 [AS 988] 18 msec 12 msec 13 msec
    csr1000v-SP_C>


Reminder:  You can telnet to csr1000v-SP_C from the jumpbox @ 192.168.1.15 with root/default user/pass:
 
.. code-block:: none

    ubuntu@jumphost:~$ telnet 192.168.1.15
    Trying 192.168.1.15...
    Connected to 192.168.1.15.
    Escape character is '^]'.
    
    
    User Access Verification
    
    Username: root
    Password: 
    csr1000v-SP_C>



Jumpbox:  Verify curl to Virtual Server 10.99.99.102 is still up via East DC @ E_B_BIGIP:

.. code-block:: none

    root@jumphost:~# curl 10.99.99.102
    <html><body><h1>It works!</h1>
    <p>This is the default web page for this server.</p>
    <p>The web server software is running but no content has been added, yet.</p>
    </body></html>



Jumpbox:  Verify traceroute to Virtual Server 10.99.99.102 is still up via East DC @ E_B_BIGIP:

.. code-block:: none

    root@jumphost:~# traceroute 10.99.99.102
    traceroute to 10.99.99.102 (10.99.99.102), 30 hops max, 60 byte packets
        1  192.168.1.15 (192.168.1.15)  13.403 ms  13.047 ms  12.418 ms
        2  172.16.99.4 (172.16.99.4)  12.830 ms  12.649 ms  12.351 ms
        3  172.16.6.4 (172.16.6.4)  31.121 ms  44.958 ms  44.866 ms
        4  10.99.99.102 (10.99.99.102)  44.458 ms  45.634 ms  60.454 ms
    root@jumphost:~# 

	
.. Note:: 
    Now that E_A_BIGIP is in maintenance mode we only have E_B_BIGIP taking all the traffic within the East DC for Virtual Servers on 10.99.99.0/24 via SP_C.

    Let's also match AS Path Prepending on E_B_BIGIP such that both East BIG IP's have been added to maintenance mode and are no longer taking any traffic via SP_C.  
				
    This is because the AS Path will be longer via East DC as compared to West DC after we make the next set of changes.


Create AS-Path-Prepend-OUT route-map on E_B_BIGIP for 10.99.99.0/24 to insert 1 AS Path prepend into the prefix:
----------------------------------------------------------------------------------------------------------------
		
E_B_BIGIP-13:  Create AS-Path-Prepend-OUT route-map on E_B_BIGIP for 10.99.99.0/24 to insert 1 AS Path prepend into the prefix:

.. code-block:: none

    [root@E_B_BIGIP-13:Active:Standalone] config # imish -r 0
    E_B_BIGIP-13.local[0]>en
    E_B_BIGIP-13.local[0]#conf t
    Enter configuration commands, one per line.  End with CNTL/Z.
    E_B_BIGIP-13.local[0](config)#ip prefix-list as-path-prepend-prefix seq 10 permit 10.99.99.0/24
    E_B_BIGIP-13.local[0](config)#
    E_B_BIGIP-13.local[0](config)#route-map AS-Path-Prepend-OUT permit 100
    E_B_BIGIP-13.local[0](config-route-map)# match ip address prefix-list as-path-prepend-prefix
    E_B_BIGIP-13.local[0](config-route-map)# set as-path prepend 988
    E_B_BIGIP-13.local[0](config-route-map)#route-map AS-Path-Prepend-OUT permit 200
    E_B_BIGIP-13.local[0](config-route-map)#router bgp 65203
    E_B_BIGIP-13.local[0](config-router)#neighbor 10.2.30.4 route-map AS-Path-Prepend-OUT out
    E_B_BIGIP-13.local[0](config-router)#end
    E_B_BIGIP-13.local[0]#wr
    E_B_BIGIP-13.local[0]#clear ip bgp *


E_B_BIGIP-13:  Verify AS-Path-Prepend-OUT has inserted 1 AS Path prepend into the prefix towards CPE @ 10.2.30.4:		

.. code-block:: none

    E_B_BIGIP-13.local[0]#sh ip bgp nei 10.2.30.4 advertised-routes | i 10.99.99.0/24
    *> 10.99.99.0/24    10.2.30.3                         100      32768 988 i
		

E_CPE_A_CSR1k:  Verify AS-Path-Prepending inbound on E_CPE_A for 10.99.99.0/24 from E_B_BIGIP. 

.. code-block:: none

    csr1000v-E_CPE_A>show ip bgp vpnv4 vrf internet neighbors 10.2.30.3 routes
    BGP table version is 71, local router ID is 2.2.2.2
    Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
                    r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
                    x best-external, a additional-path, c RIB-compressed, 
                    t secondary path, 
    Origin codes: i - IGP, e - EGP, ? - incomplete
    RPKI validation codes: V valid, I invalid, N Not found
    
            Network          Next Hop            Metric LocPrf Weight Path
    Route Distinguisher: 65201:1000 (default for vrf internet)
        *m   3.3.3.3/32       10.2.30.3       4294967295             0 65203 65205 i
        *m   10.99.99.0/24    10.2.30.3       4294967295             0 65203 988 i
    
    Total number of prefixes 2

.. Note::
    You will also notice the ‘*m’ notation has been restored above. This means the prefixes are selected for multipath since we have equalized the previous AS Path prepend configured on E_A_BIGIP.


Let's move along... 

E_CPE_A_CSR1k:  Verify that both E_A_BIGIP & E_B_BIGIP is now valid again for 10.99.99.0/24 on E_CPE_A.  You will Note that the next hop for 10.99.99.0/24 is both E_A_BIGIP @ 10.2.20.3 and E_B_BIGIP @ 10.2.30.3

.. code-block:: none

    csr1000v-E_CPE_A>sh ip route vrf internet
 
    Routing Table: internet
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
    B        1.1.1.1 [20/4294967294] via 172.16.6.3, 1d01h
    B        1.1.1.2 [20/4294967294] via 172.16.6.3, 1d01h
          3.0.0.0/32 is subnetted, 1 subnets
    B        3.3.3.3 [20/4294967294] via 10.2.30.3, 00:08:00
                     [20/4294967294] via 10.2.20.3, 00:08:00
          10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
    C        10.2.20.0/24 is directly connected, GigabitEthernet2
    L        10.2.20.4/32 is directly connected, GigabitEthernet2
    C        10.2.30.0/24 is directly connected, GigabitEthernet3
    L        10.2.30.4/32 is directly connected, GigabitEthernet3
    B        10.99.99.0/24 [20/4294967294] via 10.2.30.3, 00:07:57
                           [20/4294967294] via 10.2.20.3, 00:07:57
          99.0.0.0/24 is subnetted, 1 subnets
    B        99.99.99.0 [20/4294967294] via 172.16.6.3, 1d01h
          172.16.0.0/16 is variably subnetted, 5 subnets, 2 masks
    B        172.16.1.0/24 [20/4294967294] via 172.16.6.3, 1d01h
    B        172.16.2.0/24 [20/4294967294] via 172.16.6.3, 1d01h
    C        172.16.6.0/24 is directly connected, GigabitEthernet5
    L        172.16.6.4/32 is directly connected, GigabitEthernet5
    B        172.16.99.0/24 [20/0] via 172.16.6.3, 1d01h
    csr1000v-E_CPE_A>


.. Note::
    Congratulations! E_B_BIGIP has successfully been placed in maintenance mode within the East DC and is no longer taking any traffic.  This was achieved by inserting the additional AS Path prepend in the previous step eliminating this as a candidate for BGP multipath selection on E_CPE_A. 
 
Now the entire East DC is in maintenance mode as both E_A_BIGIP and E_B_BIGIP are no longer taking traffic. 
 
That is, 10.99.99.0/24 is preferred via the West DC from the jumpbox when leveraging SP_C. Let’s continue with additional validation.
 
    
E_CPE_A_CSR1k:  We can observe that prepending is happening for 10.99.99.0/24 on E_CPE_A for both E_A_BIGIP & E_B_BIGIP:
		
.. code-block:: none

    csr1000v-E_CPE_A>show ip bgp vpnv4 vrf internet | i 988
    *m   10.99.99.0/24    10.2.30.3       4294967295             0 65203 988 i
    *>                    10.2.20.3       4294967295             0 65202 988 i
	
		
csr1000v-SP_C:  Verify 10.99.99.0/24 is available on SP_C BGP RIB table via East DC.  You will notice the best path is via West DC via AS 65101.

Reminder:  You can telnet to csr1000v-SP_C from the jumpbox @ 192.168.1.15 with root/default user/pass:
		
.. code-block:: none

    csr1000v-SP_C>sh ip bgp 10.99.99.100/24
    BGP routing table entry for 10.99.99.0/24, version 25
    Paths: (2 available, best #2, table default)
        Advertised to update-groups:
            1         
        Refresh Epoch 1
        65002 988 988
        172.16.99.4 from 172.16.99.4 (172.1.1.2)
            Origin IGP, localpref 100, valid, external
            rx pathid: 0, tx pathid: 0
        Refresh Epoch 1
        65001 65101, (aggregated by 65101 192.168.255.10)
        172.16.99.3 from 172.16.99.3 (172.1.1.1)
            Origin incomplete, localpref 100, valid, external, atomic-aggregate, best
            rx pathid: 0, tx pathid: 0x0


csr1000v-SP_C:  Verify 10.99.99.0/24 is no longer installed on SP_C IP routing table via East DC.  You will notice the route installed is via West DC via AS 65001.

.. code-block:: none

    csr1000v-SP_C>sh ip route 10.99.99.100
    Routing entry for 10.99.99.0/24
        Known via "bgp 65003", distance 20, metric 0
        Tag 65001, type external
        Last update from 172.16.99.3 00:07:29 ago
        Routing Descriptor Blocks:
        * 172.16.99.3, from 172.16.99.3, 00:07:29 ago
            Route metric is 0, traffic share count is 1
            AS Hops 2
            Route tag 65001
            MPLS label: none

		
.. Note::  
    This prefix is no longer installed in the routing table via East DC because the AS Path length is larger than that of West DC. At this point traffic is now via West DC for 10.99.99.0/24 from SP_C point-of-view.
 

csr1000v-SP_C:  Verify path via Virtual Server 10.99.99.101 is now via West DC - AS 65101.  Note that below output may not be an exact match as this can be via either 172.16.1.4 or 172.16.2.4 leveraging AS 65101 via West_CPE_A or West_CPE_B.

.. code-block:: none

    csr1000v-SP_C>traceroute 10.99.99.101
    Type escape sequence to abort.
    Tracing the route to 10.99.99.101
    VRF info: (vrf in name/id, vrf out name/id)
        1 172.16.99.3 [AS 65001] 8 msec 8 msec 7 msec
        2 172.16.1.4 [AS 65001] 11 msec 8 msec 10 msec
        3 10.99.99.101 [AS 65101] 14 msec 13 msec 15 msec
    csr1000v-SP_C>


Jumpbox:  Verify curl to Virtual Server 10.99.99.101 is up via West DC:

.. code-block:: none

    root@jumphost:~# curl 10.99.99.101
    <html><body><h1>It works!</h1>
    <p>This is the default web page for this server.</p>
    <p>The web server software is running but no content has been added, yet.</p>
    </body></html>
    root@jumphost:~# 


Jumpbox:  Verify traceroute to Virtual Server 10.99.99.101 is West DC.  Note that below output may not be an exact match as this can be via either 172.16.1.4 or 172.16.2.4 leveraging AS 65101 via West_CPE_A or West_CPE_B.		
  
.. code-block:: none

    root@jumphost:~# traceroute 10.99.99.101
    traceroute to 10.99.99.101 (10.99.99.101), 30 hops max, 60 byte packets
        1  192.168.1.15 (192.168.1.15)  2.504 ms  15.470 ms  15.277 ms
        2  172.16.99.3 (172.16.99.3)  18.709 ms  19.369 ms  19.762 ms
        3  172.16.2.4 (172.16.2.4)  25.569 ms  28.738 ms  44.922 ms
        4  10.99.99.101 (10.99.99.101)  44.591 ms  47.980 ms  51.598 ms
    root@jumphost:~# 

		
Anycast DC Failover section - Swing Traffic back to East DC by adding 2 x /25 specific routes which comprise of the overall 10.99.99.0 /24
------------------------------------------------------------------------------------------------------------------------------------------
		
.. Note:: 
    In previous section we verified that 10.99.99.0/24 is only installed in the IP Routing table of SP_C via West DC. However, East DC is available as backup path in BGP RIB.

In this section we will swing traffic back to East DC by utilizing 2 x /25's.
		
E_A_BIGIP-13:  Configure BGP on E_A_BIGIP-13 to originate 10.99.99.0 /25 and 10.99.99.128 /25:
		
.. code-block:: none

    [root@E_A_BIGIP-13:Active:Standalone] config # imish -r 0
    E_A_BIGIP-13.local[0]>en
    E_A_BIGIP-13.local[0]#conf t
    Enter configuration commands, one per line.  End with CNTL/Z.
    E_A_BIGIP-13.local[0](config)#router bgp 65202
    E_A_BIGIP-13.local[0](config-router)#network 10.99.99.0/25
    E_A_BIGIP-13.local[0](config-router)#network 10.99.99.128/25
    E_A_BIGIP-13.local[0](config)#end
    E_A_BIGIP-13.local[0]#clear ip bgp *
    E_A_BIGIP-13.local[0]#wr
    Building configuration...

		
E_A_BIGIP-13: Verify 10.99.99.0/24, 10.99.99.0/25, and 10.99.99.128/25 are advertised via E_A_BIGIP to E_CPE_A @ 10.2.20.4:
	
.. code-block:: none

    E_A_BIGIP-13.local[0]#sh ip bgp nei 10.2.20.4 ad
    BGP table version is 2, local router ID is 10.2.40.3
    Status codes: s suppressed, d damped, h history, * valid, > best, i - internal
    Origin codes: i - IGP, e - EGP, ? - incomplete
    
        Network          Next Hop            Metric     LocPrf     Weight Path
    *> 3.3.3.3/32       10.2.20.3                                      0 65205 i
    *> 10.3.99.0/24     10.2.20.3                                      0 65205 i
    *> 10.99.99.0/24    10.2.20.3                         100      32768 988 i
    *> 10.99.99.0/25    10.2.20.3                         100      32768 i
    *> 10.99.99.128/25  10.2.20.3                         100      32768 i
    
    Total number of prefixes 5


csr1000v-SP_C:  Verify 10.99.99.0/25 is available on SP_C BGP RIB table via East DC leveraging 10.99.99.0 /25.  You will notice the best path is via East DC via AS 65002 988.

Reminder:  You can telnet to csr1000v-SP_C from the jumpbox @ 192.168.1.15 with root/default user/pass.

.. code-block:: none

    csr1000v-SP_C>sh ip bgp 10.99.99.102
        BGP routing table entry for 10.99.99.0/25, version 30
        Paths: (2 available, best #2, table default)
            Advertised to update-groups:
                1         
            Refresh Epoch 1
            65001 65002 988
                172.16.99.3 from 172.16.99.3 (172.1.1.1)
                Origin IGP, localpref 100, valid, external
                rx pathid: 0, tx pathid: 0
            Refresh Epoch 1
            65002 988
                172.16.99.4 from 172.16.99.4 (172.1.1.2)
                Origin IGP, localpref 100, valid, external, best
                rx pathid: 0, tx pathid: 0x0



csr1000v-SP_C:  Let's also verify 10.99.99.0/25

.. code-block:: none

    csr1000v-SP_C>sh ip bgp 10.99.99.0/25
        BGP routing table entry for 10.99.99.0/25, version 26
        Paths: (2 available, best #2, table default)
            Advertised to update-groups:
                1         
            Refresh Epoch 1
            65001 65002 988
            172.16.99.3 from 172.16.99.3 (172.1.1.1)
                Origin IGP, localpref 100, valid, external
                rx pathid: 0, tx pathid: 0
            Refresh Epoch 1
            65002 988
                172.16.99.4 from 172.16.99.4 (172.1.1.2)
                Origin IGP, localpref 100, valid, external, best
                rx pathid: 0, tx pathid: 0x0


csr1000v-SP_C:  Verify 10.99.99.102 is installed on SP_C IP routing table via East DC leveraging 10.99.99.0/25.  You will notice the route installed is via East DC via AS 65002.

.. code-block:: none

    csr1000v-SP_C>sh ip route 10.99.99.102
    Routing entry for 10.99.99.0/25
        Known via "bgp 65003", distance 20, metric 0
        Tag 65002, type external
        Last update from 172.16.99.4 00:03:10 ago
        Routing Descriptor Blocks:
        * 172.16.99.4, from 172.16.99.4, 00:03:10 ago
            Route metric is 0, traffic share count is 1
            AS Hops 2
            Route tag 65002
            MPLS label: none

.. Note::
    You will observe that the IP Routing table of SP_C will prefer the path via East DC for the 10.99.99.102 Virtual Server due longest match of 10.99.99.0/25 even though 10.99.99.0/24 is via West DC.
 
We can expect the same behavior with 10.99.99.128 /25.  Let’s validate.

csr1000v-SP_C:  Verify 10.99.99.128 /25 is available on SP_C IP BGP RIB table via East DC.  You will notice the best path is via East DC via AS 65002 988.

.. code-block:: none

    csr1000v-SP_C>sh ip bgp 10.99.99.128/25
    BGP routing table entry for 10.99.99.128/25, version 39
    Paths: (2 available, best #2, table default)
        Advertised to update-groups:
            1         
        Refresh Epoch 1
        65001 65002 988
        172.16.99.3 from 172.16.99.3 (172.1.1.1)
            Origin IGP, localpref 100, valid, external
            rx pathid: 0, tx pathid: 0
        Refresh Epoch 1
        65002 988
        172.16.99.4 from 172.16.99.4 (172.1.1.2)
            Origin IGP, localpref 100, valid, external, best
            rx pathid: 0, tx pathid: 0x0


csr1000v-SP_C:  Verify 10.99.99.128 /25 is installed on SP_C IP routing table via East DC leveraging 10.99.99.128 /25.  You will notice the route installed is via East DC via AS 65002.   
 
.. code-block:: none

    csr1000v-SP_C>sh ip route 10.99.99.128 255.255.255.128
    Routing entry for 10.99.99.128/25
        Known via "bgp 65003", distance 20, metric 0
        Tag 65002, type external
        Last update from 172.16.99.4 00:03:16 ago
        Routing Descriptor Blocks:
        * 172.16.99.4, from 172.16.99.4, 00:03:16 ago
            Route metric is 0, traffic share count is 1
            AS Hops 2
            Route tag 65002
            MPLS label: none


.. Note::
    What observations are made with 10.99.99.0/24? You will notice this remains the same with West DC preferred via AS Path length for 10.99.99.0/24.


csr1000v-SP_C:  Verify 10.99.99.0 /24 is available on SP_C BGP RIB table via West DC.  You will notice the best path is via West DC via AS 65101.

.. code-block:: none

    csr1000v-SP_C>sh ip bgp 10.99.99.0/24
    BGP routing table entry for 10.99.99.0/24, version 25
    Paths: (2 available, best #2, table default)
        Advertised to update-groups:
            1         
        Refresh Epoch 1
        65002 988 988
        172.16.99.4 from 172.16.99.4 (172.1.1.2)
            Origin IGP, localpref 100, valid, external
            rx pathid: 0, tx pathid: 0
        Refresh Epoch 1
        65001 65101, (aggregated by 65101 192.168.255.10)
        172.16.99.3 from 172.16.99.3 (172.1.1.1)
            Origin incomplete, localpref 100, valid, external, atomic-aggregate, best
            rx pathid: 0, tx pathid: 0x0

		

csr1000v-SP_C:  Verify 10.99.99.0 /24 is available on SP_C IP routing table via West DC.  You will notice the best path is via West DC via AS 65001.		    
		
.. code-block:: none

    csr1000v-SP_C>sh ip route 10.99.99.0 255.255.255.0    
    Routing entry for 10.99.99.0/24
        Known via "bgp 65003", distance 20, metric 0
        Tag 65001, type external
        Last update from 172.16.99.3 01:00:50 ago
        Routing Descriptor Blocks:
        * 172.16.99.3, from 172.16.99.3, 01:00:50 ago
            Route metric is 0, traffic share count is 1
            AS Hops 2
            Route tag 65001
            MPLS label: none


		
csr1000v-SP_C:  Verify path via Virtual Server 10.99.99.102 is now via East DC due to the /25's being originated from East DC via E_A_BIGIP-13 .

.. code-block:: none

    csr1000v-SP_C>traceroute 10.99.99.102
    Type escape sequence to abort.
    Tracing the route to 10.99.99.102
    VRF info: (vrf in name/id, vrf out name/id)
        1 172.16.99.4 [AS 65001] 7 msec 6 msec 8 msec
        2 172.16.6.4 [AS 65002] 9 msec 12 msec 10 msec
        3 10.99.99.102 [AS 988] 17 msec 21 msec 16 msec
    csr1000v-SP_C>


Jumpbox:  Verify curl to Virtual Server 10.99.99.102 is up via East DC:

.. code-block:: none

    root@jumphost:~# curl 10.99.99.102
    <html><body><h1>It works!</h1>
    <p>This is the default web page for this server.</p>
    <p>The web server software is running but no content has been added, yet.</p>
    </body></html>
    root@jumphost:~# 


Jumpbox:  Verify traceroute to Virtual Server 10.99.99.102 is East DC. 

.. code-block:: none

    root@jumphost:~# traceroute 10.99.99.102
    traceroute to 10.99.99.102 (10.99.99.102), 30 hops max, 60 byte packets
        1  192.168.1.15 (192.168.1.15)  14.200 ms  14.073 ms  13.654 ms
        2  172.16.99.4 (172.16.99.4)  21.305 ms  27.179 ms  27.071 ms
        3  172.16.6.4 (172.16.6.4)  26.755 ms  26.447 ms  26.119 ms
        4  10.99.99.102 (10.99.99.102)  36.549 ms  48.875 ms  48.795 ms
    root@jumphost:~# 

    
.. Note::
    Congratulations!  You have successfully swing traffic back to the East DC even though 10.99.99.0/24 is currently preferred via West DC from SP_C.  This was accomplished by introducing specific /25’s from the East DC via E_A_BIGIP-13. 

        Let’s finish this off as you are in the home stretch!  We will finish the East DC by originating the same /25’s on E_B_BIGIP-13 for consistency with E_A_BIGIP-13.


Re-introduce E_B_BIGIP-13 in the East DC via the /25's:
-------------------------------------------------------
		
In this section we will finish the configuration on the East DC to originate the 2 x /25's on E_B_BIGIP-13.  This will match the origination of 2 x 25’s previously completed on E_A_BIGIP-13.        

E_B_BIGIP-13:  Configure BGP on E_B_BIGIP-13 to originate 10.99.99.0 /25 and 10.99.99.128 /25:

.. code-block:: none

    [root@E_B_BIGIP-13:Active:Standalone] config # imish -r 0
    E_B_BIGIP-13.local[0]>enE_B_BIGIP-13.local[0]#conf t
    Enter configuration commands, one per line.  End with CNTL/Z.
    E_B_BIGIP-13.local[0](config)#router bgp 65203
    E_B_BIGIP-13.local[0](config-router)#network 10.99.99.0/25
    E_B_BIGIP-13.local[0](config-router)#network 10.99.99.128/25
    E_B_BIGIP-13.local[0](config)#end
    E_B_BIGIP-13.local[0]#clear ip bgp *
    E_B_BIGIP-13.local[0]#wr
    Building configuration...


E_B_BIGIP-13:  Verify 10.99.99.0/24, 10.99.99.0/25, and 10.99.99.128/25 are advertised via E_B_BIGIP to E_CPE_A @ 10.2.30.4:

.. code-block:: none

    E_B_BIGIP-13.local[0]#sh ip bgp nei 10.2.30.4 advertised-routes 
    BGP table version is 9, local router ID is 10.2.50.3
    Status codes: s suppressed, d damped, h history, * valid, > best, i - internal
    Origin codes: i - IGP, e - EGP, ? - incomplete
    
        Network          Next Hop            Metric     LocPrf     Weight Path
    *> 3.3.3.3/32       10.2.30.3                                      0 65205 i
    *> 10.3.99.0/24     10.2.30.3                                      0 65205 i
    *> 10.99.99.0/24    10.2.30.3                         100      32768 988 i
    *> 10.99.99.0/25    10.2.30.3                         100      32768 i
    *> 10.99.99.128/25  10.2.30.3                         100      32768 i



E_CPE_A_CSR1k:  Verify that 10.99.99.0/24, 10.99.99.0/24, and 10.99.99.128/25 are learned via the routing table on E_CPE_A from  both E_A_BIGIP @ 10.2.20.3 and E_B_BIGIP @ 10.2.30.3

.. code-block:: none

    csr1000v-E_CPE_A>sh ip route vrf internet

    
    Routing Table: internet
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
    B        1.1.1.1 [20/4294967294] via 172.16.6.3, 1d02h
    B        1.1.1.2 [20/4294967294] via 172.16.6.3, 1d02h
        3.0.0.0/32 is subnetted, 1 subnets
    B        3.3.3.3 [20/4294967294] via 10.2.30.3, 00:04:59
                    [20/4294967294] via 10.2.20.3, 00:04:59
        10.0.0.0/8 is variably subnetted, 7 subnets, 3 masks
    C        10.2.20.0/24 is directly connected, GigabitEthernet2
    L        10.2.20.4/32 is directly connected, GigabitEthernet2
    C        10.2.30.0/24 is directly connected, GigabitEthernet3
    L        10.2.30.4/32 is directly connected, GigabitEthernet3
    B        10.99.99.0/24 [20/4294967294] via 10.2.30.3, 00:05:20
                        [20/4294967294] via 10.2.20.3, 00:05:20
    B        10.99.99.0/25 [20/4294967294] via 10.2.30.3, 00:05:28
                        [20/4294967294] via 10.2.20.3, 00:05:28
    B        10.99.99.128/25 [20/4294967294] via 10.2.30.3, 00:05:28
                            [20/4294967294] via 10.2.20.3, 00:05:28
        99.0.0.0/24 is subnetted, 1 subnets
    B        99.99.99.0 [20/4294967294] via 172.16.6.3, 1d03h
        172.16.0.0/16 is variably subnetted, 5 subnets, 2 masks
    B        172.16.1.0/24 [20/4294967294] via 172.16.6.3, 1d02h
    B        172.16.2.0/24 [20/4294967294] via 172.16.6.3, 1d02h
    C        172.16.6.0/24 is directly connected, GigabitEthernet5
    L        172.16.6.4/32 is directly connected, GigabitEthernet5
    B        172.16.99.0/24 [20/0] via 172.16.6.3, 1d03h
    csr1000v-E_CPE_A> 


E_CPE_A_CSR1k:  As an example, let’s take a closer look the bgp table for 10.99.99.128/25 on E_CPE_A:

.. code-block:: none

    csr1000v-E_CPE_A>sh ip bgp vpnv4 vrf internet 10.99.99.128/25
    BGP routing table entry for 65201:1000:10.99.99.128/25, version 98
    BGP Bestpath: deterministic-med: aigp-ignore: med
    Paths: (2 available, best #1, table internet)
    Multipath: eiBGP
        Advertised to update-groups:
            3          4         
        Refresh Epoch 1
        65202
        10.2.20.3 (via vrf internet) from 10.2.20.3 (10.2.40.3)
            Origin IGP, metric 4294967295, localpref 100, valid, external, multipath, best
            Extended Community: RT:65201:1000
            rx pathid: 0, tx pathid: 0x0
        Refresh Epoch 1
        65203
        10.2.30.3 (via vrf internet) from 10.2.30.3 (10.2.50.3)
            Origin IGP, metric 4294967295, localpref 100, valid, external, multipath(oldest)
            Extended Community: RT:65201:1000
            rx pathid: 0, tx pathid: 0

		
csr1000v-SP_C:  Verify nothing changed w.r.t. 10.99.99.0/25 and 10.99.99.128/25 and are still in the IP Routing table of SP_C via East DC after adding the /25's on E_B_BIGIP-13.

First let’s take a look at the BGP table and confirm nothing changed since we previously added the 2 x /25’s on E_A_BIGIP-13 and now completed E_B_BIGIP-13.

First will be 10.99.99.0 /25:

.. code-block:: none

    csr1000v-SP_C>sh ip bgp 10.99.99.0 255.255.255.128
    BGP routing table entry for 10.99.99.0/25, version 38
    Paths: (2 available, best #2, table default)
        Advertised to update-groups:
            1         
        Refresh Epoch 1
        65001 65002 988
        172.16.99.3 from 172.16.99.3 (172.1.1.1)
            Origin IGP, localpref 100, valid, external
            rx pathid: 0, tx pathid: 0
        Refresh Epoch 1
        65002 988
        172.16.99.4 from 172.16.99.4 (172.1.1.2)
            Origin IGP, localpref 100, valid, external, best
            rx pathid: 0, tx pathid: 0x0


Second will be 10.99.99.128 /25:

.. code-block:: none

    csr1000v-SP_C>sh ip bgp 10.99.99.128 255.255.255.128
    BGP routing table entry for 10.99.99.128/25, version 39
    Paths: (2 available, best #2, table default)
        Advertised to update-groups:
            1         
        Refresh Epoch 1
        65001 65002 988
        172.16.99.3 from 172.16.99.3 (172.1.1.1)
            Origin IGP, localpref 100, valid, external
            rx pathid: 0, tx pathid: 0
        Refresh Epoch 1
        65002 988
        172.16.99.4 from 172.16.99.4 (172.1.1.2)
            Origin IGP, localpref 100, valid, external, best
            rx pathid: 0, tx pathid: 0x0


Last will be 10.99.99.0 /24:

.. code-block:: none

    csr1000v-SP_C>sh ip bgp 10.99.99.0 255.255.255.0    
    BGP routing table entry for 10.99.99.0/24, version 25
    Paths: (2 available, best #2, table default)
        Advertised to update-groups:
            1         
        Refresh Epoch 1
        65002 988 988
        172.16.99.4 from 172.16.99.4 (172.1.1.2)
            Origin IGP, localpref 100, valid, external
            rx pathid: 0, tx pathid: 0
        Refresh Epoch 1
        65001 65101, (aggregated by 65101 192.168.255.10)
        172.16.99.3 from 172.16.99.3 (172.1.1.1)
            Origin incomplete, localpref 100, valid, external, atomic-aggregate, best
            rx pathid: 0, tx pathid: 0x0



.. Note::
    You will also observe that the IP Routing table on SP_C will still prefer West DC for the 10.99.99.0/24 due to previous AS Path Prepending exercise inserted from the East DC towards SP.

csr1000v-SP_C:  Now let’s take a look at the ip routing table and confirm nothing has changed since we previously added the 2 x /25’s on E_A_BIGIP-13 and now completed E_B_BIGIP-13.		

First will be 10.99.99.0 /25

.. code-block:: none

    csr1000v-SP_C>sh ip route 10.99.99.0 255.255.255.128  
    Routing entry for 10.99.99.0/25
        Known via "bgp 65003", distance 20, metric 0
        Tag 65002, type external
        Last update from 172.16.99.4 00:19:27 ago
        Routing Descriptor Blocks:
        * 172.16.99.4, from 172.16.99.4, 00:19:27 ago
            Route metric is 0, traffic share count is 1
            AS Hops 2
            Route tag 65002
            MPLS label: none


Second will be 10.99.99.128 /25

.. code-block:: none

    csr1000v-SP_C>sh ip route 10.99.99.128 255.255.255.128
    Routing entry for 10.99.99.128/25
        Known via "bgp 65003", distance 20, metric 0
        Tag 65002, type external
        Last update from 172.16.99.4 00:19:34 ago
        Routing Descriptor Blocks:
        * 172.16.99.4, from 172.16.99.4, 00:19:34 ago
            Route metric is 0, traffic share count is 1
            AS Hops 2
            Route tag 65002
            MPLS label: none


Last will be 10.99.99.0 /24

.. code-block:: none

    csr1000v-SP_C>sh ip route 10.99.99.0 255.255.255.0    
    Routing entry for 10.99.99.0/24
        Known via "bgp 65003", distance 20, metric 0
        Tag 65001, type external
        Last update from 172.16.99.3 01:00:50 ago
        Routing Descriptor Blocks:
        * 172.16.99.3, from 172.16.99.3, 01:00:50 ago
            Route metric is 0, traffic share count is 1
            AS Hops 2
            Route tag 65001
            MPLS label: none
		

csr1000v-SP_C:  Verify path via Virtual Server 10.99.99.102 is still via East DC with introduction of adding the /25's from East DC via both E_A_BIGIP-13 & E_B_BIGIP-13 .
 	
.. code-block:: none

    csr1000v-SP_C>traceroute 10.99.99.102
    Type escape sequence to abort.
    Tracing the route to 10.99.99.102
    VRF info: (vrf in name/id, vrf out name/id)
        1 172.16.99.4 [AS 65001] 7 msec 7 msec 8 msec
        2 172.16.6.4 [AS 65002] 23 msec 11 msec 11 msec
        3 10.99.99.102 [AS 988] 14 msec 16 msec 14 msec
    csr1000v-SP_C>


Jumpbox:  Verify curl to Virtual Server 10.99.99.102 is up via East DC:

.. code-block:: none

	root@jumphost:~# curl 10.99.99.102
	<html><body><h1>It works!</h1>
	<p>This is the default web page for this server.</p>
	<p>The web server software is running but no content has been added, yet.</p>
	</body></html>
	

Jumpbox:  Verify traceroute to Virtual Server 10.99.99.102 is East DC. 

.. code-block:: none

    root@jumphost:~# traceroute 10.99.99.102
    traceroute to 10.99.99.102 (10.99.99.102), 30 hops max, 60 byte packets
        1  192.168.1.15 (192.168.1.15)  23.599 ms  21.587 ms  20.725 ms
        2  172.16.99.4 (172.16.99.4)  25.015 ms  24.031 ms  23.148 ms
        3  172.16.6.4 (172.16.6.4)  34.033 ms  33.082 ms  38.138 ms
        4  10.99.99.102 (10.99.99.102)  37.173 ms  36.389 ms  53.688 ms


.. Note::
    Congratulations!  This section is now complete and everything checks out as expected.  We are now going to move on to the final step and validate with a Virtual Server on the upper /25 – 10.99.99.128 /25.
 
		
Create an application configuration for a virtual server and a pool member on E_A_BIGIP-13 and E_A_BIGIP-13 to validate reachability via 10.99.99.128/25:
----------------------------------------------------------------------------------------------------------------------------------------------------------------
			
Create the following virtual server and pool member on both E_A_BIGIP-13 and E_B_BIGIP-13
			
.. code-block:: none

    tmsh create ltm virtual vip3 destination 10.99.99.129:80 source-address-translation { type automap } pool pool1 profiles add { tcp http }
    
    tmsh save sys config partitions all


E_A_BIGIP-13:  Your virtual server should now show available on E_A_BIGIP-13

.. code-block:: none

    root@E_A_BIGIP-13:Active:Standalone] config # tmsh show ltm virtual vip3

    ------------------------------------------------------------------
    Ltm::Virtual Server: vip3      
    ------------------------------------------------------------------
    Status                         
      Availability     : available 
      State            : enabled   
      Reason           : The virtual server is available
      CMP              : enabled   
      CMP Mode         : all-cpus  
      Destination      : 10.99.99.129:80


E_B_BIGIP-13:  Your virtual server should now show available on E_B_BIGIP-13

.. code-block:: none

    root@E_B_BIGIP-13:Active:Standalone] config # tmsh show ltm virtual vip3

    ------------------------------------------------------------------
    Ltm::Virtual Server: vip3      
    ------------------------------------------------------------------
    Status                         
      Availability     : available 
      State            : enabled   
      Reason           : The virtual server is available
      CMP              : enabled   
      CMP Mode         : all-cpus  
      Destination      : 10.99.99.129:80		


Verify path via Virtual Server 10.99.99.129 which falls on 10.99.99.128/25 is via East DC with introduction of adding the /25's from East DC.

csr1000v-SP_C:  Verify path via Virtual Server 10.99.99.129 which falls on 10.99.99.128/25 is via East DC with introduction of adding the /25's from East DC.  You can validate via traceroute to 10.99.99.129 on SP_C:

.. code-block:: none

    csr1000v-SP_C>traceroute 10.99.99.129
    Type escape sequence to abort.
    Tracing the route to 10.99.99.129
    VRF info: (vrf in name/id, vrf out name/id)
        1 172.16.99.4 [AS 65001] 4 msec 6 msec 5 msec
        2 172.16.6.4 [AS 65002] 7 msec 8 msec 9 msec
        3 10.99.99.129 [AS 988] 7 msec 8 msec 7 msec
    csr1000v-SP_C>
		

csr1000v-SP_C:  You can validate via show ip route for 10.99.99.129 on SP_C:

.. code-block:: none

  csr1000v-SP_C>sh ip route 10.99.99.129
    Routing entry for 10.99.99.128/25
        Known via "bgp 65003", distance 20, metric 0
        Tag 65002, type external
        Last update from 172.16.99.4 00:24:58 ago
        Routing Descriptor Blocks:
        * 172.16.99.4, from 172.16.99.4, 00:24:58 ago
            Route metric is 0, traffic share count is 1
            AS Hops 2
            Route tag 65002
            MPLS label: none
    csr1000v-SP_C>
		

Jumpbox:  Verify curl to Virtual Server 10.99.99.129 is up via East DC:

.. code-block:: none

    root@jumphost:~# curl 10.99.99.129
    <html><body><h1>It works!</h1>
    <p>This is the default web page for this server.</p>
    <p>The web server software is running but no content has been added, yet.</p>
    </body></html>
    root@jumphost:~# 


Jumpbox:  Verify traceroute to Virtual Server 10.99.99.129 is East DC. 

.. code-block:: none

    root@jumphost:~# traceroute 10.99.99.129
    traceroute to 10.99.99.129 (10.99.99.129), 30 hops max, 60 byte packets
        1  192.168.1.15 (192.168.1.15)  2.569 ms  9.153 ms  9.097 ms
        2  172.16.99.4 (172.16.99.4)  23.348 ms  22.639 ms  22.585 ms
        3  172.16.6.4 (172.16.6.4)  25.018 ms  24.391 ms  23.766 ms
        4  10.99.99.129 (10.99.99.129)  30.824 ms  30.220 ms  39.918 ms
    root@jumphost:~# 
		
	
.. Note::
    Congratulations!  You have successfully completed lab 2.
