Lab 7: Persistence Mirroring and Connection Mirroring
-----------------------------------------------------

In Lab 7, we will continue to enhance & optimize the BIG-IP configuration.  We will specifically be enabling Virtual Server configuration, to enhance Connection Mirroring & Persistence Mirroring.

Lab Tasks:
**********
* Task 1: Persistence Mirroring Profile
* Task 2: Create LTM Pool Configuration Objects
* Task 3: Create LTM Virtual Server Configuration Object with connection mirroring enabled
* Task 4:  Perform a Configuration Synchronization between BIG-IPs

Task 1: Persistence Mirroring Profile
=====================================

In Task 1, we will create a new Persistence Profile, enabling persistence mirroring.

Persistance mirroring is used to share persistence information between BIG-IP's in a cluster.

By default, persistence mirroring is **NOT** enabled.  You will need to create a dedicated persistence profile that has this setting enabled.

**DO NOT** edit default BIG-IP profiles. Always create a new profile with the desired settings and use the default profile as parent profiles. 
Default profiles will be overwritten with the next software update.

#. **Navigate to**: Local Traffic > Profiles > Persistence, and click the "+" button to create a new profile:
   .. image:: ../images/image136.png

#. Provide a Name for your Custom Persistence Profile, i.e.:  *source_addr_mirror_persist*
    - Select Source Address Affinity from the Persistnece Type drop-down
    - Ensure the Parent Profile is set to *source_addr*
    - You will need to place a "checkmark" Under the Custom setting for Mirror Persistence:


      .. image:: ../images/image137.png

   - Place checkmarks in the Mirror Persistence field, and Click the "Save" button:


     .. image:: ../images/image138.png

   - You should see two Source Address profiles, one custom and one default:


     .. image:: ../images/image139.png


Task 2: Create LTM Pool Configuration Objects 
=============================================

In Task 2, we will create LTM Virtual Server configuration objects that will be synchronized between BIG-IPs. This will validate that ConfigSync is working correctly. We will perform these changes on the ACTIVE BIG-IP, and then we will sync these changes to the STANDBY BIG-IP.

We will create a pool object:

1.  Back-end Server Pool - this will be used as the pool for the Virtual Server object

**On the ACTIVE BIG-IP, Create Pool & Node Objects**

#. **Navigate to**: Local Traffic > Pools > Pool List > click the "+" sign to create a new pool

   .. image:: ../images/image114.png

#. Create the following Pool Configuration Objects:

Server Pool:
   -  **Name:** server_pool
   -  **Health Monitors:** gateway_icmp
     -  Within the Resources Section:
         -  **New Node Address:** 10.1.10.200   
         -  **Service Port:** \* All Services       
         - Click the Add button
 
        .. image:: ../images/image123.png

   - Click the "Finished" Button:
        .. image:: ../images/image135.png

After completion of this task, you should be presented with the following 2 pools:
      .. image:: ../images/image127.png

Task 3:  Create LTM Virtual Server Configuration Object with connection mirroring enabled
=========================================================================================
In this task, we will create a simple HTTP Virtual Server object.  This will be used to demonstrate the additional failover features you can apply at the Virtual Server level.

- Navigate to:  Local Traffic --> Virtual Servers --> Virtual Server List, then click the "+" sign:
    -   .. image:: ../images/image128.png
- Create the Virtual Server with the following settings:
General Properties:
    -  **Name:**  test_http_vs
    -  **Type:**  Standard
    -  **Destination Address/Mask:**  10.1.10.55
    -  **Service Port:**  80 (HTTP)
     .. image:: ../images/image129.png

From the Configuration section, at the Basic drop-down, select Advanced, and Configure the following settings:
     .. image:: ../images/image140.png
    -  **HTTP Profile (Client):**  http
    -  **Source Address Translation:**  From the drop-down, select AutoMap:
       .. image:: ../images/image148.png
    -  **Connection Mirroring:**  Place a checkmark on this setting
     .. image:: ../images/image141.png
     .. image:: ../images/image143.png
    - Under the  **Resources:** Section, Define the following settings, and Click the "Finished" Button:
       -  **Default Pool:**  server_pool
       -  **Default Persistence Profile:**  source_addr_mirror_persist
     .. image:: ../images/image142.png

You should be presented with the following Virtual Server after creation:
     .. image:: ../images/image149.png

Task 4:  Perform a Configuration Synchronization between BIG-IPs
================================================================

**On the ACTIVE BIG-IP**

#. Notice the "Changes Pending" in the upper-left corner

   .. image:: ../images/image52.png

#. Click this hyperlink to go to the Overview screen.

#. Review the recommendations, and perform a ConfigSync to peer

   .. image:: ../images/image53.png

#. While the configuration is being pushed, you will see a "Syncing" icon display in the middle:

   .. image:: ../images/image54.png

#. Once the ConfigSync process is complete, your BIG-IPs should indicate an "In Sync" state, and be in an Active / Standby cluster

#. Verify the sync state:

   .. image:: ../images/image55.png


Lab Summary
***********
In this lab, you enhanced your HA configuration to leverage HA Groups.  
HA Groups provide the ability to monitor an "object" and take action upon an event.  
After completion of these lab tasks, your BIG-IP should be leveraging HA Groups for Failover.  In the following lab, we will test & validate how this configuration enhances the failover timing.

This completes lab 7, and concludes the **BIG-IP HA Failover - Do it the Proper Way** lab.

We hope this lab experience was educational and beneficial.  If you have any feedback, or suggestions on making this better, please provide feedback.

Thank you, 
F5 Solutions Engineers