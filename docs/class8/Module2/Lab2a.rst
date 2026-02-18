Task 1: Review Base TCP Profiles
================================

Review general TCP profiles available to TMOS.

#. From the left-side menu, go to Local Traffic > Profiles > Protocol > TCP.
#. Click the **Parent Profile** column title to sort the profiles

  Most profiles in TMOS have a parent/child structure (or from CLI - defaults-from structure).  Within the list of TCP profiles, you can see that all profiles end up sourcing from the base profile named tcp. 

.. image:: ../images/tcp_profiles_sorted.png

  As TMOS has upgraded over the years, changes have been made to the base TCP profile and to maintain compatability with previous relases, new child profiles have been created to override the base profile with the settings of the older profiles <<reword??>>

* Click on the tcp-legacy profile to see how options are overridden from the TCP parent profile.  The key option carried over from the older TCP profile is the Memory Management Send Buffer limit of 65535 bytes.  This is the 16-bit Window size limit from the original TCP standard (RFC 793).

.. image:: ../images/tcp_legacy_buffers.png

* At this point, the web01-vs1 Virtual Server is using the older TCP profiles - tcp-wan-optimized (client-side) and tcp-lan-optimized (server-side).  These profiles are parented from tcp-legacy and have small TCP buffers that do not allow for TCP Window scaling.  These profiles are commonly assigned to Virtual Servers on BIG-IPs that have been upgraded from older versions of TMOS -  For exaample v10 > v12 > v14 > v15 > v17. 

* Connect to the Ubuntu-Client via SSh using the Access dropdown

.. image:: ../images/udf_slient_ssh.png

* Click 'open terminal' if prompted
  
.. image:: ../images/udf_open_terminal.png

* Type 'yes' in response to the fingerprint prompt
  
 .. image:: ../images/udf_fingerprint.png

* Connect to BIGIP01 via SSH using the Access dropdown of the component and follow the same prompts as with the Ubuntu-Client

  .. image:: ../images/udp_bigip01_ssh.png

* Click Open Terminal if prompted

  .. image:: ../images/udp_client_ssh.png

* Enter 'yes' if prompted for fingerprint

  .. image:: ../images/udf_fingerprint.png

* Start a packet capture from the SSH window of BIGIP01::

  What is this following TCPDUMP command doing?

    timeout 5s: Run the command for 5s then quit
    tcpdump:  Command to run
    -nni: No name resolution and No part resolution and interface
    internal: The 'interface' name - the server-side VLAN in the lab
    host 10.1.10.15:  The internal floating selfIP used as the source filter
    tcp[14:2] == 0:  Bytes 14 and 15 of the TCP header showing TCP window size - we want zero
    tcp[13] == 16: Filtering on TCP ACK as TCP Zero can also be seen with FIN during connection close
    -s 500: We only concerned with TCP flags so the snaplength is 500 Bytes

timeout 5s tcpdump -nni internal host 10.1.10.15 and 'tcp[14:2] == 0 && tcp[13] == 16' -s 500

  Since we have background traffic running through BIGIP01, you should see 500-700 packets during the 5s capture.

Log Examples::

09:51:20.983280 IP 10.1.10.15.36658 > 10.1.10.32.443: Flags [.], ack 1114961, win 0, options [nop,nop,TS val 1824155031 ecr 3982370403], length 0
09:51:20.984960 IP 10.1.10.15.56662 > 10.1.10.30.443: Flags [.], ack 1050717, win 0, options [nop,nop,TS val 1824155032 ecr 909863226], length 0
09:51:20.991289 IP 10.1.10.15.36528 > 10.1.10.32.443: Flags [.], ack 1552257, win 0, options [nop,nop,TS val 1824155038 ecr 3982370620], length 0
09:51:20.996657 IP 10.1.10.15.33414 > 10.1.10.32.443: Flags [.], ack 1560945, win 0, options [nop,nop,TS val 1824155043 ecr 3982370624], length 0
09:51:20.999251 IP 10.1.10.15.36820 > 10.1.10.32.443: Flags [.], ack 458642, win 0, options [nop,nop,TS val 1824155046 ecr 3982370421], length 0

The capture output shows the SelfIP (10.1.10.15) of BIGIP01 sending TCP Zero Window ACKs to the application pool members (10.1.10.30-34).  This means BIGIP01 is telling the pool members to stop sending data while it waits for the the client-side to catchup.  As mentioned earlier, the client had 200ms of latency injected in the path to BIGIP01.

From the BIGIP01 SSH window, run the following packet capture filtering on SYN and SYN/ACK packets ('tcp[13] & 2 != 0') from the client.:

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

