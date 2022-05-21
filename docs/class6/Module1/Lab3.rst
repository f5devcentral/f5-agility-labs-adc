Lab 3:  Modify BIG-IP settings, Sync, & Perform Validation
----------------------------------------------------------

Lab Tasks:
**********
* Task 1: Modify Self IP Port Lockdown
* Task 2: Verify an Active / Standby "In Sync" State
* Task 3: Create an LTM Node Configuration Object
* Task 4: Perform a Configuration Synchronization between BIG-IPs

Task 1:  Modify Self IP Port Lockdown
=====================================

**On each BIG-IP:**

#. **Navigate to**: Network > Self IPs

#. Click the VLAN 30 Self IP hyperlink

   -  .. image:: ../images/image45.png

#. Under the Port Lockdown drop-down, change the value to "Allow Default," then click the Update button

   -  .. image:: ../images/image46.png

   .. list-table:: 
      :widths: auto
      :align: center
      :header-rows: 1
   
      * - Question:
        - Are the devices Active / Standby? Why or why not?
      * - Answer:
        - No, still Active / Active - Failover via unicast Self IPs & port lockdown

#. On both BIG-IPs, modify the Internal & External Self IP Port Lockdown settings

   -  Change from "Allow None" to "Allow Default"

   .. list-table:: 
      :widths: auto
      :align: center
      :header-rows: 1
   
      * - Question:
        - Are the devices Active / Standby? Why or why not?
      * - Answer:
        - Yes, they are now Active / Standby for Failover
      * - Log Example:
        - ``Apr 28 12:34:47 bigipB.f5demo.com info sod[7297]: 010c0085:6: First failover status message received from device bigipA.f5demo.com (10.1.1.5) (unicast: -> 10.1.20.242).Apr 28 12:34:47 bigipB.f5demo.com warning sod[7297]: 010c0084:4: Failover status message received after 3977.100 second gap, from device bigipA.f5demo.com (10.1.1.5) (unicast: -> 10.1.20.242).``

#. **Bonus Question:** How do you know what "ports" are allowed in the "default" port-lockdown list?

   .. list-table:: 
      :widths: auto
      :align: center
      :header-rows: 0
   
      * - **GUI**
        - **Navigate to**: Help, expand Port Lockdown, and review
        - .. image:: ../images/image47.png
      * - **CLI**
        - tmsh list net self-allow one-line
        - ``net self-allow { defaults { igmp:any ospf:any pim:any tcp:domain tcp:f5-iquery tcp:https tcp:snmp tcp:ssh udp:520 udp:cap udp:domain udp:f5-iquery udp:snmp } }``

Task 2: Verify an Active / Standby "In Sync" State
==================================================

#. On each BIG-IP, review the current state.

   -  BIG-IP-A:

   -  .. image:: ../images/image48.png

   -  BIG-IP-B:

   -  .. image:: ../images/image49.png


#. If your BIG-IPs are in a state other than the above, please review the previous steps to ensure correct configuration.

Task 3: Create an LTM Node Configuration Object
===============================================

In this task, we will create an LTM node object that will be synchronized between BIG-IPs. This will validate that ConfigSync is working correctly. We will perform these changes on the ACTIVE BIG-IP, and then we will sync these changes to the STANDBY BIG-IP.

**On the ACTIVE BIG-IP**

#. **Navigate to**: Local Traffic > Nodes > Node List > click the "+" sign to create a new node

   -  .. image:: ../images/image50.png

#. Create the following Windows Server Node Object, and click Finished

   -  **Name:** win_server

   -  **Address:** 10.1.10.199

   -  .. image:: ../images/image51.png

Task 4:  Perform a Configuration Synchronization between BIG-IPs
================================================================

**On the ACTIVE BIG-IP**

#. Notice the "Changes Pending" in the upper-left corner

   -  .. image:: ../images/image52.png

#. Click this hyperlink to go to the Overview screen.

#. Review the recommendations, and perform a ConfigSync to peer

   -  .. image:: ../images/image53.png

#. While the configuration is being pushed, you will see a "Syncing" icon display in the middle:

   -  .. image:: ../images/image54.png

#. Once the ConfigSync process is complete, your BIG-IPs should indicate an "In Sync" state, and be in an Active / Standby cluster

#. Verify the sync state:

   -  .. image:: ../images/image55.png
