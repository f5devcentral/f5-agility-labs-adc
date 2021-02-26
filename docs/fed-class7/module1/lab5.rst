Lab 5: High Availability
------------------------

BIG-IP systems are generally deployed in pairs since redundant systems add
reliability and management stability over a single system.  BIG-IP systems
typically are configured in Active-Standby mode.   This lab will familiarize
students with High-Availability.   High-Availability on the BIG-IP platform
is referred to as Device Service Clusters (DSC)

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Students will be configuring a High-Availability cluster using both BIGIP01
and BIGIP02 with BIGIP01 serving as the Active device in the DSC cluster.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Task 2** - Configure the failover networking

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. From the Windows 10 Jumphost log into BIGIP01 and BIGIP02

#. Configure the HA VLAN on both BIG-IP systems by navigation to the Network
   pane, clicking on VLANs, and click on create.

#. On BIGIP01 and BIGIP02 assign Network Interface 1.3 (untagged) to a VLAN called **HA**

#. The next step will be to configure a Self-IP for the HA VLAN.

#. Use the below table as a reference for Self-IP addresses on each BIG-IP:

   **Be sure to click Finished when complete**

   +--------------+--------------------------------+
   | Form field   | Value                          |
   +==============+================================+
   | Name         | ha_selfip                      |
   +--------------+--------------------------------+
   | IP Address   | 10.1.30.240                    |
   +--------------+--------------------------------+
   | Netmask      | 255.255.255.0                  |
   +--------------+--------------------------------+
   | VLAN/Tunnel  | HA                             |
   +--------------+--------------------------------+
   | Port Lockdown| Allow Default                  |
   +--------------+--------------------------------+


   |image20|


   +--------------+--------------------------------+
   | Form field   | Value                          |
   +==============+================================+
   | Name         | ha_selfip                      |
   +--------------+--------------------------------+
   | IP Address   | 10.1.30.241                    |
   +--------------+--------------------------------+
   | Netmask      | 255.255.255.0                  |
   +--------------+--------------------------------+
   | VLAN/Tunnel  | HA                             |
   +--------------+--------------------------------+
   | Port Lockdown| Allow Default                  |
   +--------------+--------------------------------+


   |image21|


#. On each BIG-IP configure the device object failover parameters the BIG-IP will send to other BIG-IP systems that are part of a sync-only or sync-failover group

#. Under Device Management, Devices, select the local BIG-IP **it will be notated with the (Self) suffix.**

#. Under Device Connectivity on the top bar select:

#. ConfigSync

#. Use the Self-IP Address of the HA VLAN for the local addresses

#. Failover Network

#. In the **Failover Unicast Configuration** section select the **Add** button

#. Use the Self-IP address of the HA VLAN for the **Address**

#. Leave the **Port** at the default setting of 1026

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Task 2** - Configure the Device Trust

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#.  On BIGIP01 from the **Device Management** pane, click on **Device Trust**
    > **Device Trust Members**, and select **Add**.

#.  On BIGIP01, enter 10.1.1.6 which is the Management address of BIGIP02
    and enter the admin User ID and Password.

#. Click **Retrieve Device Information**, click **Device Certificate Matches**, and then click **Add Device**.

#. On BIGIP02 verify that bigip01 is displayed as a trusted device under **Device Trust**.

|image22|


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Task 3** - Configure a Sync-Failover Group

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#.  On BIGIP01, Under the **Device Management** Navigation pane, click on
    **Device Group** and click **Create**
    
#. Create a new device Group named **syncfailover** with **Group Type** of Sync-Failover.

#. Add BIGIP01 and BIGIP02 to the **Includes** box.

#. Check the **Network Failover** setting for the group.

#. Click **Finished**.

#. Attempt to perform a **Manual** sync by clicking on the **Overview** section
   under the **Device Management** Navigation menu.

#. Ensure **bigip01.f5demo.com** is selected and then click **Sync**.

-  Was the full synchronization successful?

-  Are the configurations identical on both BIG-IP systems? 

-  Check the virtual servers on BIGIP02.


|image23|



^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Task 4** - Configure a Floating Traffic-Group

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The default Traffic-Group processes application traffic for the local device.
A Floating Traffic-Group contains the following objects:

- Virtual IP addresses
- NATs
- SNAT translation addresses
- Self IP Addresses
- Folders

#. From the **Device Management** Navigation pane, click on **Traffic Groups**

#. Click on the traffic group **traffic-group-1**

#. The **Failover Method** default should be set to
   **Failover using Preferred Device Order and then Load Aware**

#. Ensure BIGIP01 and BIGIP02 have been added to the **Preferred Order**
   window.

#. Click **Save** at the bottom.

#. Select the **Failover Objects** tab at the top to see the failover objects.

#. Click the **Changes Pending** at the top of the screen and then click **Sync**.

#. Verify you're able to access **https://10.1.10.200**

#. Force a failover by clicking **Device Management** > **Traffic Groups** > **traffic-group1** and then select **Force to Standby**

#. Verify you're still able to access **https://10.1.10.200**


.. |image20| image:: images/image20.PNG
   :width: 4.32107in
   :height: 1.33645in
.. |image21| image:: images/image21.PNG
   :width: 6.32107in
   :height: 2.33645in
.. |image22| image:: images/image22.PNG
   :width: 6.32107in
   :height: 5.33645in
.. |image23| image:: images/image23.PNG
      :width: 6.32107in
      :height: 5.33645in
