Lab 3: Review Base TCP Profiles
===============================

In Lab 3, you will look at the base TCP profiles within TMOS and some legacy TCP profiles that may exxist within your BIG-IP environments.   options for reviewing the application flows through the BIGIP at the HTTP and TCP layers.  With more details about the traffic, adjustments can be made to improve the speed of the data through the BIGIP.

v16+ TCP Profiles - https://my.f5.com/manage/s/article/K50411377
v15 TCP Profiles - https://my.f5.com/manage/s/article/K74767112
v15+ F5-TCP_WAN Profile - https://my.f5.com/manage/s/article/K10281257
v15+ F5-TCP_LAN Profile - https://my.f5.com/manage/s/article/K37412295
v10-12 TCP-WAN-OPTIMIZED - https://my.f5.com/manage/s/article/K7405
v10-12 TCP-LAN-OPTIMIZED - https://my.f5.com/manage/s/article/K7406


Task 1:
*******

Review genearal TCP profiles available to TMOS.

#. From the left-side menu, go to Local Traffic > Profiles > Protocol > TCP.
#. Click the **Parent Profile** column title to sort the profiles

  Most profiles in TMOS have a parent/child structure (or from CLI - defaults-from structure).  Within the list of TCP profiles, you can see that all profiles source from the base profile named tcp. 

.. image:: ../images/tcp_profiles_sorted.png

  As TMOS has upgraded over the years, changes have been made to the base TCP profile and to maintain compatability with previous relases, new child profiles have been created to override the base profile with the setting of the older profiles <<reword??>>

* Click on the tcp-legacy profile to see how options are overridden from the TCP parent profile.  The key option carried over from the older TCP profile is the Memory Management Send Buffer limit of 65535 bytes.  This is the 16-bit Window size limit from the original TCP standard (RFC 793).

.. image:: ../images/tcp_legacy_buffers.png

  If tcp-wan-optimized/tcp-lan-optimized profiles are in use, they are based on the older tcp-legacy parent profile with the small send buffer.  This small does not allow for TCP Window Scaling.

* Connect to the Ubuntu-Client via SSh using the Access dropdown

.. image:: ../images/udf_slient_ssh.png

* Click 'open terminal' if prompted
  
.. image:: ../images/udf_open_terminal.png

* Type 'yes' in response to the fingerprint prompt
  
 .. image:: ../images/udf_fingerprint.png

* Connect to BIGIP01 via SSH using the Access dropdown of the component and follow the same prompts as with the Ubuntu-Client
* Start a packet capture from the SSH window of BIGIP01::

What is this command doing?

  timeout 5s: Run the command for 5s then quit
  tcpdump:  Command to run
  -nni: No name resolution and No part resolution - just return the raw numbers
  internal: The 'interface' name - the server-side VLAN in the lab
  host 10.1.10.15:  The internal floating selfIP used as the source filter
  tcp[14:2] == 0:  Bytes 14 and 15 of the TCP header showing TCP window size - we want zero
  tcp[13] == 16: Filtering on TCP ACK as TCP Zero can also be seen with FIN during connection close
  -s 500: We only concerned with TCP flags so the snaplength is 500 Bytes

timeout 5s tcpdump -nni internal host 10.1.10.15 and 'tcp[14:2] == 0 && tcp[13] == 16' -s 500

  Since we have background traffic running through BIGIP01, you should see 500-700 packets during the 5s capture.

Samples::

09:51:20.983280 IP 10.1.10.15.36658 > 10.1.10.32.443: Flags [.], ack 1114961, win 0, options [nop,nop,TS val 1824155031 ecr 3982370403], length 0
09:51:20.984960 IP 10.1.10.15.56662 > 10.1.10.30.443: Flags [.], ack 1050717, win 0, options [nop,nop,TS val 1824155032 ecr 909863226], length 0
09:51:20.991289 IP 10.1.10.15.36528 > 10.1.10.32.443: Flags [.], ack 1552257, win 0, options [nop,nop,TS val 1824155038 ecr 3982370620], length 0
09:51:20.996657 IP 10.1.10.15.33414 > 10.1.10.32.443: Flags [.], ack 1560945, win 0, options [nop,nop,TS val 1824155043 ecr 3982370624], length 0
09:51:20.999251 IP 10.1.10.15.36820 > 10.1.10.32.443: Flags [.], ack 458642, win 0, options [nop,nop,TS val 1824155046 ecr 3982370421], length 0

