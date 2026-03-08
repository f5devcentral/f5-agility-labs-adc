Lab 1: The Basics (Networking, Pools and Virtual Servers)
=========================================================

In this lab we will access the Management Graphical User Interface (GUI). We will then create the
VLANs, and assign self IP addresses to our VLAN. As mentioned during
our lecture portion, BIG-IPs may be put in-line or one-armed depending
on your customer’s requirements and topology.

Log on to the **bigip01** TMUI (GUI) Interface:
  - **UDF** - Under Components, select the **Access** drop-down and then **TMUI**

Navigate the BIG-IP Configuration Utility (GUI)
-----------------------------------------------

The BIG-IP Configuration Utility (GUI) is a web-based interface that provides a point-and-click interface for managing the BIG-IP system.

You can perform various tasks such as:

* Building and managing traffic objects (e.g., virtual servers, pools, nodes).
* Configuring modules (LTM, DNS/GTM, Advanced WAF/ASM, APM, AFM).
* Managing system/network settings (e.g., VLANs, routes).
* Viewing statistics, health checks, and logs.

The following sections are important to guide you through the process of creating VLANs, self IPs, pools, and virtual servers using the BIG-IP GUI.

* **Main**/Help/About
* **Local Traffic** >> Virtual servers, Profiles, Pools, Monitors
* **Device Management** >> Devices, Device Trust, Traffic Groups
* **Network** >> interfaces, self IPs, Vlans
* **System** >> Configuration, Disk Management, Software Management, License, Resource Provisioning, Platform, High Availability

Creating VLANs
--------------

First, you will need to log into **bigip01**. You will need create two untagged VLANs, one client-side VLAN (**client_vlan**) and one server-side VLAN (**server_vlan)** for the devices in your network.

1. From the sidebar select **Network** **> VLANs** then select
   **Create**

   .. image:: ../images/image10.png
      :width: 800px


a. Under **General Properties**:

   i. **Name**: client_vlan

b. The name is for management purposes only, you could name them after
   your children or pets

   i. **Tag**: <leave blank>

      1. Entering a tag is only required for “\ **Tagged**\ ” (802.1q)
         interfaces, “\ **Untagged**\ ” interfaces will automatically
         get a tag which is used for internal L2 segmentation of
         traffic.

c. Under **Resources** in the **Interfaces** section:

   i. **Interface**: 1.1

   ii. **Tagging**: Untagged

   iii. Select the **Add** button. Leave all other items at the default
        setting.

      .. image:: ../images/image11.png
          :width: 600px


d. When you have completed your VLAN configuration, hit the **Finished** button

   .. important::
      Now create another untagged VLAN named **server_vlan** on interface **1.2,** following the same steps as above.


   When you are done, you should have two untagged VLANs, one on interface 1.1 and one on interface 1.2.

   .. image:: ../images/image11.1.png
       :width: 950px


Creating Self IPs
-----------------

1. Go to **Network > Self IPs**, select **Create**.

   .. image:: ../images/image12.png
      :width: 800px


   a. Create a new self IP, for the **server_vlan** and **client_vlan** VLANs. In **Network >> Self IPs >> New Self IP**, under **Configuration** enter:

      +---------------+---------------+---------------+
      |    Option     |  Server-side  |  Client-side  |
      +===============+===============+===============+
      | Name          | server_ip     | client_ip     |
      +---------------+---------------+---------------+
      | IP Address    | 10.1.20.245   | 10.1.10.245   |
      +---------------+---------------+---------------+
      | Netmask       | 255.255.255.0 | 255.255.255.0 |
      +---------------+---------------+---------------+
      | VLAN          | server_vlan   | client_vlan   |
      +---------------+---------------+---------------+
      | Port Lockdown | Allow None    | Allow None    |
      +---------------+---------------+---------------+


.. caution::
   A common mistake is to forgot to change the VLAN/Tunnel selection to the appropriate VLAN.  Make sure your self IPs are in the appropriate VLAN. 

   - The default **Port Lockdown** setting of **Allow None** means the Self IP only responds to ICMP requests.
   - The **Allow Defaults** selection opens the following on the self IP of the VLAN.
      - TCP: ssh, domain, snmp, https
      - TCP: 4353, 6699 (for F5 protocols, such as HA and iQuery)
      - UDP: 520, cap, domain, f5-iquery, snmp
      - PROTOCOL: ospf

   b. When you have completed your selfIP configuration, hit the **Finished** button. You should have something similar to the following:

      .. image:: ../images/image14.png
          :width: 800px


Assigning the Default Gateway
-----------------------------

1. Go to **Network > Routes** and then **Add**.

   a. Here is where we assign our default gateway (and other static
      routes as desired)

      .. image:: ../images/image15.png
         :width: 600px


   b. Under **Properties**

      i. **Name**: def_gw
      ii. **Destination**: 0.0.0.0
      iii. **Netmask**: 0.0.0.0
      iv. **Resource**: Use Gateway…
      v. **Gateway** **Address**: 10.1.10.1
      vi. When you have completed defining your default gateway, hit the **Finished** button

2. Verify your network configuration

   a. Use SSH or WebShell (UDF Only) to access your BIG-IP.

      i.  Ping your default gateway, 10.1.10.1

      ii. Ping a web server at 10.1.20.11.


