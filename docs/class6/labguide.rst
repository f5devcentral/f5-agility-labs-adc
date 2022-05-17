BIG-IP HA Failover with HA Groups - Lab Guide
=============================================


Lab Description:
================

Explore additional Application Deliver Controller (ADC)
High-Availability (HA) enhancements to create a highly-resilient
failover environment. This lab will demonstrate how additional Device
Management configuration objects can create a quicker failover mechanism
between an Active/Standby BIG-IP pair.


During this hands-on lab, you will learn the following:

-  How to establish a successful device-trust between BIG-IPs

-  Understand log messaging as it relates to failover scenarios

-  How to create an HA Group, and respective Configuration Objects

-  How to leverage the construct of HA Groups vs. Preferred Order for
   High-Availability Failover

-  Understand how the HA Group Objects affect the overall HA Score

..

   

Lab Assumptions:
================

-  BIG-IPs are already licensed, and provisioned appropriately

-  Basic 101-level BIG-IP Administration

-  Familiar with BIG-IP Local Traffic Management (LTM) / Application
   Delivery Controller (ADC)

-  We are using a virtualized environment, simulating "physical trunk"
   links to assist with HA Group Failover

-  BIG-IP devices are in a "standalone" state for initial deployment

-  BIG-IP Self IPs, VLANs, Routes, Trunks, and Nodes have already been
   configured

   -  You will create Device Trust between BIG-IPs

   -  Initial Device Service Clustering (DSC) configuration is present:

      -  ConfigSync = Internal Self IP of BIG-IPs

      -  Failover Network = Internal Self IPs, and Management Address

-  Walk-through is designed to be completed via the GUI / TMUI; however,
   many of the configuration objects could be *automated* via API calls
   to the BIG-IP

..

   

Goal:
=====

-  Understand HA Groups, and how they can assist with automatic HA
   failover

-  Leverage best practices for HA Group configuration objects

-  Observe logs to understand failover function



Lab Tasks:
==========



=================== ========================
.. rubric:: Task 1: **Create BIG-IP Trunks**
   :name: task-1    
=================== ========================



   On both BIG-IP devices, configure trunks under the Network
   configuration section.

   

   Use the following table to create & define your three Trunks:



+----------------+----------------------+-------------------------+
| **Trunk Name** | **Interface Member** | **Description /         |
|                |                      | Function**              |
+================+======================+=========================+
| int_trunk      | 1.1                  | Trunk to simulate a     |
|                |                      | connection to internal  |
|                |                      | infrastructure          |
+----------------+----------------------+-------------------------+
| ext_trunk      | 1.2                  | Trunk to simulate a     |
|                |                      | connection to external  |
|                |                      | infrastructure          |
+----------------+----------------------+-------------------------+
| HA_trunk       | 1.3                  | Trunk to simulate a     |
|                |                      | high-availability       |
|                |                      | network connection      |
+----------------+----------------------+-------------------------+



-  

+--------------+------------------------------------------------------+
| Navigate to: | Network --> Trunks --> Trunk List, then click the    |
|              | "+" button to create a new Trunk                     |
+--------------+------------------------------------------------------+

-  .. image:: ./images/image1.png
      :width: 4.07431in
      :height: 3.14792in

..

   

-  Provide a Trunk Name, and move the respective Available interface to
   the "Members" section.

-  Click Repeat to define your next trunk.

   -  When you define the last trunk, you may select the "Finished"
      button

..

   

-  Example - Internal Trunk:

   -  .. image:: ./images/image2.png
         :width: 5.36111in
         :height: 3.39792in

..

   

-  .. image:: ./images/image3.png
      :width: 5.12986in
      :height: 3.25in

..

   

-  External Trunk:

   -  .. image:: ./images/image4.png
         :width: 5.13889in
         :height: 3.25in

..

   

-  HA Trunk:

   -  .. image:: ./images/image5.png
         :width: 5.44444in
         :height: 3.37986in

..

   

-  View of Trunk List after creating all three Trunks:

   -  .. image:: ./images/image6.png
         :width: 7.25in
         :height: 2.57431in

..

   

   

=================== =======================
.. rubric:: Task 2: **Create BIG-IP VLANs**
   :name: task-2    
=================== =======================

..

   

   On both BIG-IP devices, configure VLANs under the Network
   configuration section.

   

   Use the following table to create & define your three VLANs:

   

=========== ======= ============= ===========
**Name**    **Tag** **Interface** **Tagging**
=========== ======= ============= ===========
int_vlan_10 10      int_trunk     Untagged
ext_vlan_20 20      ext_trunk     Untagged
HA_vlan_30  30      HA_trunk      Untagged
=========== ======= ============= ===========

