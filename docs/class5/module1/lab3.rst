Module 3:  Troubleshooting
==========================
		
Location and content of dynamic routing log files:
		
For each dynamic routing protocol, the BIG-IP system logs messages to a file that pertains to the route domain in which the protocol is running. An example of the path name to a dynamic routing log file is /var/log/zebos/rd1/zebos.log file, where rd1 is the route domain of the protocol instance.
		
On E_A_BIGIP configure the following logging statement for BGP:
		
.. code-block:: none

    [root@E_A_BIGIP-13:Active:Standalone] log # imish
    E_A_BIGIP-13.local[0]>enable
    E_A_BIGIP-13.local[0]#conf t
    Enter configuration commands, one per line.  End with CNTL/Z.
    E_A_BIGIP-13.local[0](config)#log file /var/log/zebos/rd0/zebos.log
    E_A_BIGIP-13.local[0]#wr
    Building configuration...
    [OK]
    E_A_BIGIP-13.local[0]#
    E_A_BIGIP-13.local[0]#
    E_A_BIGIP-13.local[0]#
    E_A_BIGIP-13.local[0]#exit
    
    On E_A_BIGIP configure BGP debugging:
		
.. code-block:: none

		E_A_BIGIP-13.local[0]#deb bgp all
		
.. code-block:: none

    E_A_BIGIP-13.local[0]#sh debug bgp
    BGP debugging status:
        BGP debugging is on
        BGP nht debugging is on
        BGP nsm debugging is on
        BGP events debugging is on
        BGP keepalives debugging is on
        BGP updates debugging is on
        BGP fsm debugging is on
        BGP filter debugging is on
        BGP Route Flap Dampening debugging is on
        BGP Bidirectional Forwarding Detection is on

.. code-block:: none

    E_A_BIGIP-13.local[0]#wr
    Building configuration...
    [OK]
    
    On E_A_BIGIP view BGP log messages:
		
.. code-block:: none

    root@(E_A_BIGIP-13)(cfg-sync Standalone)(Active)(/Common)(tmos)# quit
    [root@E_A_BIGIP-13:Active:Standalone] config # cd /var/log
    [root@E_A_BIGIP-13:Active:Standalone] log # ls | grep zebos
    zebos
    [root@E_A_BIGIP-13:Active:Standalone] log # cd zebos
    [root@E_A_BIGIP-13:Active:Standalone] zebos # ls
    rd0
    [root@E_A_BIGIP-13:Active:Standalone] zebos # cd rd0
    [root@E_A_BIGIP-13:Active:Standalone] rd0 # ls
    zebos.log
    [root@E_A_BIGIP-13:Active:Standalone] rd0 # more zebos.log
    2018/07/09 11:49:14 informational: BGP : [NETWORK] Accept Thread: Incoming conn from host 10.2.20.4 (FD=11)
    2018/07/09 11:49:14 informational: BGP : [NETWORK] Accept Thread: 10.2.20.4 - No such Peer configured
    2018/07/09 11:49:16 informational: BGP : [NETWORK] Accept Thread: Incoming conn from host 10.2.40.4 (FD=11)
    2018/07/09 11:49:16 informational: BGP : [NETWORK] Accept Thread: 10.2.40.4 - No such Peer configured
    2018/07/09 11:49:21 informational: BGP : [NETWORK] Accept Thread: Incoming conn from host 10.2.20.4 (FD=11)
    2018/07/09 11:49:21 informational: BGP : [NETWORK] Accept Thread: 10.2.20.4 - No such Peer configured
    2018/07/09 11:49:26 informational: BGP : [NETWORK] Accept Thread: Incoming conn from host 10.2.40.4 (FD=11)
    2018/07/09 11:49:26 informational: BGP : [NETWORK] Accept Thread: 10.2.40.4 - No such Peer configured
    2018/07/09 11:49:28 informational: BGP : [RIB] Scanning BGP Network Routes...
    2018/07/09 11:49:30 informational: BGP : [NETWORK] Accept Thread: Incoming conn from host 10.2.20.4 (FD=11)
    2018/07/09 11:49:30 informational: BGP : [NETWORK] Accept Thread: 10.2.20.4 - No such Peer configured
    2018/07/09 11:49:33 informational: BGP : [INIT] peer: 10.2.20.4 GR enabled for IPv4 (1).
    2018/07/09 11:49:33 informational: BGP : 10.2.20.4-Outgoing [FSM] State: Idle Event: 35 op_state: MASTER
    2018/07/09 11:49:33 informational: BGP : 10.2.20.4-Outgoing [FSM] State: Idle Event: 1 op_state: MASTER
    2018/07/09 11:49:36 informational: BGP : [NETWORK] Accept Thread: Incoming conn from host 10.2.40.4 (FD=11)
    2018/07/09 11:49:36 informational: BGP : [NETWORK] Accept Thread: 10.2.40.4 - No such Peer configured
		
On E_A_BIGIP disable BGP debugging:
		
.. code-block:: none

    E_A_BIGIP-13.local[0]#undebug bgp all

    
    E_A_BIGIP-13.local[0]#wr
    Building configuration...
    [OK]

.. Note:: This completes Lab 3