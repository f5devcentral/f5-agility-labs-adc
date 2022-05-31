Lab 3:  Modify BIG-IP settings, Sync, & Perform Validation
----------------------------------------------------------

Lab Tasks:
**********
* Task 1: Modify Self IP Port Lockdown
* Task 2: Verify an Active / Standby "In Sync" State
* Task 3: Create LTM Virtual Server Configuration Objects
* Task 4: Perform a Configuration Synchronization between BIG-IPs

Task 1:  Modify Self IP Port Lockdown
=====================================

For optimal security, F5 recommends that you use the port lockdown feature to allow only the protocols or services required for a self IP address.  In this Task, we will define a Custom Port Lockdown configuration on the respective Self IPs.

* For our HA VLAN, we will select **"Allow Default"**
* For our Data VLANs (internal & external), we will **"Allow Custom"**, allowing **UDP** protocol on port 1026.

There are port lockdown exceptions to be aware of.  Please review Knowledge Article `K17333 <https://support.f5.com/csp/article/K17333>`_ for further details.

In this Task, we will modify the Self IP port lockdown behavior.
 
When we created our Local Self IPs, we chose to select the "Allow None" port lockdown behavior.  As a result of this, the BIG-IP is preventing DSC communication between BIG-IPs.  In this Task, we will modify our port lockdown configuration, which will allow DSC communication between BIG-IPs.


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

#. On both BIG-IPs, modify both the Internal & External Self IP Port Lockdown settings

   -  Change from "Allow None" to **"Allow Custom"**
      - From the Port Lockdown drop-down, select "Allow Custom."  Click the radio button for UDP.  Click the radio button for Port.  In the Port field, enter 1026.  Click Add.
        -  .. image:: ../images/image112.png
        -  You should see "1026" listed in the UDP Custom List section.  Click Update.
            -  .. image:: ../images/image113.png
   - Repeat this step on the External VLAN

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

Task 3: Create LTM Virtual Server Configuration Objects
=======================================================

In this task, we will create LTM Virtual Server configuration objects that will be synchronized between BIG-IPs. This will validate that ConfigSync is working correctly. We will perform these changes on the ACTIVE BIG-IP, and then we will sync these changes to the STANDBY BIG-IP.

We will create two pool objects:
1.  Back-end Server Pool - this will be used as the pool for the Virtual Server object
2.  External Gateway Pool - this will be used in our HA Group, to validate the availability of our external gateway

**On the ACTIVE BIG-IP, Create Pool & Node Objects**

#. **Navigate to**: Local Traffic > Pools > Pool List > click the "+" sign to create a new pool

   -  .. image:: ../images/image114.png

#. Create the following Pool Configuration Objects:

**Pool 1**
   -  **Name:** server_pool
   -  **Health Monitors:** gateway_icmp
   -  Within the Resources Section:
       -  **New Node Address:** 10.1.10.199
       -  **Service Port:** *All Services
       - Click the Add button
   - Click the Repeat Button to Create Pool 2
   -   .. image:: ../images/image123.png


**Pool 2**
   -  **Name:** ext_gw_pool
   -  **Health Monitors:** gateway_icmp
   -  Within the Resources Section:
       -  **New Node Address:** 10.1.20.1
       -  **Service Port:** *All Services
       - Click the Add button
           -   .. image:: ../images/image124.png

       - **NOTE** You will need to highlight the previous Node Object (10.1.10.199) and click the Delete button to remove from the Node List.
            -   .. image:: ../images/image125.png

   -  Click the Finished Button
   -   .. image:: ../images/image126.png

After completion of this task, you should be presented with the following 2 pools:
   -   .. image:: ../images/image127.png

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


Lab Summary
^^^^^^^^^^^
In this lab, you configured addtional BIG-IP configuration objects.  After completion of these lab tasks, you should have an Active/Standby High-Availability (HA) BIG-IP configuration, with Local Traffic Manager (LTM) objects, that are synchronized between the BIG-IPs.  These configuration objects will assist with the following labs.

This completes Lab 3.