..

   

-  

+--------------+------------------------------------------------------+
| Navigate to: | Network --> VLANs --> VLAN List, then click the "+"  |
|              | button to create a new VLAN                          |
+--------------+------------------------------------------------------+

-  .. image:: ./images/image7.png
      :width: 4.11111in
      :height: 3.57431in

..

   

-  Create the respective VLANs per the table above.

..

   

-  Internal VLAN:

   -  .. image:: ./images/image8.png
         :width: 7.11111in
         :height: 3.95347in

..

   

-  .. image:: ./images/image9.png
      :width: 6.70347in
      :height: 6.32431in



-  External VLAN:

   -  .. image:: ./images/image10.png
         :width: 6.41667in
         :height: 6.44444in

..

   

-  HA VLAN:

   -  .. image:: ./images/image11.png
         :width: 6.41667in
         :height: 6.45347in

..

   

-  View of the VLAN List after all VLANs have been defined, and
   associated to their respective Trunk:

   -  .. image:: ./images/image12.png
         :width: 7.37986in
         :height: 2.66667in




-

=================== ==========================
.. rubric:: Task 3: **Create BIG-IP Self IPs**
   :name: task-3    
=================== ==========================

..

   

   On both BIG-IP devices, configure their respective Self IPs under the
   Network configuration section.

   

   Use the following table to create & define your three Self IPs:

   

+----------+----------+----------+----------+----------+----------+
| **       | **Name** | **IP     | **N      | **VLAN** | **Port   |
| BIG-IP** |          | A        | etmask** |          | Lo       |
|          |          | ddress** |          |          | ckdown** |
+==========+==========+==========+==========+==========+==========+
| bigipA   | sel      | 10.      | 255.2    | int      | Allow    |
|          | f_vlan10 | 1.10.241 | 55.255.0 | _vlan_10 | None     |
|          |          |          |          |          | (        |
|          |          |          |          |          | default) |
+----------+----------+----------+----------+----------+----------+
| bigipA   | sel      | 10.      | 255.2    | ext      | Allow    |
|          | f_vlan20 | 1.20.241 | 55.255.0 | _vlan_20 | None     |
|          |          |          |          |          | (        |
|          |          |          |          |          | default) |
+----------+----------+----------+----------+----------+----------+
| bigipA   | self_h   | 10.      | 255.2    | HA       | Allow    |
|          | a_vlan30 | 1.30.241 | 55.255.0 | _vlan_30 | None     |
|          |          |          |          |          | (        |
|          |          |          |          |          | default) |
+----------+----------+----------+----------+----------+----------+
| bigipB   | sel      | 10.      | 255.2    | int      | Allow    |
|          | f_vlan10 | 1.10.242 | 55.255.0 | _vlan_10 | None     |
|          |          |          |          |          | (        |
|          |          |          |          |          | default) |
+----------+----------+----------+----------+----------+----------+
| bigipB   | sel      | 10.      | 255.2    | ext      | Allow    |
|          | f_vlan20 | 1.20.242 | 55.255.0 | _vlan_20 | None     |
|          |          |          |          |          | (        |
|          |          |          |          |          | default) |
+----------+----------+----------+----------+----------+----------+
| bigipB   | self_h   | 10.      | 255.2    | HA       | Allow    |
|          | a_vlan30 | 1.30.242 | 55.255.0 | _vlan_30 | None     |
|          |          |          |          |          | (        |
|          |          |          |          |          | default) |
+----------+----------+----------+----------+----------+----------+

.. _section-1:


-

-  

+--------------+------------------------------------------------------+
| Navigate to: | Network --> Self IPs, then click the "+" button to   |
|              | create a new Self IP                                 |
+--------------+------------------------------------------------------+

..

   

-  .. image:: ./images/image13.png
      :width: 2.56458in
      :height: 1.29653in



-  Create the respective Self IPs per the table above.

..

   

-  Self IP, VLAN 10:

   -  .. image:: ./images/image14.png
         :width: 6.55556in
         :height: 3.57431in

..

   

-  Self IP, VLAN 20:

   -  .. image:: ./images/image15.png
         :width: 6.41667in
         :height: 3.68542in

..

   

-  Self IP, HA VLAN 30:

   -  .. image:: ./images/image16.png
         :width: 6.59236in
         :height: 3.81458in



-  View of the Self IP List after all Self IPs have been defined:

   -  .. image:: ./images/image17.png
         :width: 7.37986in
         :height: 2.68542in





