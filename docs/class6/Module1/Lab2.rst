Lab 2:  Configure Device Service Cluster (DSC) High-Availability Settings
=========================================================================

Lab Tasks:
==========
* Task 1: Define DSC HA Settings
* Task 2: Configure & Verify Device Trust
* Task 3: Configure the Device Group
* Task 4: Validate the Device Group Status




**Task 1:  Define Device Service Cluster High-Availability Settings**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

..





+--------------+--------------------------------------------------------------+
| Navigate to: | Device Management --> Devices --> click the (Self) hyperlink |
+--------------+--------------------------------------------------------------+

-  .. image:: ../images/image18.png
      :width: 9.89792in
      :height: 4.23125in

..



-  Click the "ConfigSync" banner:

   -  .. image:: ../images/image19.png
         :width: 5.11111in
         :height: 2.19444in



-  Under the Local Address drop-down, select the HA VLAN 30 address,
   then click the Update button:

   -  .. image:: ../images/image20.png
         :width: 5.96319in
         :height: 2.19444in



-  Click the "Failover Network" banner, then the "Add" button:

   -  .. image:: ../images/image21.png
         :width: 7.48125in
         :height: 3.24097in

..



-  From the New Failover Unicast Address drop-down, select the
   data-plane VLAN 10 address; click the "Repeat" button:

   -  .. image:: ../images/image22.png
         :width: 5.90764in
         :height: 2.07431in



-  From the New Failover Unicast Address drop-down, select the
   data-plane VLAN 20 address; click the "Finished" button:

   -  .. image:: ../images/image23.png
         :width: 5.57431in
         :height: 2in

..



-  View of the Failover Unicast Configuration:

   -  .. image:: ../images/image24.png
         :width: 7.34236in
         :height: 2.53681in





**Task 2: Configure & Verify Device Trust between BIG-IPs**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


On device *bigipB.f5demo.com*, setup the Device Trust that will be used
between BIG-IP systems



NOTE: Observe the current status of EACH BIG-IP. Prior to this Task,
they are both in an **Active / Standalone** state. Throughout this
setup, observe the changes in BIG-IP behavior.



+----------------+----------------+----------------+----------------+
| **State**      | **Notes**      | **BIG-IP-A**   | **BIG-IP-B**   |
+================+================+================+================+
| Prior to DSC   | Both in        | .. image:: ver | .. image:: ver |
| Configuration  | "Standalone"   | topal_ef3ad3b0 | topal_ef3ad3b0 |
|                | state          | 65ac47a0834351 | 65ac47a0834351 |
|                |                | 436faa7c57/med | 436faa7c57/med |
|                |                | ia/image25.png | ia/image26.png |
|                |                |    :wi         |    :wi         |
|                |                | dth: 4.62014in | dth: 4.59236in |
|                |                |    :hei        |    :hei        |
|                |                | ght: 1.19444in | ght: 1.19444in |
|                |                |                |                |
|                |                |               |               |
+----------------+----------------+----------------+----------------+
| During Device  | Devices go     | .. image:: ver | .. image:: ver |
|                | into a         | topal_ef3ad3b0 | topal_ef3ad3b0 |
| Peer Trust     | "Disconnected" | 65ac47a0834351 | 65ac47a0834351 |
| join           | state          | 436faa7c57/med | 436faa7c57/med |
|                |                | ia/image27.png | ia/image28.png |
|                |                |    :wi         |    :wi         |
|                |                | dth: 4.76875in | dth: 6.08333in |
|                |                |    :hei        |    :hei        |
|                |                | ght: 1.03681in | ght: 1.23125in |
|                |                |                |                |
|                |                |               |               |
+----------------+----------------+----------------+----------------+
| After Device   | Devices are    | .. image:: ver | .. image:: ver |
|                | BOTH "Active / | topal_ef3ad3b0 | topal_ef3ad3b0 |
| Trust Peer     | In Sync"       | 65ac47a0834351 | 65ac47a0834351 |
| Join           |                | 436faa7c57/med | 436faa7c57/med |
|                |                | ia/image29.png | ia/image30.png |
|                |                |                |    :wi         |
|                |                |  :width: 4.5in | dth: 4.56458in |
|                |                |    :hei        |    :hei        |
|                |                | ght: 1.12014in | ght: 1.14792in |
|                |                |                |                |
|                |                |               |               |
+----------------+----------------+----------------+----------------+



