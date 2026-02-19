Task 2:  Modified TCP Profiles
==============================

1. Go the the UI of BIGIP01 and change the TCP profiles of web01_vs1 to F5-tcp-progressive and click the Update button at the bottom of the page to save changes.

   .. image:: ../images/tcp_progressive.png
       :width: 500px


2. Go back to the BIGIP01 SSH window and run the same packet capture from earlier::
  
    timeout 5s tcpdump -nni internal host 10.1.10.15 and 'tcp[14:2] == 0 && tcp[13] == 16' -s 500  
   

   You should see a reduction in TCP Zero Window packets - maybe 300-450 packets in 5s.  This is an improvement but not the best for the traffic in the lab.  As we saw during Lab 1, there is a mix of 16kB-3MB files going through the system.  The f5-tcp-progressive TCP profile adjusts the TCP buffers based on connection conditions.  It uses bandwidth delay product to calculate the TCP buffers.
   
      | See https://my.f5.com/manage/s/article/K15800216 for more information.
      | Also see https://my.f5.com/manage/s/article/K3422 for content spooling.
    
   In the lab, there is very low latency on the server-side connetions (pool members) and higher latency on the client-side connections.  With most HTTP traffic the client is requesting something from the server (GET) and the server is replaying withh the requested data.  The f5-tcp-progressive TCP profile is not adequately setting the buffers because the sending side is high bandwidth with low latency.

3. From the BIGIP01 SSH window run the following packet capture filtering in SYN and SYN/ACK packets ('tcp[13] & 2 != 0') from the client.::

    tcpdump -nni external host 10.1.30.6 and 'tcp[13] & 2 != 0' -c 4



   The output will be similar to this::

      11:14:45.819337 IP 10.1.30.6.36460 > 10.1.20.103.443: Flags [S], seq 991761495, win 64240, options [mss 1460,sackOK,TS val 142205951 ecr 0,nop,wscale 7], length 0 in slot1/tmm0 lis= port=1.2 trunk=
      11:14:45.819391 IP 10.1.20.103.443 > 10.1.30.6.36460: Flags [S.], seq 2970736692, ack 991761496, win 23360, options [mss 1460,nop,wscale 9,sackOK,TS val 1829159866 ecr 142205951], length 0 out slot1/tmm0 lis=/Common/web01_vs1 port=1.2 trunk=

   Here the client (10.1.30.6) is advertising wscale 7 and BIGIP01 is responding with wscale 9 so the TCP Window can scale up to the buffer limits determined by the f5-prgressive TCP profile. 

4. Go back to the BIGIP01 UI and change webo1_vs1 to use a custom TCP profile configure for the lab with 3MB (tcp_3mb) buffers and click update at the bottom of the page to save the change.  Higher buffers will use more memory for each TCP connection  while f5-tcp-progressive can also use more CPU as it calculates the buffer sizes.
 
   .. image:: ../images/tcp_3mb.png
       :width: 500px


5. Go back to the BIGIP01 SSH window and run the same packet capture.  You should no longer see any TCP Zero Window packets.  TMOS is now able to take in the server response as fast as possible and hold the data in memory until the client acknowleges receipt.  (reword??)::

    timeout 5s tcpdump -nni internal host 10.1.10.15 and 'tcp[14:2] == 0 && tcp[13] == 16' -s 500
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on internal, link-type EN10MB (Ethernet), capture size 500 bytes

    0 packets captured
    0 packets received by filter
    0 packets dropped by kernel


Windows Scale Review Using Wireshark Screenshots
------------------------------------------------

To speed up the process, you will just review TCP Window Scale screenshots taken from packet captures from the 3 TCP profiles in used in the previous selection.


1. Base Condition - tcp-wan-optimized/tcp-lan-optimized TCP Profiles assigned

   .. figure:: ../images/tcp_base_ws_server.png
      :width: 850px

      Server-side: The TCP Window size is limited to 65353 bytes because Window Scaling is not enabled.  The TCP Window drops to zero many times throughout the single TCP stream.


2. TCP-Progressive

   .. figure:: ../images/tcp_prog_ws_server.png
      :width: 850px

      Server-side: The TCP Window size grows a bit larger towards the end of the TCP stream but there are still many drops to zero bytes.  This is due to the was TCP_Progressive calculates the buffers with a low latency link.


3. Tcp_3mb Profile

   .. figure:: ../images/tcp_3mb_ws_server.png
      :width: 850px

      Server-side: The TCP Window size grows to 3mb in about 100ms and stays there throughout the TCP stream.  There are TCP Zero Window events.



  