+---------------------+-----------------------------------------------+
| .. rubric:: Task 4: | **Define Device Service Cluster               |
|    :name: task-4    | High-Availability Settings**                  |
+---------------------+-----------------------------------------------+

..

   

-  

+--------------+--------------------------------------------------------------+
| Navigate to: | Device Management --> Devices --> click the (Self) hyperlink |
+--------------+--------------------------------------------------------------+

-  .. image:: ./images/image18.png
      :width: 9.89792in
      :height: 4.23125in

..

   

-  Click the "ConfigSync" banner:

   -  .. image:: ./images/image19.png
         :width: 5.11111in
         :height: 2.19444in



-  Under the Local Address drop-down, select the HA VLAN 30 address,
   then click the Update button:

   -  .. image:: ./images/image20.png
         :width: 5.96319in
         :height: 2.19444in



-  Click the "Failover Network" banner, then the "Add" button:

   -  .. image:: ./images/image21.png
         :width: 7.48125in
         :height: 3.24097in

..

   

-  From the New Failover Unicast Address drop-down, select the
   data-plane VLAN 10 address; click the "Repeat" button:

   -  .. image:: ./images/image22.png
         :width: 5.90764in
         :height: 2.07431in



-  From the New Failover Unicast Address drop-down, select the
   data-plane VLAN 20 address; click the "Finished" button:

   -  .. image:: ./images/image23.png
         :width: 5.57431in
         :height: 2in

..

   

-  View of the Failover Unicast Configuration:

   -  .. image:: ./images/image24.png
         :width: 7.34236in
         :height: 2.53681in





+---------------------+-----------------------------------------------+
| .. rubric:: Task 5: | .. rubric:: Configure & Verify Device Trust   |
|    :name: task-5    |    between BIG-IPs                            |
|                     |    :name:                                     |
|                     | configure-verify-device-trust-between-big-ips |
+---------------------+-----------------------------------------------+



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

-  .. image:: ./images/image31.png
      :width: 4.26875in
      :height: 2.39792in



-  Retrieve Device Credentials (Step 1 of 3):

   -  Fill in the respective form items for *bigipA.f5demo.com*, then
      click the *Retrieve Device Information* button:

      -  .. image:: ./images/image32.png
            :width: 3.86111in
            :height: 2.86111in

..

   

-  Verify Device Certificate (Step 2 of 3):

   -  Confirm the device certificate information, then click the *Device
      Certificate Matches* button:

      -  .. image:: ./images/image33.png
            :width: 8.76875in
            :height: 2.53681in

..

   

-  Add Device (Step 3 of 3):

   -  Verify the device name, and click the *Add Device* button:

   -  .. image:: ./images/image34.png
         :width: 5.26875in
         :height: 1.10208in

..

   

-  Verify *bigipA.f5demo.com*

   -  Navigate to: Device Management --> Device Trust --> Device Trust
      Members:

      -  .. image:: ./images/image35.png
            :width: 4.23125in
            :height: 2.36111in

..

   

-  Verify that *bigipB.f5demo.com* is shown in the Peer Device List:

   -  .. image:: ./images/image36.png
         :width: 9.65764in
         :height: 1.82431in



-  

+-----------+---------------------------------------------------------+
| Question: | Why are both BIG-IPs Active?                            |
+===========+=========================================================+
| Answer:   | There is no Device Group established between the        |
|           | BIG-IPs yet . . . See next Task                         |
+-----------+---------------------------------------------------------+





=================== ======================================
.. rubric:: Task 6: .. rubric:: Configure the Device Group
   :name: task-6       :name: configure-the-device-group
=================== ======================================



On *bigipA.f5demo.com*, set up the new Device Group that will be used by
both BIG-IP systems.



-  On *bigipA.f5demo.com*:

   -  

+--------------+------------------------------------------------------+
| Navigate to: | Device Management --> Device Groups page, and then   |
|              | click the "+" button:                                |
+--------------+------------------------------------------------------+

-  .. image:: ./images/image37.png
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

   

-  .. image:: ./images/image38.png
      :width: 6.86111in
      :height: 4.13889in

..

   

-  .. image:: ./images/image39.png
      :width: 6.46319in
      :height: 3.99097in



=================== ============================================
.. rubric:: Task 7: .. rubric:: Validate the Device Group Status
   :name: task-7       :name: validate-the-device-group-status
=================== ============================================

.. _section-2:


-

In this task, you will observe the current Active/Standby HA state.



-  Observe the state of each BIG-IP after Device Group creation:

   -  BIG-IP-A:

      -  .. image:: ./images/image40.png
            :width: 4.57431in
            :height: 1.19444in