The capture output shows the SelfIP (10.1.10.15) of BIGIP01 sending a TCP Zero Window to the application pool members (10.1.10.30-34).  This means BIGIP01 is telling the pool members to stop sending data while it waits for the the client-side to catchup.  As mentioned earlier, the client had 200ms of latency injected in the path to BIGIP01.

From the BIGIP01 SSH window run the following packet capture filtering in SYN and SYN/ACK packets ('tcp[13] & 2 != 0') from the client.:

  tcpdump -nni external host 10.1.30.6 and 'tcp[13] & 2 != 0' -c 4

The output will be similar to this::

  11:08:34.619362 IP 10.1.30.6.44976 > 10.1.20.103.443: Flags [S], seq 1493890342, win 64240, options [mss 1460,sackOK,TS val 141834751 ecr 0,nop,wscale 7], length 0 in slot1/tmm1 lis= port=1.2 trunk=
  11:08:34.619417 IP 10.1.20.103.443 > 10.1.30.6.44976: Flags [S.], seq 4100418120, ack 1493890343, win 4380, options [mss 1460,sackOK,TS val 1828788666 ecr 141834751], length 0 out slot1/tmm1 lis=/Common/web01_vs1 port=1.2 trunk=

The client (10.1.30.6) is advertising TCP Window Scale capability with option 'wscale 7'.  BIGIP01 is responding without a wscale option since the TCP buffers sizes are limited to TCP base maximums of 65535 Bytes. 


Go the the UI of BIGIP01 and change the TCP profiles of web01_vs1 to F5-tcp-progressive and click the Update button at the bottom of the page to save changes.

 .. image:: ../images/tcp_progressive.png

Go back to the BIGIP01 SSH window and run the same packet capture.  You should see a reduction in TCP Zero Window packets - maybe 300-450 packets in 5s.  This is an improvement but not the best for the traffic in the lab.  As we saw during Lab 2, there is a mix of 16kB-3MB files going through the system.  The f5-tcp-progressive TCP profile adjusts the TCP buffers based on connection conditions.  It uses bandwidth delay product to calculate the TCP buffers.  see https://my.f5.com/manage/s/article/K15800216 for more information.  Also see https://my.f5.com/manage/s/article/K3422 for content spooling.  In the lab, there is very low latency on the server-side connetions (pool members) and higher latency on the client-side connections.  With most HTTP traffic the client is requesting something from the server (GET) and the server is replaying withh the requested data.  The f5-tcp-progressive TCP profile is not adequately setting the buffers because the sending side is high bandwidth with low latency.

From the BIGIP01 SSH window run the following packet capture filtering in SYN and SYN/ACK packets ('tcp[13] & 2 != 0') from the client.:

  tcpdump -nni external host 10.1.30.6 and 'tcp[13] & 2 != 0' -c 4

The output will be similar to this::

  11:14:45.819337 IP 10.1.30.6.36460 > 10.1.20.103.443: Flags [S], seq 991761495, win 64240, options [mss 1460,sackOK,TS val 142205951 ecr 0,nop,wscale 7], length 0 in slot1/tmm0 lis= port=1.2 trunk=
  11:14:45.819391 IP 10.1.20.103.443 > 10.1.30.6.36460: Flags [S.], seq 2970736692, ack 991761496, win 23360, options [mss 1460,nop,wscale 9,sackOK,TS val 1829159866 ecr 142205951], length 0 out slot1/tmm0 lis=/Common/web01_vs1 port=1.2 trunk=

Here the client (10.1.30.6) is advertising wscale 7 and BIGIP01 is responding with wscale 9 so the TCP Window can scale up to the buffer limits determined by the f5-prgressive TCP profile. 

Go back to the BIGIP01 UI and change webo1_vs1 to use a custom TCP profile configure for the lab with 3MB (tcp_3mb) buffers and click update at the bottom of the page to save the change.  Higher buffers will use more memory for each TCP connection  while f5-tcp-progressive can also use more CPU as it calculates the buffer sizes.
 
 .. image:: ../images/tcp_progressive.png

