Task 2:  Benefits of Modified TCP Profiles
==========================================


1. Go the the UI of BIGIP01 and change the TCP profiles of **web01_vs1** to **F5-tcp-progressive** and click the Update button at the bottom of the page to save changes.

   .. image:: ../images/tcp_progressive.png
       :width: 600px


   The main difference with the **F5-tcp-progessive** is Auto Buffer size control to size the buffers based on link condition metrics <<reword from doc>>

   .. image:: ../images/tcp_progressive_buffers.png
       :width: 800px


2. Run the following command again from the Ubuntu-Client SSH window::
    
      ~/zerowindow3.sh


3. Go back to the BIGIP01 SSH window and run the same packet capture from earlier::
  
    timeout 5s tcpdump -nni internal host 10.1.10.15 and 'tcp[14:2] == 0 && tcp[13] == 16' -s 500  
   

   You should see a reduction in TCP Zero Window packets - maybe 300-450 packets in 5s.  This is an improvement but not the best for the traffic in the lab.  As we saw during Lab 1, there is a mix of 16kB-3MB files going through the system.  The f5-tcp-progressive TCP profile adjusts the TCP buffers based on connection conditions.  It uses bandwidth delay product to calculate the TCP buffers.
   
      
   In the lab, there is very low latency on the server-side connetions (pool members) and higher latency on the client-side connections.  With most HTTP traffic the client is requesting something from the server (GET) and the server is replying with the requested data.  The f5-tcp-progressive TCP profile is not adequately setting the buffers because the sending side is high bandwidth with low latency.

3. From the BIGIP01 SSH window, run the following packet capture filtering in SYN and SYN/ACK packets ('tcp[13] & 2 != 0') from the client.::

    tcpdump -nni external host 10.1.30.6 and 'tcp[13] & 2 != 0' -c 4



   The output will be similar to this::

      11:14:45.819337 IP 10.1.30.6.36460 > 10.1.20.103.443: Flags [S], seq 991761495, win 64240, options [mss 1460,sackOK,TS val 142205951 ecr 0,nop,wscale 7], length 0 in slot1/tmm0 lis= port=1.2 trunk=
      11:14:45.819391 IP 10.1.20.103.443 > 10.1.30.6.36460: Flags [S.], seq 2970736692, ack 991761496, win 23360, options [mss 1460,nop,wscale 9,sackOK,TS val 1829159866 ecr 142205951], length 0 out slot1/tmm0 lis=/Common/web01_vs1 port=1.2 trunk=

   Here the client (10.1.30.6) is advertising **wscale 7** with the SYN and BIGIP01 is responding with **wscale 9** in the SYN/ACK so the TCP Window can scale up to the buffer limits determined by the f5-prgressive TCP profile. 

4. Check the Ubuntu-Client output from the script.  You should see something similar to this::
   
      Run 3:...
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                       Dload  Upload   Total   Spent    Left  Speed
      100 3072k    0 3072k    0     0   760k      0 --:--:--  0:00:04 --:--:--  760k
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                       Dload  Upload   Total   Spent    Left  Speed
      100 3072k    0 3072k    0     0  2177k      0 --:--:--  0:00:01 --:--:-- 2177k
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                       Dload  Upload   Total   Spent    Left  Speed
      100 3072k    0 3072k    0     0  3046k      0 --:--:--  0:00:01 --:--:-- 3047k

      real  0m18.465s
      user  0m0.092s
      sys   0m0.100s

   With the **F5-tcp-progressive** TCP profiles, the download performance is much better even though TCP Zero Window packet were still seen during the capture.  This profile allows for TCP Window scaling so more data can be in flight pending an ACK.  It should take ~19 seconds to transfer 27MB of data.  Notice that within each of the 3 runs there is still a difference in through put from download 1 and files 2/3top because the TCP connection was still ramping up during download 1.


Test With Larger TCP Buffers
============================

1. Go back to the BIGIP01 UI and change web01_vs1 to use a custom TCP profile configure for the lab with 3MB buffers (tcp_3mb) and click update at the bottom of the page to save the change.  Higher buffers will use more memory for each TCP connection  while f5-tcp-progressive can also use more CPU as it calculates the buffer sizes.
 
   .. image:: ../images/tcp_3mb.png
       :width: 500px

2. Run the test script again from the Ubuntu-Client SSH window::
    
      ~/zerowindow3.sh