..

   

-  BIP-IP-B:

   -  .. image:: ./images/image41.png
         :width: 4.57431in
         :height: 1.09236in

..

   

-  Review the Device Management Overview screen.

   -  Attempt the "Recommendation action", and "Sync."

      -  **Is it successful?**

         -  .. image:: ./images/image42.png
               :width: 16.04653in
               :height: 6.13889in



-  Review the Overview status screen.

   -  BIG-IP-A:

      -  .. image:: ./images/image43.png
            :width: 5.52778in
            :height: 3.85208in

   -  BIG-IP-B:

      -  .. image:: ./images/image44.png
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





=================== ========================================
.. rubric:: Task 8: .. rubric:: Modify Self IP Port Lockdown
   :name: task-8       :name: modify-self-ip-port-lockdown
=================== ========================================



On each BIG-IP:



-  

+--------------+--------------------------------------------------------------+
| Navigate to: | Network --> Self IPs --> click the VLAN 30 Self IP hyperlink |
+--------------+--------------------------------------------------------------+

..

   

-  .. image:: ./images/image45.png
      :width: 7.82431in
      :height: 6.11111in

.. _section-3:


-

-  Under the Port Lockdown drop-down, change the value to "Allow
   Default," then click the Update button:

   -  .. image:: ./images/image46.png
         :width: 6.40764in
         :height: 4.37986in

.. _section-4:


-

-  

+-----------------------+---------------------------------------------+
| .. rubric:: Question: | Are the devices Active / Standby ? Why or   |
|    :name: question    | why not?                                    |
+=======================+=============================================+
| Answer:               | No, still Active / Active . . . Failover    |
|                       | via unicast Self IPs … port lockdown        |
+-----------------------+---------------------------------------------+

.. _section-5:


-

-  On both BIG-IPs, modify the Internal & External Self IP Port Lockdown
   settings

   -  Change from "Allow None" to "Allow Default"

..

   

-  

+-----------------------+---------------------------------------------+
| .. rubric:: Question: | Are the devices Active / Standby ? Why or   |
|    :name: question-1  | why not?                                    |
+=======================+=============================================+
| Answer:               | Yes, they are now Active / Standby for      |
|                       | Failover.                                   |
+-----------------------+---------------------------------------------+
| Log output:           | Apr 28 12:34:47 bigipB.f5demo.com info      |
|                       | sod[7297]: 010c0085:6: First failover       |
|                       | status message received from device         |
|                       | bigipA.f5demo.com (10.1.1.5) (unicast: ->   |
|                       | 10.1.20.242).                               |
|                       | Apr 28 12:34:47 bigipB.f5demo.com warning   |
|                       | sod[7297]: 010c0084:4: Failover status      |
|                       | message received after 3977.100 second gap, |
|                       | from device bigipA.f5demo.com (10.1.1.5)    |
|                       | (unicast: -> 10.1.20.242).                  |
+-----------------------+---------------------------------------------+

..

   

   

   

-  

+----------------------+----------------------+----------------------+
| .. rubric:: Bonus    | How do you know what |                     |
|    Question:         | "ports" are allowed  |                      |
|    :                 | in the "default"     |                      |
| name: bonus-question | port-lockdown list?  |                      |
+======================+======================+======================+
| **Answer:**          | GUI: Help, expand    | | CLI:               |
|                      | Port Lockdown, and   | |                   |
|                      | review:              |                      |
|                      |                      | tmsh list net        |
|                      |                     | self-allow           |
|                      |                      |                      |
|                      | .. image::           |                     |
|                      |  vertopal_ef3ad3b065 |                      |
|                      | ac47a0834351436faa7c | | net self-allow {   |
|                      | 57/media/image47.png | | defaults {         |
|                      |    :width: 2.37986in | | igmp:any           |
|                      |                      | | ospf:any           |
|                      |   :height: 9.10208in | | pim:any            |
|                      |                      | | tcp:domain         |
|                      |                      | | tcp:f5-iquery      |
|                      |                      | | tcp:https          |
|                      |                      | | tcp:snmp           |
|                      |                      | | tcp:ssh            |
|                      |                      | | udp:520            |
|                      |                      | | udp:cap            |
|                      |                      | | udp:domain         |
|                      |                      | | udp:f5-iquery      |
|                      |                      | | udp:snmp           |
|                      |                      | | }                  |
|                      |                      | | }                  |
+----------------------+----------------------+----------------------+





+---------------------+-----------------------------------------------+
| .. rubric:: Task 9: | .. rubric:: Verify an Active / Standby "In    |
|    :name: task-9    |    Sync" State                                |
|                     |                                               |
|                     | :name: verify-an-active-standby-in-sync-state |
+---------------------+-----------------------------------------------+

