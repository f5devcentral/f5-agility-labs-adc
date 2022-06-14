
Lab 6: Validating & Testing HA Group Functionality
--------------------------------------------------

Now that we have succesfully configured HA Groups we will perform a series of validation tests.

Lab Tasks:
==========

* Task 1: Disable an interface to force HA Group actions
* Task 2: Re-enable Interface, and Observe BIG-IP Behavior


Task 1: Disable an interface to force HA Group actions
======================================================

In this task we will: 

* Manipulate (Disable) an interface and observe BIG-IP behavior during failover
* Monitor logs to review the failover process


On the **ACTIVE** BIG-IP: 

#. **Navigate to**: Network > Interfaces, and place checkmark next to interface 1.1 then click on the **Disable** button.

   .. image:: ../images/image100.PNG

#. During this time, observe the BIG-IP status in the upper-left corner of each BIG-IP.

   .. image:: ../images/image101a.png


   .. image:: ../images/image102a.png


+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Question   | Did a failover event occur, and did the BIG-IP state change?                                                                                                                |
+============+=============================================================================================================================================================================+
| Answer     | Yes, failover occurred immediately upon a link down failure. BIG-IP-A went from **ACTIVE** to **Standby**                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|| CLI Logs: || **BIG-IP-A:**                                                                                                                                                              |
||           ||                                                                                                                                                                            |
||           || Jun 14 07:42:38 bigipA.f5demo.com info lacpd[5031]: 01160016:6: Interface 1.1, link admin status: disabled, link status: up, duplex mode: full, lacp operation state: down |
||           || Jun 14 07:42:38 bigipA.f5demo.com info lacpd[5031]: 01160010:6: Link 1.1 removed from aggregation                                                                          |
||           || Jun 14 07:42:38 bigipA.f5demo.com notice mcpd[6517]: 01bb0003:5: Trunk: int_trunk is DOWN                                                                                  |
||           || Jun 14 07:42:38 bigipA.f5demo.com notice mcpd[6517]: 010719fb:5: **HA Group bigip-ha-group score updated from 30 to 0.**                                                   |
||           || Jun 14 07:42:38 bigipA.f5demo.com notice mcpd[6517]: 01b5004a:5: Link: 1.1 is DISABLED                                                                                     |
||           || Jun 14 07:42:38 bigipA.f5demo.com notice sod[4368]: 010c0045:5: **Leaving active, group score 0 peer group score 30.**                                                     |
||           || Jun 14 07:42:38 bigipA.f5demo.com notice sod[4368]: 010c0052:5: **Standby for traffic group traffic-group-1.**                                                             |
||           ||                                                                                                                                                                            |
||           || **BIG-IP-B:**                                                                                                                                                              |
||           ||                                                                                                                                                                            |
||           || Jun 14 07:42:38 bigipB.f5demo.com notice sod[5359]: 010c006d:5: **Leaving Standby for Active (best ha score).**                                                            |
||           || Jun 14 07:42:38 bigipB.f5demo.com notice sod[5359]: 010c0053:5: **Active for traffic group traffic-group-1.**                                                              |
||           || Jun 14 07:42:38 bigipB.f5demo.com notice sod[5359]: 010c0019:5: Active                                                                                                     |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

#. Observe the log messages from each BIG-IP.

#. **Navigate to**: System > Logs > Local Traffic, enter the text *score* into the free-form text field, and click the **Search** button:

  .. image:: ../images/image194.png

  .. image:: ../images/image197.png


#. Previously, BIG-IP-A was the **ACTIVE** device.   Conversely, BIG-IP-B was previously the **STANDBY** device.  Their roles have now flipped, making BIG-IP-B **ACTIVE** and BIG-IP-A **STANDBY**:

  - BIG-IP-A (now **STANDBY**):
  
  .. image:: ../images/image195.png

  - BIG-IP-B (now **ACTIVE**):
  
  .. image:: ../images/image196.png

Task 2: Re-enable Interface, and Observe BIG-IP Behavior
========================================================

We will now re-enable Interface 1.1 on the Standby BIG-IP.

#. On the standby BIG-IP, **Navigate to**: Network > Interfaces > Interface List, and place checkmark next to interface 1.1 then click the **Enable** button.
  
   .. image:: ../images/image93.png

#. Did a failover event occur?   If so, why or why not?

Lab Summary
===========
In this lab, you tested & validated bringing down a BIG-IP interface, simulating a "link failure," and how that affects a failover event with HA Groups.  After completion of these lab tasks, you should have a better understanding of how the BIG-IP behaves with an advanced HA Group Configuration.

This completes Lab 6.

Supplemental Resources:
=======================

- `K16947: F5 recommended practices for the HA group feature <https://support.f5.com/csp/article/K16947>`_
- `Manual : BIG-IP Device Service Clustering: Administration <https://techdocs.f5.com/en-us/bigip-14-1-0/big-ip-device-service-clustering-administration-14-1-0.html>`_
- `Manual Chapter : Creating an Active-Standby Configuration using the Configuration Utility <https://techdocs.f5.com/en-us/bigip-14-1-0/big-ip-device-service-clustering-administration-14-1-0/creating-an-active-standby-configuration-using-the-configuration-utility.html>`_
- `K13946: Troubleshooting ConfigSync and device service clustering issues <https://support.f5.com/csp/article/K13946>`_
- `Archived - K41983050: The HA group score displayed with tmsh shows that the sys ha-group command no longer includes the active bonus <https://support.f5.com/csp/article/K41983050>`_


.. note:: 
  
   * Device group members should be able to communicate over ports 443, 4353, 1026 (UDP), and 22 (recommended)
   * BIG-IP ASM requires the following additional Policy Sync TCP ports: 6123-6128