3. Go back to the BIGIP01 SSH window and run the same packet capture::
  
    timeout 5s tcpdump -nni internal host 10.1.10.15 and 'tcp[14:2] == 0 && tcp[13] == 16' -s 500 
  

   You should no longer see any TCP Zero Window packets.  TMOS is now able to take in the server response as fast as possible and hold the data in memory until the client acknowleges receipt.  (reword??)::

    timeout 5s tcpdump -nni internal host 10.1.10.15 and 'tcp[14:2] == 0 && tcp[13] == 16' -s 500
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on internal, link-type EN10MB (Ethernet), capture size 500 bytes

    0 packets captured
    0 packets received by filter
    0 packets dropped by kernel


4. Check the Ubuntu-Client output from the script.  You should see something similar to this::

      Run 3:...
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                       Dload  Upload   Total   Spent    Left  Speed
      100 3072k    0 3072k    0     0   997k      0 --:--:--  0:00:03 --:--:--  998k
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                       Dload  Upload   Total   Spent    Left  Speed
      100 3072k    0 3072k    0     0  2539k      0 --:--:--  0:00:01 --:--:-- 2538k
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                       Dload  Upload   Total   Spent    Left  Speed
      100 3072k    0 3072k    0     0  3608k      0 --:--:-- --:--:-- --:--:-- 3605k

      real  0m16.954s
      user  0m0.089s
      sys   0m0.089s


   With the **tcp_3mb** TCP profiles, the download performance is slightly better but there are no TCP Zero Windows seend during the test window. This profile allows for TCP Window to scale much faster and stay and the maximum value for the remainder of the TCP stream.  It should take ~17 seconds to transfer 27MB of data and the first download is still slower in each loop.

Disable TCP Slow Start
----------------------

1. From BIGIP01 UI, go to **Local Traffic > Profiles > Protocols > TCP** and click on **tcp_3mb** from the list
2. Scroll down to the Congestion Control section
3. Click the check box to the far right of Slow Start to enable editing of the Slow Start property
4. Clear the Enabled check box for Slow Start

   .. image:: ../images/tcp_3mb_cong_control.png
       :width: 900px


5. Click the Update button at the bottom of the page to save the setting
6. Run the test script again from the Ubuntu-Client SSH window::
    
      ~/zerowindow3.sh


7. Check the Ubuntu-Client output from the script.  You should see something similar to this::

      Run 3:...
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                       Dload  Upload   Total   Spent    Left  Speed
      100 3072k    0 3072k    0     0  1683k      0 --:--:--  0:00:01 --:--:-- 1683k
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                       Dload  Upload   Total   Spent    Left  Speed
      100 3072k    0 3072k    0     0  7561k      0 --:--:-- --:--:-- --:--:-- 7566k
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                       Dload  Upload   Total   Spent    Left  Speed
      100 3072k    0 3072k    0     0  7569k      0 --:--:-- --:--:-- --:--:-- 7566k

      real	0m7.968s
      user	0m0.074s
      sys	0m0.085s

   The total time should be ~8 seconds after disabling Slow Start.  The biggest improvement in the lab is on the client side.  The server-side is able to react quicker to TCP Window changes with the low latency.  The client-side is now able to ramp up faster with Slow Start disabled.  Slow Start is there to prevent TCP connections from overloading available network bandwidth.  If you see congestion and packet retransmissions within an environment you shoulw leave Slow Start enabled.  With the flxibility of TMOS, Slow Start can be enabled/disabled on either server or client sides of the connection independently.


Windows Scale Review Using Wireshark Screenshots
------------------------------------------------

To speed up the process, you will review TCP Window Scale screenshots taken from packet captures from the 3 TCP profiles used in the previous sectiona.


1. Base Condition** - tcp-wan-optimized/tcp-lan-optimized TCP Profiles assigned

   .. figure:: ../images/tcp_base_ws_server.png
      :width: 950px

      Server-side: The TCP Window size is limited to 65353 bytes because Window Scaling is not enabled.  The TCP Window drops to zero many times throughout the single TCP stream.


2. F5-tcp-Progressive

   .. figure:: ../images/tcp_prog_ws_server.png
      :width: 950px

      Server-side: The TCP Window size grows a bit larger towards the end of the TCP stream but there are still many drops to zero bytes.  This is due to the was TCP_Progressive calculates the buffers with a low latency link.


3. Tcp_3mb Profile

   .. figure:: ../images/tcp_3mb_ws_server.png
      :width: 950px

      Server-side: The TCP Window size grows to 3mb in about 100ms and stays there throughout the TCP stream.  There are TCP Zero Window events.


Wireshark Instructions for Windows Scaling View
-----------------------------------------------

<<add some screen shots for TCP Window stats>>

  