.. _section-6:


-

On each BIG-IP, review the current state.



-  BIG-IP-A:

   -  .. image:: ./images/image48.png
         :width: 4.87014in
         :height: 1.25in

-  BIG-IP-B:

   -  .. image:: ./images/image49.png
         :width: 4.62986in
         :height: 1.24097in

.. _section-7:


-

-  If your BIG-IPs are in a state other than the above, please review
   previous steps to ensure correct configuration.

..

   

.. _section-8:


-

==================== ===========================================
.. rubric:: Task 10: **Create an LTM Node Configuration Object**
   :name: task-10    
==================== ===========================================



In this task, we will create an LTM node object that will be
synchronized between BIG-IPs. This will validate that ConfigSync is
working correctly. We will perform these changes on the ACTIVE BIG-IP,
and then we will sync these changes to the STANDBY BIG-IP.



On the ACTIVE BIG-IP:



-  

+--------------+------------------------------------------------------+
| Navigate to: | Local Traffic --> Nodes --> Node List --> click the  |
|              | "+" sign to create a new node                        |
+--------------+------------------------------------------------------+

..

   

-  .. image:: ./images/image50.png
      :width: 4.12986in
      :height: 3.56458in

.. _section-9:


-

-  Create the following Windows Server Node Object, and click Finished:

   -  Name = win_server

   -  Address = 10.1.10.199

      -  .. image:: ./images/image51.png
            :width: 6.48125in
            :height: 4.12986in



+----------------------+----------------------------------------------+
| .. rubric:: Task 10: | **Perform a Configuration Synchronization    |
|    :name: task-10-1  | between BIG-IPs**                            |
+----------------------+----------------------------------------------+

.. _section-10:


~

On the ACTIVE BIG-IP,

   

-  Notice the "Changes Pending" in the upper-left corner:

   -  .. image:: ./images/image52.png
         :width: 4.5in
         :height: 1.20347in

   -  Click this hyperlink to go to the Overview screen.

   -  Review the recommendations, and perform a ConfigSync to peer:

      -  .. image:: ./images/image53.png
            :width: 7.94444in
            :height: 6.87986in

.. _section-11:


-

-  While the configuration is being pushed, you will see a "Syncing"
   icon display in the middle:

   -  .. image:: ./images/image54.png
         :width: 9.64792in
         :height: 1.37986in

.. _section-12:


-

-  Once the ConfigSync process is complete, your BIG-IPs should indicate
   an "In Sync" state, and be in an Active / Standby cluster:

   -  Verify the sync state:

      -  .. image:: ./images/image55.png
            :width: 7.32431in
            :height: 4.14792in

.. _section-13:


-

.. _section-14:


-

==================== =======================================
.. rubric:: Task 11: **Test & Validate a Failover Scenario**
   :name: task-11    
==================== =======================================



In this task, we will simulate a "link down" failure on the ACTIVE
BIG-IP, and determine if the BIG-IP will failover automatically.



We will compare the failover timing once we create & use our HA Group
configuration design.



BONUS: From the BIG-IP CLI, perform a "follow" of the */var/log/ltm* log
to see logging data in real-time:

-  Access the "WEB SHELL" from UDF for each BIG-IP:

   -  .. image:: ./images/image56.png
         :width: 3.82431in
         :height: 2.76875in

..

   

-  Run the following command at shell prompt:

   -  *tail -f /var/log/ltm*

      -  BIG-IP-A:

         -  .. image:: ./images/image57.png
               :width: 9.65764in
               :height: 0.62014in

      -  BIG-IP-B:

         -  .. image:: ./images/image58.png
               :width: 9.35208in
               :height: 0.77778in

.. _section-15:


~

-  

+----------------------------------+----------------------------------+
| .. rubric:: From the *ACTIVE*    | Network --> Interfaces --> place |
|    BIG-IP, navigate to:          | a checkmark next to 1.1 & click  |
|    :name: fr                     | the "Disable" button             |
| om-the-active-big-ip-navigate-to |                                  |
+----------------------------------+----------------------------------+

..

   

-  .. image:: ./images/image59.png
      :width: 10.23125in
      :height: 6.08333in



-  