-

+--------------+------------------------------------------------------+
| Navigate to: | Device Management --> Device Trust --> Device Trust  |
|              | Members page, then click the "+" button to create a  |
|              | new Peer Device                                      |
+--------------+------------------------------------------------------+

-  .. image:: ../images/image31.png
      :width: 4.26875in
      :height: 2.39792in



-  Retrieve Device Credentials (Step 1 of 3):

   -  Fill in the respective form items for *bigipA.f5demo.com*, then
      click the *Retrieve Device Information* button:

      -  .. image:: ../images/image32.png
            :width: 3.86111in
            :height: 2.86111in

..



-  Verify Device Certificate (Step 2 of 3):

   -  Confirm the device certificate information, then click the *Device
      Certificate Matches* button:

      -  .. image:: ../images/image33.png
            :width: 8.76875in
            :height: 2.53681in

..



-  Add Device (Step 3 of 3):

   -  Verify the device name, and click the *Add Device* button:

   -  .. image:: ../images/image34.png
         :width: 5.26875in
         :height: 1.10208in

..



-  Verify *bigipA.f5demo.com*

   -  Navigate to: Device Management --> Device Trust --> Device Trust
      Members:

      -  .. image:: ../images/image35.png
            :width: 4.23125in
            :height: 2.36111in

..



-  Verify that *bigipB.f5demo.com* is shown in the Peer Device List:

   -  .. image:: ../images/image36.png
         :width: 9.65764in
         :height: 1.82431in



-

+-----------+---------------------------------------------------------+
| Question: | Why are both BIG-IPs Active?                            |
+===========+=========================================================+
| Answer:   | There is no Device Group established between the        |
|           | BIG-IPs yet . . . See next Task                         |
+-----------+---------------------------------------------------------+





**Task 3:  Configure the Device Group**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


On *bigipA.f5demo.com*, set up the new Device Group that will be used by
both BIG-IP systems.



-  On *bigipA.f5demo.com*:

   -

+--------------+------------------------------------------------------+
| Navigate to: | Device Management --> Device Groups page, and then   |
|              | click the "+" button:                                |
+--------------+------------------------------------------------------+

-  .. image:: ../images/image37.png
      :width: 2.62014in
      :height: 1.87014in

..



-  Create a Device Group using the following information, and then click
   Finished:

   -

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

..



-  .. image:: ../images/image38.png
      :width: 6.86111in
      :height: 4.13889in

..



-  .. image:: ../images/image39.png
      :width: 6.46319in
      :height: 3.99097in



**Task 4:  Validate the Device Group Status**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _section-2:


-

In this task, you will observe the current Active/Standby HA state.



-  Observe the state of each BIG-IP after Device Group creation:

   -  BIG-IP-A:

      -  .. image:: ../images/image40.png
            :width: 4.57431in
            :height: 1.19444in

..



-  BIP-IP-B:

   -  .. image:: ../images/image41.png
         :width: 4.57431in
         :height: 1.09236in

..



-  Review the Device Management Overview screen.

   -  Attempt the "Recommendation action", and "Sync."

      -  **Is it successful?**

         -  .. image:: ../images/image42.png
               :width: 16.04653in
               :height: 6.13889in



-  Review the Overview status screen.

   -  BIG-IP-A:

      -  .. image:: ../images/image43.png
            :width: 5.52778in
            :height: 3.85208in

   -  BIG-IP-B:

      -  .. image:: ../images/image44.png
            :width: 5.51875in
            :height: 3.28681in



-

+-----------+---------------------------------------------------------+
| Question: | Why are both BIG-IPs still ACTIVE?                      |
+===========+=========================================================+
| Answer:   | Both devices view their peer as "Device is Offline" due |
|           | to the current Self IP Port Lockdown behavior on the HA |
|           | VLAN.                                                   |
+-----------+---------------------------------------------------------+





