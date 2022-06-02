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

+--------------+------------------------------------------------------+
| Navigate to: | System --> High Availability --> HA Group List -->   |
|              | click the "+" button                                 |
+--------------+------------------------------------------------------+

  .. image:: ../images/image63.png
      

Add an HA Group Name; example: bigip-b_ha-group

   .. image:: ../images/image64.png

Next, we will add our External Gateway Pool we created earlier to the Pools HA Group Configuration:
   
Click the "Add" button

   .. image:: ../images/image131.png

In the Pool drop-down, select "ext_gw_pool," and click the "Add" button:
       
   .. image:: ../images/image132.png


Next, we will add our Trunk links to our HA Group Configuration:

Under the Trunks section, click the "Add" button:
      
   .. image:: ../images/image65.png
   

From the drop-down, select the the External Trunk object (ext_trunk), and click the "Add" button:
  
   .. image:: ../images/image133.png
   
Repeat this step for the Internal Trunk object (int_trunk):
      
   .. image:: ../images/image134.png

Verify that your HA Group Trunk configuration reflects the following, and Click the "Create HA Group" button:

   .. image:: ../images/image67.png

After HA Group Creation, you should be presented with the following screen, showing your HA Group configuration object:

   .. image:: ../images/image69.png

**Task 2: Enable HA Group as the Preferred Failover Method**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+--------------------------------+------------------------------------+
| On Active BIG-IP, Navigate to: | Device Management --> Traffic      |
|                                | Groups --> *traffic-group-1*       |
|                                | hyperlink                          |
+--------------------------------+------------------------------------+

  .. image:: ../images/image70.png

From the "Health Monitor" Section, select the drop-down for "HA Group:"

  .. image:: ../images/image71.png

Select our HA Group we created previously:

  .. image:: ../images/image72.png
         :width: 3.87014in
         :height: 0.97222in


Under the Failover Configuration section, select the radio button for "Failover to Device With Best HA Score," then click the "**Save**" button:

  .. image:: ../images/image73.png
         :width: 6.12014in
         :height: 5.85208in


Review the Traffic Groups screen, and verify the Failover Method is set to HA Score:

  .. image:: ../images/image74.png
         :width: 7.85208in
         :height: 3.28681in


Perform this same procedure to setup the HA Group configuration on **bigipB.f5demo.com**

Lab Summary
^^^^^^^^^^^
In this lab, you enhanced your HA configuration to leverage HA Groups.  
HA Groups provide the ability to monitor an "object" and take action upon and event.  
After completion of these lab tasks, your BIG-IP should be leveraging HA Groups for failover.  In the following lab, we will test & validate how this configuration enhances the failover timing.

This completes Lab 5.