Go back to the BIGIP01 SSH window and run the same packet capture.  You should no longer see any TCP Zero Window packets.  TMOS is now able to take in the server response as fast as possible and hold the data in memory until the client acknowleges receipt.  (reword??)

  timeout 5s tcpdump -nni internal host 10.1.10.15 and 'tcp[14:2] == 0 && tcp[13] == 16' -s 500
  tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
  listening on internal, link-type EN10MB (Ethernet), capture size 500 bytes

  0 packets captured
  0 packets received by filter
  0 packets dropped by kernel

How does TCP Zero Window affect traffic performance?  Using a 2nd Virtual Server on BIGIP01

* Task 1: Review AVR Data
* Task 2: Use iRules to log traffic data
* Task 3: Anything else?

Task 1: Review AVR Data
-----------------------

AVR is already provisioned with custom HTTP and TCP AVR profiles created and assigned to Virtual Servers.  When AVR profiles are initially applied to Virtual Servers, it takes around 5 minutes for the data to appear in the logs.  In the lab, the AVR profiles are already assigned and trafficc is running in the backgound to pre-populate the logs.

#. Connect to TMUI of BIGIP01 using the following credentials:

    User:     admin
    Password: admin.F5demo.com
    
#. From the left menu, select Statistics > Analytics > HTTP > Overview

  .. image:: ../images/avr_http_selection.png

    You will see the AVR overview graphs showing HTTP data flowing through the system.  On the right side of the screen, there are filtering options that we will use later.  By default, the view shows data from the last hour for all Virtual Servers with an attached HTTP AVR profile and refreshes every 5 minutes.  Feel free to change the visible time frame using the dropdown near the top left.  Since the lab systems have been running off for a few weeks you will have data for the Last week, Last month but may not see much for the Last day. 

  .. image:: ../images/avr_http_overview.png

#. Use AVR to see what is happening on the system at the HTTP layer.

    Reset the view to Last hour from the top left dropdown.  Use the Response Code filter on the right side to see what cades are available.  With the lab traffic you will have at least 200, 301, and 404 and response codes available.  Let's use the filters to see what is happening beyond 200 responses.  Click on 301 and the graphs will update showing just the data related to 301 response codes.  Click on URLs above on the right to see what URLs are causing the 301 (redirects).

  .. image:: ../images/avr_http_responses.png

    For lab, 301 responses are a result of redirect.html.  Redirects may be necessary or could be left over from a previous application migration.  You can take these results back to the application team(s) to see if redirects are expected.  It could be something as simple as clients requesting /login instead of /login/ that is causing the redirects.  Unneccesary redirects add time to the client connections and uses resources on the BIGIP and/or application servers.

    Go back to the Resonse Codes filter and click 301 to deselect it, then click on 404.  Look at the URLs filter, it will show the cause of the 404's - file_64kd.txt.  In the lab this cause is simple, the URL should be /file_64k**b**.txt not k**d**.txt.  In a production environment, you may see many other URLs resulting in 404s.  They could be simple typos from the client or from bad links in the applications.  The 404 filter may also show you evidence of someone scanning your application(s).  The point of this is learn how to see what is happening with application traffic.

  .. image:: ../images/avr_http_typo.png



------



<<Keep for code examples>>
#. **Navigate to**: Network > Trunks > Trunk List, then click the "+" button to create a new Trunk:

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

In Task 2, we will define our VLANs on our BIG-IPs.  Our VLANs will be associated with their respective trunk from Task 1.

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

#. **Navigate to**: Network > VLANs > VLAN List, then click the "+" button to create a new VLAN:

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

In Task 3, we will configure our Local Self IPs of each BIG-IP.  These IPs will be our L3 connectivity to our BIG-IP networks.

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


#. **Navigate to**: Network > Self IPs, then click the "+" button to create a new Self IP:

     .. image:: ../images/image13.png

#. Create the respective Self IPs per the table above.

   -  Self IP, VLAN 10:

     .. image:: ../images/image14.png

   -  Self IP, VLAN 20:

     .. image:: ../images/image15.png

   -  Self IP, HA VLAN 30:

     .. image:: ../images/image16.png

   -  Example view of the Self IP List from BIG-IP-A after all Self IPs have been defined:

     .. image:: ../images/image17.png


Lab Summary
***********
In this lab, you setup basic BIG-IP network-level configuration settings.  After completion of these lab tasks, you should have network connectivity between the devices on all VLANs.  These configuration objects will assist with the subsequent labs.

This completes Lab 1.