+-------------+-------------------------------------------------------+
| Question:   | Did the BIG-IP failover? Why or why not?              |
+=============+=======================================================+
| Answer:     | No, the BIG-IP did not failover, as the interface     |
|             | objects are not part of the HA configuration.         |
+-------------+-------------------------------------------------------+
| Log Output: | Apr 28 15:13:47 bigipB.f5demo.com info lacpd[7293]:   |
|             | 01160016:6: Interface 1.1, link admin status:         |
|             | disabled, link status: up, duplex mode: full, lacp    |
|             | operation state: down                                 |
|             | Apr 28 15:13:47 bigipB.f5demo.com info lacpd[7293]:   |
|             | 01160010:6: Link 1.1 removed from aggregation         |
|             | Apr 28 15:13:47 bigipB.f5demo.com notice mcpd[4745]:  |
|             | 01bb0003:5: Trunk: int_trunk is DOWN                  |
|             | Apr 28 15:13:47 bigipB.f5demo.com notice mcpd[4745]:  |
|             | 01b5004a:5: Link: 1.1 is DISABLED                     |
|             | Apr 28 15:13:50 bigipB.f5demo.com warning sod[7297]:  |
|             | 010c0083:4: No failover status messages received for  |
|             | 3.100 seconds, from device bigipA.f5demo.com          |
|             | (10.1.1.5) (unicast: -> 10.1.10.242).                 |
+-------------+-------------------------------------------------------+



   

-  Now, from the *ACTIVE* BIG-IP, Disable the 1.2 Interface.

   -  .. image:: ./images/image60.png
         :width: 7.57431in
         :height: 2.90764in



-  

+----------+----------------------------------------------------------+
| Question | Did the BIG-IPs failover? Are they Active/Standby?       |
+==========+==========================================================+
| Answer   | No failover. Both BIG-IPs in an Active / Active state    |
+----------+----------------------------------------------------------+
| Logs:    | Apr 28 15:19:38 bigipB.f5demo.com info lacpd[7293]:      |
|          | 01160016:6: Interface 1.2, link admin status: disabled,  |
|          | link status: up, duplex mode: full, lacp operation       |
|          | state: down                                              |
|          | Apr 28 15:19:38 bigipB.f5demo.com info lacpd[7293]:      |
|          | 01160010:6: Link 1.2 removed from aggregation            |
|          | Apr 28 15:19:38 bigipB.f5demo.com notice mcpd[4745]:     |
|          | 01bb0003:5: Trunk: ext_trunk is DOWN                     |
|          | Apr 28 15:19:38 bigipB.f5demo.com notice mcpd[4745]:     |
|          | 01b5004a:5: Link: 1.2 is DISABLED                        |
|          | **Apr 28 15:19:41 bigipB.f5demo.com warning sod[7297]:   |
|          | 010c0083:4: No failover status messages received for     |
|          | 3.100 seconds, from device bigipA.f5demo.com (10.1.1.5)  |
|          | (unicast: -> 10.1.20.242).                               |
|          | Apr 28 15:19:41 bigipB.f5demo.com notice sod[7297]:      |
|          | 010c007e:5: Not receiving status updates from peer       |
|          | device bigipA.f5demo.com (10.1.1.5) (Disconnected).**    |
+----------+----------------------------------------------------------+

..

   

-  .. image:: ./images/image61.png
      :width: 21.12014in
      :height: 6.57431in

.. _section-16:


-

.. _section-17:


-

==================== ===========================================
.. rubric:: Task 11: **Enable both Interfaces on ACTIVE BIG-IP**
   :name: task-11-1  
==================== ===========================================



-  

+--------------+------------------------------------------------------+
| Navigate to: | Network --> Interfaces --> place checkmarks in 1.1 & |
|              | 1.2, then click the "Enable" button:                 |
+--------------+------------------------------------------------------+

-  .. image:: ./images/image62.png
      :width: 5.27778in
      :height: 5.71319in



.. _section-18:


-

==================== ====================
.. rubric:: Task 12: **Create HA Groups**
   :name: task-12    
==================== ====================



NOTES:

-  HA group configuration is device specific and is not synced between
   members of a DSC group.

   -  You must create a separate HA group on every device in the device
      group for this traffic group.



-  On each BIG-IP, create an HA Group:

   -  

+--------------+------------------------------------------------------+
| Navigate to: | System --> High Availability --> HA Group List -->   |
|              | click the "+" button                                 |
+--------------+------------------------------------------------------+

..

   

-  .. image:: ./images/image63.png
      :width: 5.49097in
      :height: 4.42569in

.. _section-19:


~

-  Add an HA Group Name; example: bigip-b_ha-group

   -  .. image:: ./images/image64.png
         :width: 4.28681in
         :height: 4.77778in

.. _section-20:


-

-  Next, we will add our Trunk links to our HA Group Configuration:

   -  .. image:: ./images/image65.png
         :width: 7.66667in
         :height: 3.23125in

