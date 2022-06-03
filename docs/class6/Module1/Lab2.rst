Lab 2:  Configure Device Service Cluster (DSC) High-Availability Settings
-------------------------------------------------------------------------

In Lab 2, we will configure DSC configuration objects, which will assist with establishing a device-trust between BIG-IPs, allowing a successful highly-available Active/Standby BIG-IP pair.

For addtional details on DSC, please refer to this Article: `BIG-IP Device Service Clustering: Administration <https://techdocs.f5.com/en-us/bigip-14-1-0/big-ip-device-service-clustering-administration-14-1-0.html>`_

Lab Tasks:
**********
* Task 1: Define DSC HA Settings
* Task 2: Configure & Verify Device Trust
* Task 3: Configure the Device Group
* Task 4: Setup MAC Masquerade
* Task 5: Create Floating Self IPs on BIG-IP-A
* Task 6: Validate the Device Group Status

Task 1:  Define Device Service Cluster High-Availability Settings
=================================================================

In Task 1, we will define our respective DSC configuration items.

#. **Navigate to**: Device Management > Devices > click the (Self) hyperlink:

     .. image:: ../images/image18.png

#. Click the "ConfigSync" banner:

     .. image:: ../images/image19.png

#. Under the Local Address drop-down, select the HA VLAN 30 address, then click the Update button

     .. image:: ../images/image20.png

#. Click the "Failover Network" banner, then the "Add" button:

     .. image:: ../images/image21.png

#. From the New Failover Unicast Address drop-down, select the Management Address, and click the "Repeat" button:

     .. image:: ../images/image115.png


#. From the New Failover Unicast Address drop-down, select the data-plane VLAN 10 address and click the "Repeat" button:

     .. image:: ../images/image22.png

#. From the New Failover Unicast Address drop-down, select the data-plane VLAN 20 address; click the "Finished" button:

     .. image:: ../images/image23.png

#. View of the Failover Unicast Configuration:

     .. image:: ../images/image24.png

#. Click the "Mirroring" banner:

     .. image:: ../images/image110.png


#. From the Primary Local Mirror Adddress drop-down, select the HA VLAN 30 address, and click the "Update" button:

     .. image:: ../images/image111.png

To take advantage of Connection Mirroring, there are addtional BIG-IP configuration items to configure, specifically as it relates to the Virtual Server.  We will address this configuration in Lab 3.  

For information on enabling connection mirroring for your Virtual Server, please refer to this link, `Enable connection mirroring for a virtual server <https://support.f5.com/csp/article/K84303332#s2>`_

For more information on Connection Mirroring Configuration, please refer to Knowledge Article `K84303332 <https://support.f5.com/csp/article/K84303332>`_


Task 2: Configure & Verify Device Trust between BIG-IPs
=======================================================

In Task 2, we will define the configuration to establish our device-trust between BIG-IPs.

On device *bigipB.f5demo.com*, setup the Device Trust that will be used between BIG-IP systems

   NOTE: Observe the current status of EACH BIG-IP. Prior to this Task, they are both in an **Active / Standalone** state. Throughout this setup, observe the changes in BIG-IP behavior.

.. list-table:: 
   :widths: auto
   :align: center
   :header-rows: 1

   * - State
     - Notes
     - bigipA
     - bigipB
   * - Prior to DSC configuration
     - Both devices in "Standalone" state
     -  .. image:: ../images/image25.png
     -  .. image:: ../images/image26.png
   * - During device peer join trust
     - Both devices enter "Disconnected" state
     -  .. image:: ../images/image27.png
     -  .. image:: ../images/image28.png
   * - After device peer join trust
     - *Both* devices enter "Active / In Sync" state
     -  .. image:: ../images/image29.png
     -  .. image:: ../images/image30.png

#. **Navigate to**: Device Management > Device Trust > Device Trust Members page, then click the "+" button to create a new Peer Device:

     .. image:: ../images/image31.png

#. Retrieve Device Credentials (Step 1 of 3):

   Fill in the respective form items for *bigipA.f5demo.com*, then click the *Retrieve Device Information* button

     .. image:: ../images/image32.png

#. Verify Device Certificate (Step 2 of 3):

   Confirm the device certificate information, then click the *Device Certificate Matches* button

     .. image:: ../images/image33.png

#. Add Device (Step 3 of 3):

   Verify the device name, and click the *Add Device* button

     .. image:: ../images/image34.png

#. Verify *bigipA.f5demo.com*

   Navigate to: Device Management --> Device Trust --> Device Trust Members

     .. image:: ../images/image35.png

#. Verify that *bigipB.f5demo.com* is shown in the Peer Device List:

     .. image:: ../images/image36.png

+-----------+---------------------------------------------------------+
| Question: | Why are both BIG-IPs Active?                            |
+===========+=========================================================+
| Answer:   | There is no Device Group established between the        |
|           | BIG-IPs yet . . . See next task                         |
+-----------+---------------------------------------------------------+

Task 3:  Configure the Device Group
===================================

In Task 3, we will define the device group on the BIG-IPs.

On *bigipA.f5demo.com*, set up the new Device Group that will be used by
both BIG-IP systems.

#. **Navigate to**: Device Management > Device Groups page, and then click the "+" button:

     .. image:: ../images/image37.png

