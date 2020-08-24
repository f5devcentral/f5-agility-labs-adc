Lab 2: Sync Only exercise
=========================

**Objective:** Add a sync only device group. You have already configured
two VE’s in an Active/Active Configuration with two traffic groups. Add
a 3\ :sup:`rd` VE. Create new Sync-Only group, and new Partition that
will leverage new Sync-Only group. Create new SSL profile which will
sync to all devices.

**Prerequisites and Notes**

Have at least 2 VE’s in an Active/Active Failover Configuration.

Note that for this exercise we will use three network interfaces (as in
in the previous failover exercise)

``1.1`` = External Network Interface (Wan Side)

``1.2`` = Internal network interface (LAN Side)

``1.3`` = High Availability Network Interface

TASK 1 – Add HA Self IP to bigip3.lab
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Go to **Network -> Self Ips -> Create**

   - Name: ``192.168.1.12``
   - IP Address: ``192.168.1.12``
   - Netmask: ``255.255.255.0``
   - VLAN: ``HA``
   - Port Lockdown: ``Allow Default``

#. Click **Finished**

   |image18|

TASK 2 – Update Config-Sync Properties on bigip3.lab
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Go to **Device Management -> Devices**

#. Click on then ``bigip3.lab (Self)`` link

#. Go to **ConfigSync** tab

#. Select ``192.168.1.12 (HA)`` in Local Address dropdown

#. Click **Update**

   |image19|

TASK 3 – Add bigip3 to peer list on bigip1
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Go to **Device Management -> Device Trust -> Device Trust Members -> Add**

   |image20|

#. Click on **Retrieve Device Information**

#. Click **Device Certificate Matches**

#. Click **Add Device**

   |image21|

#. Go to **bigip3** and verify **bigip1** and **bigip2** are now in the peer list

   |image22|

TASK 4 – Create New Sync Only Group
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**On bigip1:**

*Create a sync only group*

#. Go to **Device Management -> Device Groups -> Create**

   - Name = ``device_group_02_so``
   - Group Type = ``Sync-Only``
   - Members = All 3 bigip’s

   |image23|

#. Click **Finished**

*Perform initial sync*

#. Click **Awaiting Initial Sync** in the upper-left of the GUI

#. Choose ``device_group_02_so``, then choose ``bigip1``.

#. Select **Push** the selected device configuration to the group and then
   click **Sync**

   |image24|

TASK 5 – Create New Partition and SSL Profile, Configure for Sync-Only
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**On bigip1:**

*Create new Partition*

#. Go to **System -> Users -> Partition List -> Create**

   - Partition Name = ``partition_02_so``
   - Device Group = (uncheck "Inherit device group from root folder" box),
     ``device_group_02_so``
   - Traffic Group = ``None``

   |image25|

#. Click **Finished**

*Create new Client SSL Profile*

#. Go to **Local Traffic -> Profiles -> SSL -> Client**

#. Change Partition to ``partition_02_so`` in the upper-right of the GUI

   |image26|

#. Click **Create**

   - Name = ``clientssl_02_lab``
   - Accept all defaults

#. Click Finished

*Sync Changes*

#. On **bigip2** and **bigip3**, confirm this Sync-Only clientssl profile has
   synced

#. Go to **Local Traffic -> Profiles -> SSL –> Client**

#. Choose ``partition_02_so``

#. Is ``clientssl_02_lab`` there?

What are some practical uses for Sync-Only device groups?


.. |image18| image:: /_static/class2/image20.png
   :width: 6.33611in
   :height: 3.00080in
.. |image19| image:: /_static/class2/image21.png
   :width: 7.05000in
   :height: 2.45764in
.. |image20| image:: /_static/class2/image22.png
   :width: 6.58611in
   :height: 3.25283in
.. |image21| image:: /_static/class2/image23.png
   :width: 7.05000in
   :height: 3.69931in
.. |image22| image:: /_static/class2/image24.png
   :width: 7.05000in
   :height: 1.90833in
.. |image23| image:: /_static/class2/image25.png
   :width: 7.05000in
   :height: 2.71181in
.. |image24| image:: /_static/class2/image26.png
   :width: 7.05000in
   :height: 4.67917in
.. |image25| image:: /_static/class2/image27.png
   :width: 7.05000in
   :height: 4.12917in
.. |image26| image:: /_static/class2/image28.png
   :width: 7.05000in
   :height: 2.51597in