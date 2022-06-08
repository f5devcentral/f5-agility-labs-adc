Lab 3:  Modify BIG-IP settings, Sync, & Perform Validation
----------------------------------------------------------

In Lab 3, we will modify port lockdown settings, and validate BIG-IP configuration synchronization.

Lab Tasks:
==========

* Task 1: Modify Self IP Port Lockdown
* Task 2: Verify an Active / Standby "In Sync" State

Task 1:  Modify Self IP Port Lockdown
=====================================

In Task 1, we will modify our "Allow None" Self IP port lockdown behavior; we will define a Custom Port Lockdown configuration on the respective Self IPs.

For optimal security, F5 recommends that you use the port lockdown feature to allow only the protocols or services required for a self IP address.

* For our HA VLAN, we will select **"Allow Default"**
* For our Data VLANs (internal & external), we will **"Allow Custom"**, allowing **UDP** protocol on port 1026.

There are port lockdown exceptions to be aware of.  Please review Knowledge Article `K17333 <https://support.f5.com/csp/article/K17333>`_ for further details.
 
In Lab 1, when we created our Local Self IPs, we chose to select the "Allow None" port lockdown behavior.  As a result of this, the BIG-IP is preventing DSC communication between BIG-IPs.  In this Task, we will modify our port lockdown configuration, which will allow DSC communication between BIG-IPs.


**On each BIG-IP:**

*NOTE Do the modifications only on the SELF-IP. **DO NOT** modify the floating IP Address port lockdown. The floating IP address port lockdown status has to be **"none"**


#. **Navigate to**: Network > Self IPs

#. Click the VLAN 30 Self IP hyperlink

   .. image:: ../images/image45.png

#. Under the Port Lockdown drop-down, change the value to "Allow Default," then click the Update button

   .. image:: ../images/image46.png

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
      
      - From the Port Lockdown drop-down, select "Allow Custom." 
      - Click the radio button for UDP.  
      - Click the radio button for Port.  
      - In the Port field, enter 1026.  
      - Click Add.
      
      .. image:: ../images/image112.png
      
      You should see "1026" listed in the UDP Custom List section.  Click Update.
         
      .. image:: ../images/image113.png

   - Repeat this step on the External VLAN







#. On BIG-IP-A, modify the Floating Self IPs per the previous steps:
    - Floating VLAN 30 = Allow Default, then Click Update:
  
      .. image:: ../images/image150.png

    - Floating VLAN 10 & 20 = Allow Custom, UDP port 1026, then Click Update:
      
      -  VLAN 10:
    
         .. image:: ../images/image151.png

      -  VLAN 20:
  
         .. image:: ../images/image152.png


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
   
      * - **CLI**
        - tmsh list net self-allow one-line
        - ``net self-allow { defaults { igmp:any ospf:any pim:any tcp:domain tcp:f5-iquery tcp:https tcp:snmp tcp:ssh udp:520 udp:cap udp:domain udp:f5-iquery udp:snmp } }``

   For more information check following K-Article: https://support.f5.com/csp/article/K17333
   

Task 2: Verify an Active / Standby "In Sync" State
==================================================

In Task 2, we will perform a BIG-IP device synchronization.  This will validate our HA DSC configuration.

#. On each BIG-IP, review the current state.

   -  BIG-IP-A:

      .. image:: ../images/image48.png

   -  BIG-IP-B:

      .. image:: ../images/image49.png


#. If your BIG-IPs are in a state other than the above, please review the previous steps to ensure correct configuration.


Lab Summary
===========

In this lab, you modified BIG-IP Self IP port lockdown configuration settings.  After completion of these lab tasks, you should have an Active/Standby High-Availability (HA) BIG-IP configuration, with Local Traffic Manager (LTM) objects, that are synchronized between the BIG-IPs.  These configuration objects will assist with the following labs.

This completes Lab 3.