Lab 4:  Test & Validate a Failover Scenario
-------------------------------------------

In Lab 4, we will manipulate BIG-IP interfaces to see how it affects basic failover events.

Lab Tasks:
==========

* Task 1: Disable BIG-IP Interfaces
* Task 2: Enable both Interfaces on ACTIVE BIG-IP

Task 1: Disable BIG-IP Interfaces
==================================

In Task 1, we will simulate a "link down" failure on the ACTIVE
BIG-IP, and determine if the BIG-IP will failover automatically.

We will compare the failover timing once we create & use our HA Group
configuration design.

BONUS: From the BIG-IP CLI, perform a "follow" of the */var/log/ltm* log
to see logging data in real-time:

-  Access the "WEB SHELL" from UDF for each BIG-IP:

   .. image:: ../images/image56.png
      

-  Run the following command at shell prompt:

   .. code-block:: bash
   
     tail -f /var/log/ltm

   BIG-IP-A:

   .. image:: ../images/image57.png
   
   BIG-IP-B:

   .. image:: ../images/image58.png

#. From the **ACTIVE**, BIG-IP, **Navigate to**:  Network > Interfaces > place a checkmark next to 1.1 & click the **"Disable"** button:

   .. image:: ../images/image59.png

+--------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Question:    | Did the BIG-IP failover? Why or why not?                                                                                                                                                      |
+==============+===============================================================================================================================================================================================+
|| Answer:     || No, the BIG-IP did not failover, as the BIG-IP is not interface-aware; we are still sending failover communication across Interface 1.2 / Data Vlan 20.                                      |
||             || We will address this with HA Group Configuration objects in the following lab.                                                                                                               |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|| Log Output: || Jun 13 11:18:59 bigipB.f5demo.com info lacpd[6714]: 01160016:6: Interface 1.1, link admin status: disabled, link status: up, duplex mode: full, lacp operation state: down                   |
||             || Jun 13 11:18:59 bigipB.f5demo.com info lacpd[6714]: 01160010:6: Link 1.1 removed from aggregation                                                                                            |
||             || Jun 13 11:18:59 bigipB.f5demo.com notice mcpd[7341]: 01bb0003:5: Trunk: int_trunk is DOWN                                                                                                    |
||             || Jun 13 11:18:59 bigipB.f5demo.com notice mcpd[7341]: 01b5004a:5: Link: 1.1 is DISABLED                                                                                                       |
||             || Jun 13 11:19:02 bigipB.f5demo.com warning sod[5345]: 010c0083:4: No failover status messages received for 3.100 seconds, from device bigipA.f5demo.com (10.1.1.5) (unicast: -> 10.1.10.242). |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


#. Now, from the **ACTIVE** BIG-IP, place a checkmark next to interface 1.2, and click the **"Disable"** button: 

   .. image:: ../images/image60.png

+----------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Question | Did the BIG-IPs failover? Are they Active/Standby?                                                                                                                                            |
+==========+===============================================================================================================================================================================================+
|| Answer  || No, the BIG-IPs did NOT failover; both BIG-IPs are in an Active / Standby, In Sync state.                                                                                                    |
||         || The reason for this is, we are allowing/sending Failover communication across the management address. The F5 has an established                                                              |
+----------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|| Logs:   || Jun 13 11:26:46 bigipB.f5demo.com info lacpd[6714]: 01160016:6: Interface 1.2, link admin status: disabled, link status: up, duplex mode: full, lacp operation state: down                   |
||         || Jun 13 11:26:46 bigipB.f5demo.com info lacpd[6714]: 01160010:6: Link 1.2 removed from aggregation                                                                                            |
||         || Jun 13 11:26:46 bigipB.f5demo.com notice mcpd[7341]: 01bb0003:5: Trunk: ext_trunk is DOWN                                                                                                    |
||         || Jun 13 11:26:46 bigipB.f5demo.com notice mcpd[7341]: 01b5004a:5: Link: 1.2 is DISABLED                                                                                                       |
||         || Jun 13 11:26:49 bigipB.f5demo.com warning sod[5345]: 010c0083:4: No failover status messages received for 3.100 seconds, from device bigipA.f5demo.com (10.1.1.5) (unicast: -> 10.1.20.242). |
+----------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

.. note::
   - You can perform a *tcpdump* on the management interface to validate this.
   - This is an example of a tcpdump output, showing BIG-IP-B management communicating with BIG-IP-A management on 1026
      - 11:39:08.158537 IP 10.1.1.6.39126 > 10.1.1.5.cap: failover_packet {
   - For detailed information on failover messages, see Knowledge Article `K67120033: Overview of network failover messages <https://support.f5.com/csp/article/K67120033>`_

#. Verify & Validate the current HA State:
  
  - BIG-IP-B (Active / In Sync): 
    
    .. image:: ../images/image189.png

  - BIG-IP-A (Standby / In Sync):
    
    .. image:: ../images/image190.png


Task 2: Enable both Interfaces on ACTIVE BIG-IP
===============================================

In Task 2, we will re-enable our BIG-IP interfaces to restore connectivity.

#. **Navigate to**: Network > Interfaces > place checkmarks in 1.1 & 1.2, then click the **"Enable"** button:

   .. image:: ../images/image62.png

#.  You should observe that failover did **NOT** occur when our data interfaces went "down." This is due to having our Management Address in our Failover configuration.


Lab Summary
===========

In this lab, you tested & validated bringing down BIG-IP interfaces on the ACTIVE BIG-IP, simulating a "network link failure," and how that affects a failover event.  

What you should observe is that the basic HA configuration does not take into account a link failure.
The following labs will enhance the failover configuration, building a deeper understanding of best practices for HA configuration.

This completes Lab 4.