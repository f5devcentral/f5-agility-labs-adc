Lab7: Persistance mirroring and connection mirroring
----------------------------------------------------

Task 1: Persistence mirroring Profile
=====================================

Persistance mirroring is used to share persistence information between BIG-IP's in a cluster.

It is per default not enabled and require to create a dedicated profile that has it enabled.

DO NOT edit the default profiles. Always create a new profile with the desired settings and use the default profile as parent profiles. 
Default profiles will be overwritten with the next software update.


Task 2: Create LTM Pool Configuration Objects 
=============================================

In this task, we will create LTM Virtual Server configuration objects that will be synchronized between BIG-IPs. This will validate that ConfigSync is working correctly. We will perform these changes on the ACTIVE BIG-IP, and then we will sync these changes to the STANDBY BIG-IP.

We will create two pool objects:

1.  Back-end Server Pool - this will be used as the pool for the Virtual Server object
2.  External Gateway Pool - this will be used in our HA Group, to validate the availability of our external gateway

**On the ACTIVE BIG-IP, Create Pool & Node Objects**

#. **Navigate to**: Local Traffic > Pools > Pool List > click the "+" sign to create a new pool

   .. image:: ../images/image114.png

#. Create the following Pool Configuration Objects:

   **Pool 1**
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
    -  **Name:**  test_http_vs
    -  **Type:**  Standard
    -  **Destination Address/Mask:**  10.1.10.55
    -  **Service Port:**  80 (HTTP)
    -  **HTTP Profile (Client):**  http
    -  **Resources:**
       -  **Default Pool:**  server_pool
       -  **Default Persistence Profile:**  source_addr
    - Click the "Finished" Button

     .. image:: ../images/image129.png
     .. image:: ../images/image130.png


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
