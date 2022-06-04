
Lab 6: Validating, Testing & Troubleshooting HA Group Functionality
-------------------------------------------------------------------

Now that we have succesfully configured HA Groups we will perform a series of validation tests, as well as provide tips for troubleshooting.


Lab Tasks:
==========

* Task 1: Disable an interface to force HA Group actions
* Task 2: Re-enable Interface, and Observe BIG-IP Behavior


Task 1: Disable an interface to force HA Group actions
======================================================

In this task we will: 

* Manipulate an interface on the active BIG-IP
* Disable an interface and observe BIG-IP behavior during failover
* Monitor logs to review the failover process


On the active BIG-IP: 

* Navigate to Network, Interfaces, and place  checkmark next to interface 1.1 then click on the **Disable** button.


  .. image:: ../images/image100.PNG



* During this time observe the BIG-IP status in the upper-left corner of each BIG-IP.  Did a failover event occur and did state change?

  .. image:: ../images/image101.png

  .. image:: ../images/image102.png

  Observe the log messages from each BIG-IP.  Previously, BIG-IP A was the Standby device.   Conversely, BIG-IP B was previously the Active
      device.

  .. image:: ../images/image91.png

  .. image:: ../images/image92.png


Task 2: Re-enable Interface, and Observe BIG-IP Behavior
========================================================

In Task 2, we will now re-enable Interface 1.1 on the Standby BIG-IP by performing the following step.

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


**NOTE**:
   Ports	Device group members should be able to communicate over ports 443, 4353, 1026 (UDP), and 22 (recommended).
   BIG-IP ASM requires the following additional Policy Sync TCP ports: 6123-6128
