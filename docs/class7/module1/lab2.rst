Lab 2: Work with SNAT, Profiles, and Monitors
---------------------------------------------

In this lab you will gain experience using SNAT Auto Map for inbound
requests as well as outbound requests from internal users. You’ll also
use an HTTP and stream profile to make global modifications to text
within a web site. Finally you’ll see how using health monitors ensures
that the BIG-IP knows which web servers are available for client
requests.  SNATs, or Secure Network Address Translation, provide a mapping between nodes, often internal devices
and a SNAT address. SNATs can be configured many different ways including
one-to-one mappings, many-to-one mappings, or all to one mappings.  In all cases
a SNAT must be enabled on the VLAN where the node's traffic arrives on the BIG-IP system.
In this lab we will focus on the SNAT Automap feature which automatically maps the source
address of an allowed host to an address from a defined group.   Often times this is a Self-IP
address on the BIG-IP.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Task 1** – Use SNAT Auto Map

A virtual server configured on a BIG-IP system typically translates the destination IP address
of an incoming packet to load balance requests.  By default the source IP address remains unchanged.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. In the Configuration Utility, open the **Pool List** page.

#. For this lab we will use the existing **LAMP** pool which contains the below members.
   Make sure load balancing method is Round Robin.

   +---------------+------------------------------------+
   | Form field    | Value                              |
   +===============+====================================+
   | Name          | LAMP                               |
   +---------------+------------------------------------+
   | LAMP_Server1  | Address: 10.1.20.11                |
   |               | Service Port: 80                   |
   +---------------+------------------------------------+
   | LAMP_Server2  | Address: 10.1.20.12                |
   |               | Service Port: 80                   |
   +---------------+------------------------------------+
   | LAMP_Server3  | Address: 10.1.20.13                |
   |               | Service Port: 80                   |
   +---------------+------------------------------------+
   | LAMP_Server4  | Address: 10.1.20.14                |
   |               | Service Port: 80                   |
   +---------------+------------------------------------+

#. Open the **Virtual Server List** page and click on the **LAMP** virtual server


#. From the Windows Jump Host open putty, and then connect to **BIGIP01 10.1.1.4** and log
   in as **root** / **admin.F5demo.com**.

   |image7|

#. At the CLI type (or copy and paste):

   ``tcpdump -i external port 80``

#. Open a second putty session and connect to **BIGIP02 10.1.1.6**.

#. At the CLI type (or copy and paste):

   ``tcpdump -i internal port 80``

#. Use a new tab to access **http://10.1.10.200**, and then close the
   tab.

   The page displays as expected.

#. Examine the **tcpdump** in windows.

   On the external VLAN the communication is between the client IP
   address (**10.1.10.190**) and the virtual server (**10.1.10.200**).

   On the internal VLAN the communication is between the client IP
   address (**10.1.10.190**) and a back-end web server (**10.1.20.x**).

#. In both **tcpdump** sessions press the **Enter** key several times to
   move the log entries to the top of the window.

#. On the internal VLAN the requests are from the client IP address to a
   back-end web server, however there are no responses from the web
   server.

#. Press the **Enter** key several times to move the log entries to the
   top of the window.

#. In the Configuration Utility you should be in the **LAMP** virtual server configuration.

#. From the **Source Address Translation** list select **Auto Map** which has already
   been configured when the virtual was created.  Notice there are several options to include
   SNAT, Auto Map, and None.

   |image8|

#. Use an incognito window to access **http://10.1.10.200**, and then
   close the window.

   SNAT Auto Map ensures that responses to server request are always sent
   back to the BIG-IP system.

#. Examine the **tcpdump** window.

   On the external VLAN the communication is still between the client IP
   address (**10.1.10.190**) and the virtual server (**10.1.10.200**).

   On the internal VLAN the communication is now between the BIG-IP
   internal self IP address (**10.1.20.245**) and a back-end web
   server (**10.1.20.x**).

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Task 2** – Create a SNAT for Internal Resources

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Press the **Enter** key several times to move the log entries to the
   top of the window.

#. On the Windows server, change the default gateway to **10.1.20.245**
   (the BIG-IP internal floating self IP address).

#. On the Windows server, use Internet Explorer to access
   **www.f5.com**.

   The request fails as the internal resource has no access to the WAN (or
   the Internet).

#. Close the page, then on the Windows desktop examine the **tcpdump**
   windows.

   No requests are being sent to the Internet by the BIG-IP system on
   behalf of the internal resource.

#. In the Configuration Utility, open the **Local Traffic > Address
   Translation > SNAT List** page and click **Create**.

#. Use the following information for the new SNAT, and then click
   **Finished**.

   +-------------------------+--------------------------------+
   | Form field              | Value                          |
   +=========================+================================+
   | Name                    | internal\_snat                 |
   +-------------------------+--------------------------------+
   | Translation             | IP Address: 10.1.10.245        |
   +-------------------------+--------------------------------+
   | Origin                  | Address List                   |
   +-------------------------+--------------------------------+
   | Address/Prefix Length   | 10.1.20.0/24 (Click **Add**)   |
   +-------------------------+--------------------------------+

#. On the Windows server, use Internet Explorer to access
   **www.f5.com**.

   The internal user now has public access to the internet using the SNAT
   IP address of **10.1.10.100**.

#. On the Windows desktop, examine the **tcpdump** windows.

   On the external VLAN the communication is between the SNAT IP address
   (**10.1.10.100**) and the Internet resources.

   On the internal VLAN the communication is between the internal client
   (**10.1.20.251**) and the Internet resources.

#. Close the **putty** sessions.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Task 3** – Use Profiles with a Virtual Server

Profiles are a powerful configuration tool providing an easy way to define
traffic policies and apply those policies across virtual servers.   Through
a profile you can also change a setting for traffic across many different
applications.