..

   

-  Add the External Trunk object, and click the "Add" button; repeat
   this step for the Internal Trunk object:

   -  .. image:: ./images/image66.png
         :width: 5.74097in
         :height: 3.33333in

.. _section-21:


-

-  Verify that your HA Group Trunk configuration reflects the following:

   -  .. image:: ./images/image67.png
         :width: 10.79653in
         :height: 2.07431in

.. _section-22:


-

-  Verify your HA Group Configuration; once complete, click the "Create
   HA Group" button:

   -  .. image:: ./images/image68.png
         :width: 10.66667in
         :height: 4.83333in

.. _section-23:


-

-  After HA Group Creation, you should be presented with the following
   screen, showing your HA Group configuration object:

   -  .. image:: ./images/image69.png
         :width: 6.45347in
         :height: 1.99097in

.. _section-24:


-

+----------------------+------------------------------------------------------+
| .. rubric:: Task 13: | **Enable HA Group as the Preferred Failover Method** |
|    :name: task-13    |                                                      |
+----------------------+------------------------------------------------------+



-  

+--------------------------------+------------------------------------+
| On Active BIG-IP, Navigate to: | Device Management --> Traffic      |
|                                | Groups --> *traffic-group-1*       |
|                                | hyperlink                          |
+--------------------------------+------------------------------------+

..

   

-  .. image:: ./images/image70.png
      :width: 6.33333in
      :height: 2.5in

..

   

-  From the "Health Monitor" Section, select the drop-down for "HA
   Group:"

   -  .. image:: ./images/image71.png
         :width: 7.61111in
         :height: 7.28681in

..

   

-  Select our HA Group we created previously:

   -  .. image:: ./images/image72.png
         :width: 3.87014in
         :height: 0.97222in

..

   

-  Under the Failover Configuration section, select the radio button for
   "Failover to Device With Best HA Score," then click the "**Save**"
   button:

   -  .. image:: ./images/image73.png
         :width: 6.12014in
         :height: 5.85208in

.. _section-25:


~

-  Review the Traffic Groups screen, and verify the Failover Method is
   set to HA Score:

   -  .. image:: ./images/image74.png
         :width: 7.85208in
         :height: 3.28681in

.. _section-26:


-

-  Perform this same procedure to setup the HA Group configuration on
   *bigipB.f5demo.com*





=================== =================================================
.. rubric:: Task X: **Disable an Interface to force HA Group action**
   :name: task-x    
=================== =================================================



In this task, we will manipulate an interface on the Active BIG-IP. We
will disable the interface, and observe the BIG-IP behavior during
failover. We will use our logs to review, and compare the messages to
the manual "force to standby" method.



-  

+--------------------------------+------------------------------------+
| On Active BIG-IP, Navigate to: | Network --> Interfaces --> place a |
|                                | checkmark next to 1.1, and click   |
|                                | the Disable button                 |
+--------------------------------+------------------------------------+

..

   

-  .. image:: ./images/image75.png
      :width: 9.56458in
      :height: 4.37986in

-  Observe the BIG-IP status in the upper-left corner:

   -  .. image:: ./images/image76.png
         :width: 2.19444in
         :height: 0.96319in

..

   

-  .. image:: ./images/image77.png
      :width: 1.94444in
      :height: 1.01875in



-  Observe the logs from each BIG-IP:

   -  BIG-IP-A:

      -  .. image:: ./images/image78.png
            :width: 14.63889in
            :height: 2.17569in

..

   

-  .. image:: ./images/image79.png
      :width: 11.25903in
      :height: 3.15764in

..

   

-  BIG-IP-B:

   -  .. image:: ./images/image80.png
         :width: 9.62986in
         :height: 1.46319in

.. _section-27:


~

-  .. image:: ./images/image81.png
      :width: 9.01875in
      :height: 1.90764in

.. _section-28:


-

=================== =================================================
.. rubric:: Task X: **Modify additional HA Group Objects & Validate**
   :name: task-x-1  
=================== =================================================



How do the HA Group configuration Objects affect the HA Group Score?

.. _section-29:


-

-  Review the current HA Group Score from CLI with the command *tmsh
   show sys ha-group:*

   -  **BIG-IP A:**

      -  .. image:: ./images/image82.png
            :width: 5.24097in
            :height: 1.53681in

   -  **BIP-IP B:**

      -  .. image:: ./images/image83.png
            :width: 5.34236in
            :height: 1.58333in

..

   

   



-  What state are your BIG-IPs in?



