
Lab 6: Validating & Testing HA Group Functionality
--------------------------------------------------

Now that we have succesfully configured HA Groups we will perform a series of validation tests.

Lab Tasks:
==========

* Task 1: Disable an interface to force HA Group actions
* Task 2: Verify BIG-IP Logs
* Task 3: Verify HA Score
* Task 4: Re-enable Interface, and Observe BIG-IP Behavior
* Task 5: Test & Validate Gateway Pool Failure
* Task 6: Verify HA Score
* Task 7: Restore GW Pool & Sync BIG-IPs

Task 1: Disable an interface to force HA Group actions
======================================================

In this task we will: 

* Manipulate (Disable) an interface and observe BIG-IP behavior during failover
* Monitor logs to review the failover process


On the **ACTIVE** BIG-IP: 

#. **Navigate to**: Network > Interfaces, and place checkmark next to interface 1.1 then click on the **Disable** button:

   .. image:: ../images/image100.PNG

#. During this time, observe the BIG-IP status in the upper-left corner of each BIG-IP.

   .. image:: ../images/image101a.png


   .. image:: ../images/image102a.png


+----------+-----------------------------------------------------------------------------------------------------------+
| Question | Did a failover event occur, and did the BIG-IP state change?                                              |
+==========+===========================================================================================================+
| Answer   | Yes, failover occurred immediately upon a link down failure. BIG-IP-A went from **ACTIVE** to **Standby** |
+----------+-----------------------------------------------------------------------------------------------------------+

Task 2: Verify BIG-IP Logs
==========================

#. Observe the log messages from each BIG-IP.

#. **Navigate to**: System > Logs > Local Traffic, enter the text *score* into the free-form text field, and click the **Search** button:

   .. image:: ../images/image194.png

   .. image:: ../images/image197.png

#. Previously, BIG-IP-A was the **ACTIVE** device.   Conversely, BIG-IP-B was previously the **STANDBY** device.  Their roles have now flipped, making BIG-IP-B **ACTIVE** and BIG-IP-A **STANDBY**:

   - BIG-IP-A (now **STANDBY**):
  
     .. image:: ../images/image195.png

   - BIG-IP-B (now **ACTIVE**):
  
     .. image:: ../images/image196.png

#. Log output example from CLI (/var/log/ltm):

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



Task 3: Verify HA Score
=======================

In this Task, we will verify the BIG-IP HA Score values.

#. On each BIG-IP, **Navigate to**: System > High Availability > HA Group List, then click the HA Group name hyperlink:

   .. image:: ../images/image198.png


   .. image:: ../images/image199.png

#. Observe how each BIG-IP is honoring their HA Group objects to contribute to their HA Score. The highest score will be **ACTIVE**:
  
   - BIG-IP-A:

     .. image:: ../images/image200.png


   - BIG-IP-B:
   
     .. image:: ../images/image201.png

.. note:: 

   - You can also view this information from CLI with the following *tmsh* command:
       - *tmsh show sys ha-group detail*

Task 4: Re-enable Interface, and Observe BIG-IP Behavior
========================================================

We will now re-enable Interface 1.1 on the **STANDBY** BIG-IP.

#. On the **STANDBY** BIG-IP, **Navigate to**: Network > Interfaces > Interface List, and place checkmark next to interface 1.1 then click the **Enable** button:
  
   .. image:: ../images/image93.png

+----------+----------------------------------------------------------------------------------------------------------------------------------------------+
| Question | Did a failover event occur? If so, why or why not?                                                                                           |
+==========+==============================================================================================================================================+
| Answer   | No, the BIG-IPs did not failover because the **ACTIVE** BIG-IP HA Score did **NOT** change; the **ACTIVE** bonus kept this device **ACTIVE** |
+----------+----------------------------------------------------------------------------------------------------------------------------------------------+

Task 5: Test & Validate Gateway Pool Failure
============================================

In this Task, we will manipulate our upstream gateway pool to simulate an upstream network / path failure. This will validate an addtional HA Group object, and how it affects BIG-IP HA failover.

We will **force offline** our gateway pool member to force the pool to fail, causing a gateway pool failure.

#. On the **ACTIVE** BIG-IP, **Navigate to**: Local Traffic > Pools > Pool List, and click the **ext_gw_pool** hyperlink:
 
   
   .. image:: ../images/image202.png

#. Click the **Members** tab:

   .. image:: ../images/image206.png

#. Place a checkmark next to the Member, and click the **Force Offline** button:

   .. image:: ../images/image207.png

#. Observe the BIG-IP HA state, and answer the following:

+----------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Question | Did a failover event occur? If so, why or why not?                                                                                                                             |
+==========+================================================================================================================================================================================+
|| Answer  || Yes, forcing the gateway pool member offline causes a gateway pool failure on the **ACTIVE** BIG-IP, causing the HA Score to drop to "zero", causing a BIG-IP failover event. |
||         || You can validate this by reviewing the HA Group Score and/or logs.                                                                                                            |
+----------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Task 6: Verify HA Score
=======================

In this Task, we will validate HA Group Score on both BIG-IPs.

#. On each BIG-IP, **Navigate to**: System > High Availability > HA Group List, then click the HA Group name hyperlink:

   .. image:: ../images/image198.png


   .. image:: ../images/image199.png

#. Observe the Pool object on **STANDBY** BIG-IP.  Due to the gateway member failure (forced offline), it is **NOT** contributing to the HA Score.

   BIG-IP-B (now STANDBY):

   .. image:: ../images/image208.png

   BIG-IP-A (now ACTIVE)

   .. image:: ../images/image209.png

Task 7: Restore GW Pool & Sync BIG-IPs
======================================

In this Task, prior to proceeding to Lab 7, we need to restore our gateway pool member on the **STANDBY** BIG-IP, and synchronize BIG-IP configurations.

#. On the **STANDBY** BIG-IP, **Navigate to**: Local Traffic > Pools > Pool List, and click the **ext_gw_pool** hyperlink:
 
   
   .. image:: ../images/image210.png

#. Click the **Members** tab:


   .. image:: ../images/image211.png


#. Place a checkmark next to the Member, and click the **Enable** button:

   .. image:: ../images/image212.png

#. Refresh the **Members** page, and confirm a green pool member resource:

   .. image:: ../images/image213.png

#. Click the **Changes Pending** hyperlink, and review the recommendations.  Perform the recommendations, and Sync BIG-IPs:

   .. image:: ../images/image214.png

   .. image:: ../images/image215.png

After this Task, your BIG-IPs should be **In Sync** and **Active/Standby**.

Lab Summary
===========
In this lab, you tested & validated bringing down a BIG-IP interface, simulating a "link failure," and how that affects a failover event with HA Groups.  

We also observed how a gateway pool can affect the HA Score, and Failover timing.  These HA Group objects contribute to the overall HA Score health, with the highest HA Score becoming the **ACTIVE** BIG-IP.

After completion of these lab tasks, you should have a better understanding of how the BIG-IP behaves with an advanced HA Group Configuration.

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