#. Create a Device Group using the following information, and then click Finished

   +-------------+-------------------------------------------------------+
   | Name        | [give device group a name]                            |
   |             |                                                       |
   |             | Example: bigip-a_bigip-b_dg                           |
   +=============+=======================================================+
   | Group Type  | Sync-Failover                                         |
   +-------------+-------------------------------------------------------+
   | Description | [OPTIONAL] provide a description                      |
   |             |                                                       |
   |             | Example: Sync-failover Device Group for BIG-IP A & B  |
   +-------------+-------------------------------------------------------+
   | Members     | Move both bipipA & bipipB *from the Available* column |
   |             | to the *Includes* column                              |
   +-------------+-------------------------------------------------------+

     .. image:: ../images/image38.png

     .. image:: ../images/image39.png

Task 4:  Setup MAC Masquerade
=============================

In Task 4, we will setup MAC masquerading at the traffic-group level, allowing a "floating MAC" to be shared across the traffic-group.  

To optimize the flow of traffic during failover events, you can configure MAC masquerade addresses for any defined traffic groups on the BIG-IP system. A MAC masquerade address is a unique, floating MAC address that you create. You can assign one MAC masquerade address to each traffic group on a BIG-IP device. 

In Virtualized environments, there are some configuration caveats to be aware of; please review the **Notes** section in Article `K13502: Configuring MAC masquerade (11.x - 16.x) <https://support.f5.com/csp/article/K13502>`_

First, we need to obtain a Unique MAC address to use for our MAC Masquerade.  We will leverage one of our Virtual Interfaces MACs; we'll flip the 1st MAC HEX value to "02."

1.  **Navigate to**: Network > Interfaces, and copy the 1.1 MAC address to your "copy/paste" machine buffer:
   
     .. image:: ../images/image116.png

2.  Now, **Navigate to**: Device Management > Traffic Groups > click the traffic-group-1 hyperlink:
   
     .. image:: ../images/image117.png

3.  In the MAC Masquerade Address Field, paste the previously saved MAC Address:
   
     .. image:: ../images/image118.png

     Replace the "52" with "02" and click Save

     .. image:: ../images/image119.png

4. After this change, Sync your BIG-IP configurations.
   
     .. image:: ../images/image120.png


Task 5:  Create Floating Self IPs on BIG-IP-A
==================================================

In Task 5, we will define Floating Self IP Objects on the BIG-IP-A, which are shared objects between an Active/Standby BIG-IP pair.  

On the BIG-IP-A, create the following Floating Self IP Objects.  These will be shared configuration objects that will be synced in Lab 3.  Only create the Floating Self IPs on BIG-IP-A:

   Use the following table to create & define your three Self IPs:

   .. list-table:: 
      :widths: auto
      :align: center
      :header-rows: 1
   
      * - BIG-IP
        - Name
        - IP address
        - Netmask
        - VLAN
        - Port Lockdown
      * - bigipA
        - self_vlan10_float
        - 10.1.10.240
        - 255.255.255.0
        - int_vlan_10
        - Allow None (default)
      * - bigipA
        - self_vlan20_float
        - 10.1.20.240
        - 255.255.255.0
        - ext_vlan_20
        - Allow None (default)
      * - bigipA
        - self_vlan30_float
        - 10.1.30.240
        - 255.255.255.0
        - HA_vlan_30
        - Allow None (default)

#. **Navigate to**: Network > Self IPs, then click the "+" button to create a new Self IP:

     .. image:: ../images/image13.png

#. Create the respective Self IPs per the table above.

VLAN 10 Float:
     .. image:: ../images/image144.png

VLAN 20 Float:
     .. image:: ../images/image145.png

VLAN 30 Float:
     .. image:: ../images/image146.png

After creation of your Floating Self IPs, your Self IP List should reflect the following on BIG-IP-A:
     .. image:: ../images/image147.png

Task 6:  Validate the Device Group Status
=========================================

In Task 6, you will observe the current Active/Standby HA state.

#. Observe the state of each BIG-IP after Device Group creation

   - bigipA:

     .. image:: ../images/image40.png

   - bigipB:

     .. image:: ../images/image41.png

#. Review the Device Management Overview screen

#. Attempt the "Recommendation action", and "Sync."

     .. image:: ../images/image42.png

   +-----------+---------------------------------------------------------+
   | Question: | Were you able to syncronize the devices?                |
   +===========+=========================================================+
   | Answer:   |                                                         |
   +-----------+---------------------------------------------------------+

#. Review the Overview status screen

   - bigipA:

     .. image:: ../images/image43.png

   - bigipB:

     .. image:: ../images/image44.png


   +-----------+---------------------------------------------------------+
   | Question: | Why are both BIG-IPs still ACTIVE?                      |
   +===========+=========================================================+
   | Answer:   | Both devices view their peer as "Device is Offline" due |
   |           | to the current Self IP Port Lockdown behavior on the HA |
   |           | VLAN.                                                   |
   +-----------+---------------------------------------------------------+

Lab Summary
***********
In this lab, you setup BIG-IP Device Service Clustering (DSC) configuration settings.  After completion of these lab tasks, you should have the required configuration to assist in establishing your DSC between BIG-IPs.  These configuration objects will assist with the subsequent labs.

This completes Lab 2.