+---------------------+-----------------------------------------------+
| .. rubric:: Task X: | **Disable the Internal Interface to force HA  |
|    :name: task-x-2  | Group Failover**                              |
+---------------------+-----------------------------------------------+



On the ACTIVE BIG-IP, disable the 1.2 Interface to simulate a failed
physical connection to the switch.



-  Navigate to Network --> Interfaces --> Interface List

   -  .. image:: ./images/image84.png
         :width: 4.59236in
         :height: 1.83333in





-  Place a checkmark next to "1.2" and Click the "**Disable**" button:

   -  .. image:: ./images/image85.png
         :width: 12.40764in
         :height: 2.44444in

.. _section-30:


~

.. _section-31:


~

-  Did a failover occur? What did the logs indicate? What happens to our
   HA state?

   -  **BIG-IP-A:**

      -  .. image:: ./images/image86.png
            :width: 15.24097in
            :height: 3.51875in

..

   

-  .. image:: ./images/image87.png
      :width: 2.07431in
      :height: 1.10208in



-  **BIG-IP-B:**

   -  .. image:: ./images/image88.png
         :width: 15.24097in
         :height: 6.78681in

..

   

-  .. image:: ./images/image89.png
      :width: 2.06458in
      :height: 1.12986in











Resources:
----------

-  `K16947: F5 recommended practices for the HA group
   feature <https://support.f5.com/csp/article/K16947>`__

-  `Manual : BIG-IP Device Service Clustering:
   Administration <https://techdocs.f5.com/en-us/bigip-14-1-0/big-ip-device-service-clustering-administration-14-1-0.html>`__

-  `Manual Chapter : Creating an Active-Standby Configuration using the
   Configuration
   Utility <https://techdocs.f5.com/en-us/bigip-14-1-0/big-ip-device-service-clustering-administration-14-1-0/creating-an-active-standby-configuration-using-the-configuration-utility.html>`__

-  `K13946: Troubleshooting ConfigSync and device service clustering
   issues <https://support.f5.com/csp/article/K13946>`__

   -  

+---------+-----------------------------------------------------------+
| *Ports* | *Device group members should be able to communicate over  |
|         | ports 443, 4353, 1026 (UDP), and 22 (recommended).*       |
|         |                                                           |
|         | *BIG-IP ASM requires the following additional Policy Sync |
|         | TCP ports: 6123-6128*                                     |
+---------+-----------------------------------------------------------+

..

   

.. _section-32:


-

Supplemental Resources:
-----------------------

-  `K41983050: The HA group score displayed with tmsh shows that the sys
   ha-group command no longer includes the active
   bonus <https://support.f5.com/csp/article/K41983050>`__

-  `K64414880: The Configuration utility may calculate and display an
   incorrect HA group
   score <https://support.f5.com/csp/article/K64414880>`__



Appendix:
---------



-  *tmsh cheat sheet:*

=================== ========================
.. rubric:: Task 1: **Create BIG-IP Trunks**
   :name: task-1-1  
=================== ========================

..

   *tmsh create net trunk int_trunk interfaces add { 1.1 }*

   *tmsh create net trunk ext_trunk interfaces add { 1.2 }*

   *tmsh create net trunk HA_trunk interfaces add { 1.3 }*

   

   

=================== =======================
.. rubric:: Task 2: **Create BIG-IP VLANs**
   :name: task-2-1  
=================== =======================

..

   *tmsh create net vlan int_vlan_10 tag 10 interfaces add { int_trunk
   }*

   *tmsh create net vlan ext_vlan_20 tag 20 interfaces add { ext_trunk
   }*

   *tmsh create net vlan HA_vlan_30 tag 30 interfaces add { HA_trunk }*

.. _section-33:


~

.. _section-34:


~

=================== ==========================
.. rubric:: Task 3: **Create BIG-IP Self IPs**
   :name: task-3-1  
=================== ==========================

..

   *tmsh create net self self_vlan10 address 10.1.10.242/24 vlan
   int_vlan_10*

   *tmsh create net self self_vlan20 address 10.1.20.242/24 vlan
   ext_vlan_20*

   *tmsh create net self self_ha_vlan30 address 10.1.30.242/24 vlan
   HA_vlan_30*

   

   

+---------------------+-----------------------------------------------+
| .. rubric:: Task 4: | **Define Device Service Cluster               |
|    :name: task-4-1  | High-Availability Settings**                  |
+---------------------+-----------------------------------------------+

..

   *tmsh modify cm device bigip1 configsync-ip 10.1.30.242*

   *tmsh modify cm device bigip1 unicast-address { { ip 10.1.10.242 } {
   ip 10.1.20.242 } }*
