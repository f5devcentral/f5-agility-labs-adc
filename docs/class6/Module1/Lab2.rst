Lab 2:  Configure Device Service Cluster (DSC) High-Availability Settings
-------------------------------------------------------------------------

Lab Tasks:
**********
* Task 1: Define DSC HA Settings
* Task 2: Configure & Verify Device Trust
* Task 3: Configure the Device Group
* Task 4: Validate the Device Group Status

Task 1:  Define Device Service Cluster High-Availability Settings
=================================================================


#. **Navigate to**: Device Management > Devices > click the (Self) hyperlink

   -  .. image:: ../images/image18.png

#. Click the "ConfigSync" banner:

   -  .. image:: ../images/image19.png

#. Under the Local Address drop-down, select the HA VLAN 30 address, then click the Update button

   -  .. image:: ../images/image20.png

#. Click the "Failover Network" banner, then the "Add" button:

   -  .. image:: ../images/image21.png

#. From the New Failover Unicast Address drop-down, select the data-plane VLAN 10 address and click the "Repeat" button:

   -  .. image:: ../images/image22.png

#. From the New Failover Unicast Address drop-down, select the data-plane VLAN 20 address; click the "Finished" button:

   -  .. image:: ../images/image23.png

#. View of the Failover Unicast Configuration:

   -  .. image:: ../images/image24.png

Task 2: Configure & Verify Device Trust between BIG-IPs
=======================================================

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

#. **Navigate to**: Device Management > Device Trust > Device Trust Members page, then click the "+" button to create a new Peer Device

   -  .. image:: ../images/image31.png

#. Retrieve Device Credentials (Step 1 of 3):

   - Fill in the respective form items for *bigipA.f5demo.com*, then click the *Retrieve Device Information* button

   -  .. image:: ../images/image32.png

#. Verify Device Certificate (Step 2 of 3):

   -  Confirm the device certificate information, then click the *Device Certificate Matches* button

   -  .. image:: ../images/image33.png

#. Add Device (Step 3 of 3):

   - Verify the device name, and click the *Add Device* button

   -  .. image:: ../images/image34.png

#. Verify *bigipA.f5demo.com*

   -  Navigate to: Device Management --> Device Trust --> Device Trust Members

   -  .. image:: ../images/image35.png

#. Verify that *bigipB.f5demo.com* is shown in the Peer Device List:

   -  .. image:: ../images/image36.png

+-----------+---------------------------------------------------------+
| Question: | Why are both BIG-IPs Active?                            |
+===========+=========================================================+
| Answer:   | There is no Device Group established between the        |
|           | BIG-IPs yet . . . See next task                         |
+-----------+---------------------------------------------------------+

Task 3:  Configure the Device Group
===================================

On *bigipA.f5demo.com*, set up the new Device Group that will be used by
both BIG-IP systems.

#. **Navigate to**: Device Management > Device Groups page, and then click the "+" button

   -  .. image:: ../images/image37.png

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

   -  .. image:: ../images/image38.png

   -  .. image:: ../images/image39.png

Task 4:  Validate the Device Group Status
=========================================

In this task, you will observe the current Active/Standby HA state.

#. Observe the state of each BIG-IP after Device Group creation

   - bigipA:

   -  .. image:: ../images/image40.png

   - bigipB:

   -  .. image:: ../images/image41.png

#. Review the Device Management Overview screen

#. Attempt the "Recommendation action", and "Sync."

   -  .. image:: ../images/image42.png

   +-----------+---------------------------------------------------------+
   | Question: | Were you able to syncronize the devices?                |
   +===========+=========================================================+
   | Answer:   |                                                         |
   +-----------+---------------------------------------------------------+

#. Review the Overview status screen

   - bigipA:

   -  .. image:: ../images/image43.png

   - bigipB:

   -  .. image:: ../images/image44.png


   +-----------+---------------------------------------------------------+
   | Question: | Why are both BIG-IPs still ACTIVE?                      |
   +===========+=========================================================+
   | Answer:   | Both devices view their peer as "Device is Offline" due |
   |           | to the current Self IP Port Lockdown behavior on the HA |
   |           | VLAN.                                                   |
   +-----------+---------------------------------------------------------+