Profiles provide

- A centralized place to define specific traffic behavior such as compression, SSL,
  and authentication that can be applied to multiple virtual servers.

- A centralized place to change any setting and have them applied to all applications
  using an existing profile.  A profile tells a virtual server how to process packets
  it receives through the BIG-IP system.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. From the Jump Host use a new tab to access **http://10.1.10.200**, and then select the
   **Stream Profile Example** link toward the bottom of the page and examine the text.

   Instead of updating all the web site code we’ll use profiles on the BIG-IP system to update the web site.

#. Close the tab.

#. In the Configuration Utility, open the **Local Traffic > Profiles >
   Other > Stream** page and click **Create**.

#. Use the following information for the profile, and then click
   **Finished**.

   +--------------+---------------------+
   | Form field   | Value               |
   +==============+=====================+
   | Name         | name\_change        |
   +--------------+---------------------+
   | Source       | Lorax Bank          |
   +--------------+---------------------+
   | Target       | Seuss Bank          |
   +--------------+---------------------+

#. Open the **Virtual Server List** page and click **LAMP**.

#. From the **Configuration** list select **Advanced**.

#. From the **HTTP Profile** list select **http**.

   |image9|

#. From the **Stream Profile** list select **name\_change**.

#. In the **Acceleration** section, from the **HTTP Compression
   Profile** list select **httpcompression**.

#. From the **Web Acceleration Profile** list select
   **optimized-caching**, and then click **Update**.

#. Use an incognito window to access **http://10.1.10.200**, and then
   select the **Stream Profile Example** link toward the bottom of the page and examine the text.

   Although the logo need to be updated, all the text on the page now
   references **Seuss Bank**.

#. Remove the **Stream Profile**, **Web Acceleration Profile**, and **Acceleration** profile
   from the **LAMP** Virtual Server and update.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Task 4** – Work with Monitors

During this section of the lab we will review many of the available
monitors and how to customize them.  The BIG-IP system includes a set of
pre-defined monitor templates for address, service, content, and interactive checks.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. From the Windows Jump host open a new tab in a browser and enter the
   following URL **http://10.1.10.200/HealthCheck.html**

   We’re going to use this web page to identify if the web server is up or down.

#. Close the health check page.

#. In the Configuration Utility, open the **Local Traffic > Monitors**
   page and click **Create**.

#. Use the following information for the monitor, and then click
   **Finished**.

   +--------------------------+---------------------------------+
   | Form field               | Value                           |
   +==========================+=================================+
   | Name                     | LAMP_monitor                    |
   +--------------------------+---------------------------------+
   | Type                     | http                            |
   +--------------------------+---------------------------------+
   | Interval                 | 4                               |
   +--------------------------+---------------------------------+
   | Timeout                  | 13                              |
   +--------------------------+---------------------------------+
   | Send String              | GET /HealthCheck.html\\r\\n     |
   +--------------------------+---------------------------------+
   | Receive String           | SERVER\_UP                      |
   +--------------------------+---------------------------------+
   | Receive Disable String   | SERVER\_DOWN                    |
   +--------------------------+---------------------------------+

#. Open the **Pool List** page and click **LAMP**.

#. Identify the current **Availability** status of the pool.

   Unknown identifies when a pool or node doesn’t have a configured
   monitor.

#. Add **LAMP\_monitor** to the **Active** list and click **Update**.

   The **Availability** of the pool changes to **Available (Enabled)**.

#. Open the **Local Traffic > Nodes > Node List** page.

   Notice that all the nodes currently display unknown.

#. Open the **Local Traffic > Nodes > Default Monitor** page.

#. Add **gateway\_icmp** to the **Active** list and click **Update**.

#. Return to the **Nodes >ode List** page.

   All nodes now display. This means that they are all sending **icmp**
   responses.

#. Open the **Local Traffic > Network Map** page and view the status for
   **LAMP**.

   The virtual server, pool, and all three pool members display available.

#. Use your mouse to hover over the pool members.

   All two nodes also display available.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Sub-Task 1** – Take Lamp_Server1 Offline

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. From the Windows Jump Host while logged into BIG-IP01 click on **Pools**,
   then click on **LAMP** pool, click the **Members** tab then click on
   the **LAMP_Server1** then click on the **Disabled** radio button.

#. In the Configuration Utility under **Local Traffic** open the
   **Network Map**.

   |image10|

#. Use your mouse to hover over the pool members.

   The first pool member is offline, while the other pool members are available.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Sub-Task 2** – Disable all pool members

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. From the Windows Jump Host while logged into BIG-IP01 click on **Pools**,
   click on **LAMP** pool, click the **Members** tab, click
   the **check box** to select all members and then click on the **Disable** radio button.

#. In the Configuration Utility under **Local Traffic** open the
   **Network Map**.

#. Notice that the virtual server and pool display unavailable.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Sub- Task 3** – Bring all pool members back online

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. From the Windows Jump Host while logged into BIG-IP01 click on **Pools**,
   click on **LAMP** pool, click the **Members** tab, click
   the **check box** to select all members and then click on the **Enable** radio button.

#. Use an incognito window to access **http://10.1.10.200**.

This concludes Lab 2 and a basic overview of Secure Network Address Translation
(SNAT), Profiles, and Monitors.  Please begin Lab 3.


 .. |image7| image:: images/image7.PNG
    :width: 3.32107in
    :height: 0.33645in

 ..  |image8| image:: images/image8.PNG
    :width: 3.32107in
    :height: 0.33645in

 ..  |image9| image:: images/image9.PNG
    :width: 1.32107in
    :height: 0.33645in

 ..  |image10| image:: images/image10.PNG
    :width: 1.32107in
    :height: 0.33645in
