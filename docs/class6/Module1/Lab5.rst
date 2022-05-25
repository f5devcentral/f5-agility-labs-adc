Lab 5:  Create & Enable HA Groups
=================================

Lab Tasks:
==========
* Task 1: Create HA Groups
* Task 2: Enable HA Group as the Preferred Failover Method

**Task 1: Create HA Groups**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^


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



**Task 2: Enable HA Group as the Preferred Failover Method**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

Lab Summary
^^^^^^^^^^^
In this lab, you enhanced your HA configuration to leverage HA Groups.  HA Groups provide the ability to monitor an "object" and take action upon and event.  After completion of these lab tasks, your BIG-IP should be leveraging HA Groups for failover.  In the following lab, we will test & validate how this configuration enhances the failover timing.

This completes Lab 5.