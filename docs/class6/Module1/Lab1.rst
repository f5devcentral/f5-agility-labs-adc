Lab 1: Configure BIG-IP Trunks, VLAN's, and Self-IP's
-----------------------------------------------------

Lab Tasks:
**********

* Task 1: Create BIG-IP Trunks
* Task 2: Create BIG-IP VLANs
* Task 3: Create BIG-IP Self IPs

Task 1: Create BIG-IP Trunks
============================

#. On both BIG-IP devices, configure trunks under the Network configuration section.

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

#. **Navigate to**: Network > Trunks > Trunk List, then click the "+" button to create a new Trunk

   .. image:: ../images/image1.png

#. Provide a Trunk Name, and move the respective Available interface to the "Members" section.

#. Click Repeat to define your next trunk.

   When you define the last trunk, you may select the "Finished" button

   -  Internal Trunk:
   
    .. image:: ../images/image2.png


    .. image:: ../images/image3.png

   -  External Trunk:

     .. image:: ../images/image4.png

   -  HA Trunk:

     .. image:: ../images/image5.png

   -  View of Trunk List after creating all three trunks:

     .. image:: ../images/image6.png


Task 2: Create BIG-IP VLANs
===========================

#. On both BIG-IP devices, configure VLANs under the Network configuration section.

   Use the following table to create & define your three VLANs:

   +------------+----+-----------+----------+
   |Name        |Tag |Interface  | Tagging  |
   +============+====+===========+==========+
   |int_vlan_10 | 10 |int_trunk  | Untagged |
   +------------+----+-----------+----------+
   |ext_vlan_20 | 20 |ext_trunk  | Untagged |
   +------------+----+-----------+----------+
   |HA_vlan_30  | 30 |HA_trunk   | Untagged |
   +------------+----+-----------+----------+

#. **Navigate to**: Network > VLANs > VLAN List, then click the "+" button to create a new VLAN

     .. image:: ../images/image7.png

#. Create the respective VLANs per the table above.

   -  Internal VLAN:

     .. image:: ../images/image8.png

     .. image:: ../images/image9.png

   -  External VLAN:

     .. image:: ../images/image10.png

   -  HA VLAN:

     .. image:: ../images/image11.png

   -  View of the VLAN List after all VLANs have been defined, and associated to their respective Trunk:

     .. image:: ../images/image12.png

Task 3: Create BIG-IP Self IPs
==============================
#. On both BIG-IP devices, configure their respective Self IPs under the Network configuration section.

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
        - self_vlan10
        - 10.1.10.241
        - 255.255.255.0
        - int_vlan_10
        - Allow None (default)
      * - bigipA
        - self_vlan20
        - 10.1.20.241
        - 255.255.255.0
        - ext_vlan_20
        - Allow None (default)
      * - bigipA
        - self_vlan30
        - 10.1.30.241
        - 255.255.255.0
        - HA_vlan_30
        - Allow None (default)
      * - bigipB
        - self_vlan10
        - 10.1.10.242
        - 255.255.255.0
        - int_vlan_10
        - Allow None (default)
      * - bigipB
        - self_vlan20
        - 10.1.20.242
        - 255.255.255.0
        - ext_vlan_20
        - Allow None (default)
      * - bigipB
        - self_vlan30
        - 10.1.30.242
        - 255.255.255.0
        - HA_vlan_30
        - Allow None (default)

On BIG-IP A, create the following Floating Self IP Objects.  These will be shared configuration objects that will be synced in Lab 3.  Only create the Floating Self IPs on BIG-IP-A:

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


#. **Navigate to**: Network > Self IPs, then click the "+" button to create a new Self IP

     .. image:: ../images/image13.png

#. Create the respective Self IPs per the table above.

   -  Self IP, VLAN 10:

     .. image:: ../images/image14.png

   -  Self IP, VLAN 20:

     .. image:: ../images/image15.png

   -  Self IP, HA VLAN 30:

     .. image:: ../images/image16.png

   -  View of the Self IP List after all Self IPs have been defined:

     .. image:: ../images/image17.png


Lab Summary
^^^^^^^^^^^
In this lab, you setup basic BIG-IP network configuration settings.  After completion of these lab tasks, you should have network connectivity between the devices on all VLANs.  These configuration objects will assist with the following labs.

This completes Lab 1.