Creating Pools
--------------
In this lab we will build a pool and virtual server to support our web
site and verify our configurations by accessing our web servers through
the BIG-IP. Verification will be performed visually and through various
statistical interfaces.

1. From the sidebar select **Local Traffic >>** **Pools** then select
   **Create**. Here we will create our new pool

   .. image:: ../images/image16.png
      :width: 600px

   a. Under **Configuration**:

      i. **Name**: www_pool (The name is for management purposes only, no spaces can be used)
      ii. **Description**: <optional>
      iii. **Health** **Monitor**: http

   b. Under **Resources**

      i. **Load Balancing Method**: <leave at the default Round Robin>
      ii. **Priority Group Activation**: <leave at default>
      iii. **New Members**:

         +-------------+------------------+
         | **Address** | **Service Port** |
         +=============+==================+
         | 10.1.20.11  | 80               |
         +-------------+------------------+
         | 10.1.20.12  | 80               |
         +-------------+------------------+
         | 10.1.20.13  | 80               |
         +-------------+------------------+


2. As you enter each IP address and port combination hit **Add** button

   a. When you have complete your pool configuration, hit the **Finished** button

      .. image:: ../images/image17.png
         :width: 600px


Creating Virtual Servers
------------------------

1. Under **Local Traffic** select **Virtual Servers** then select **Create**.

   .. image:: ../images/image18.png
      :width: 600px


2. Under **General Properties**

   a. **Name:** www_vs
   b. **Description**: <optional>
   c. **Type:** Standard
   d. **Source/Address:** <leave blank>

      .. note:: The default is 0.0.0.0/0, all source IP address are allowed


   e. **Destination** **Address/Mask:** 10.1.10.100

      .. note:: The default mask is /32


   f. **Service Port**: 80 or HTTP

3. Under **Configurations**

   i.  The web servers do not use the BIG-IP LTM as the default gateway.
       This means return traffic will route around the BIG-IP LTM and
       the TCP handshake will fail. To prevent this we can configure
       SNAT Automap on the Virtual Server. This will translate the
       client IP to the self IP of the egress VLAN and ensure the
       response returns to the BIG-IP.

   ii. **Source Address Translation**: Auto Map

      .. image:: ../images/image19.png
          :width: 500px


3. Under **Resources**

   a. **iRules**: none
   b. **Policies**: none
   c. **Default Pool**: From the drop down menu, select the pool (**www_pool**) which you created earlier
   d. **Default Persistence Profile**: None
   e. **Fallback Persistence Profile**: None

4. When you have complete your pool configuration, hit the **Finished**
   button

5. You have now created a Virtual Server

   .. image:: ../images/image20.png
       :width: 900px
 

.. note:: Items in the GUI that are blue are links ie. shortcuts



Now let’s see if our virtual server works!
------------------------------------------

1. **UDF** - log into your **Ubuntu Jumpbox via xRDP access method** using the credentials provided in Documentation section

2. Open the browser from the bottom of the screen, go to the Virtual Server you just created http://10.1.10.100

3. Refresh the browser screen several times (use “<ctrl>” F5)

   .. image:: ../images/image21.png
       :width: 700px


4. Go to your BIG-IP and view the statistics for the **www_vs** virtual
   server and the **www_pool** pool and its associated members

5. Go to **Statistics >> Module Statistics >> Local Traffic**

6. Choose **Virtual Servers** from "Statistics Type" drop-down menu
   
   .. image:: ../images/image22.png
       :width: 700px


7. or, Go to **Local Traffic >> Virtual Servers >> Statistics**

8. Go to **Local Traffic >> Pools >> Statistics** to see pool member statistics

   * Did each pool member receive the same number of connections?
   * Did each pool member receive approximately the same number of bytes?
   * Note the Source and Destination address when you go to directly and through the virtual server

9. Let’s archive our configuration in case we have to fall back later.

   a. Go to **System >> Archives** and select **Create**.
   b. Name your archive **lab1_the_basics_net_pool_vs**


Extra Credit!
-------------

You can also review statistics via the CLI, simply SSH to the management
IP of your BIG-IP. Refer to your Student Information page and Network
Diagram for the IP address.

1. Check out the Linux CLI and TMSH

   a. Review the information of the following command:

      i. **export TERM=vt100; bigtop –n**

      ii. Type **q** to quit.

   b. Take a look at the TMOS CLI, type “\ **tmsh**\ ” to enter the
      Traffic Management Shell

      i.   (tmos)# **show ltm pool**

      ii.  (tmos)# **show ltm pool detail**

           1. show statistics from all pools

      iii. (tmos)# **show ltm virtual**

      iv.  (tmos)# **show ltm virtual detail**

           2. Show statistics of all virtual servers

2. Check out the Dashboard!

   a. Go to **Statistics >> Dashboard**

      .. image:: ../images/image23.png
         :width: 600px


   b. This will take you to the BIG-IP Dashboard page. The F5 **BIG-IP Dashboard** in the Configuration utility provides real-time, customizable graphical views of system performance, including CPU usage, memory, network traffic (PPS, Mbps), and connection statistics. It allows users to create custom "view sets" combining data from various modules (LTM, DNS, AFM) to monitor application health and traffic flow.
   
      .. image:: ../images/dashboard.png
         :width: 950px

   
   Feel free to explore the dashboard and create your own view sets to monitor the performance of your virtual server and pool!