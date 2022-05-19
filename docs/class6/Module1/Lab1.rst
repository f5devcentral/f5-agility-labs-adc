Lab 1: Configure BIG-IP Trunks, VLAN's, and Self-IP's
=====================================

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
