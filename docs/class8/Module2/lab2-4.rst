Task 4:  TCP Retranmission Timeout 
==================================

In this section, you will review default TCP packet retransmission behaviors and what adjustments can be made to reduce recovery time.  The best solution is to find and eliminate the cause of TCP retranmissions due to packet loss, link saturation or server load issues. This isn't always possible so there are some changes to the TCP profiles that can help.  The focus of this section will be TCP 3-way handshake failures due to packet loss or non-responsive servers.  Since the conditions of the internal LAN-side connections are typically known or controlled versus the many variables that can exist onthe WAN-side.  The Retransmission Timeout (RTO) adjustements in this section would be applied in a server-sdie TCP profile.

1. Run the following command From the BIGIP01 **Web Shell**::

    tcpdump -nni internal host 10.1.10.15 and port 75 -c 5


2. Run the following command from the Ubuntu-Client **Web Shell**::

    telnet web01.f5tcp.net 75


3. Go back to the BIGIP01 **Web Shell** browser tab and you will see tcpdump output similar to this::
   
    00:25:29.231087 IP 10.1.10.15.60148 > 10.1.10.30.75: Flags [S], seq 1490480506, win 14600, options [mss 1460,nop,wscale 0,sackOK,TS val 3514603278 ecr 0], length 0
    00:25:30.231063 IP 10.1.10.15.60148 > 10.1.10.30.75: Flags [S], seq 1490480506, win 14600, options [mss 1460,nop,wscale 0,sackOK,TS val 3514604278 ecr 0], length 0
    00:25:31.230965 IP 10.1.10.15.60148 > 10.1.10.30.75: Flags [S], seq 1490480506, win 14600, options [mss 1460,nop,wscale 0,sackOK,TS val 3514605278 ecr 0], length 0
    00:25:32.231209 IP 10.1.10.15.60148 > 10.1.10.30.75: Flags [S], seq 1490480506, win 14600, options [mss 1460,sackOK,eol], length 0
    00:25:33.230815 IP 10.1.10.15.60148 > 10.1.10.30.75: Flags [R.], seq 1490480507, ack 0, win 0, length 0


   The pool member is not responding to the TCP connection on port 75.  TMOS sends a SYN to pool member 10.1.10.30, in this example, but the server does not respond with a SYN/ACK.  TMOS retries the SYN 3 more times with an interval of 1 second before giving up and resetting the conneciton attempt.  This differs from the standard TCP SYN retransmit behavior where the timeout is doubled for each retransmission until the retry limit count is reached.

4. From the BIGIP01 **TMUI** window, go to **Local Traffic > Profiles > Protocol > TCP**  and click on the **tcp_retrans** profile name
   
5. From the **Timer Management** section of the TCP Profile, click the checkbox to the right of Minimum RTO (Retransmission Timeout) then change the Minimum RTO value to 200.

   .. image:: ../images/tcp_timer_management.png
       :width: 700px
     

6. From the Loss Detection and Recovery section, click the checkbox to the right of Initial Retransmission Timeout Base Multiplier for SYN Retransmission then change the value to 200.

   .. image:: ../images/tcp_loss_detect.png
       :width: 700px

      
7. Click the **Update** button at the bottom of the screen to save the changes
8. TCP Profile **tcp_retrans** is already applied to Virtual Server **retransmit_vs1** so no adjustments to the Virtual Server are needed
9. Run the following command From the BIGIP01 **Web Shell** window::

    tcpdump -nni internal host 10.1.10.15 and port 75 -c 5


10. Run the following command from the Ubuntu-Client **Web Shell** window::

     telnet web01.f5tcp.net 75


11. Go back to the BIGIP01 **Web Shell** browser tab and you will see tcpdump output similar to this::

     00:55:15.342766 IP 10.1.10.15.46930 > 10.1.10.30.75: Flags [S], seq 3657141189, win 14600, options [mss 1460,nop,wscale 0,sackOK,TS val 3516389389 ecr 0], length 0
     00:55:15.545402 IP 10.1.10.15.46930 > 10.1.10.30.75: Flags [S], seq 3657141189, win 14600, options [mss 1460,nop,wscale 0,sackOK,TS val 3516389592 ecr 0], length 0
     00:55:15.748466 IP 10.1.10.15.46930 > 10.1.10.30.75: Flags [S], seq 3657141189, win 14600, options [mss 1460,nop,wscale 0,sackOK,TS val 3516389795 ecr 0], length 0
     00:55:15.951326 IP 10.1.10.15.46930 > 10.1.10.30.75: Flags [S], seq 3657141189, win 14600, options [mss 1460,sackOK,eol], length 0
     00:55:16.154219 IP 10.1.10.15.46930 > 10.1.10.30.75: Flags [R.], seq 3657141190, ack 0, win 0, length 0


    TMOS is now sending SYN retries at 203ms intervals.  This change to the TCP Profile does not eliminate SYN retransmissions during TCP 3-way handshake but is does shorten the retry process allowing for quicker reselection of a working server.  In the lab, there isn't a working pool member so you will only see a shorter time to conneciton reset.



Recap
-----    

You should have a better understanding of the default TCP Profile behaviors and some adjustments that can be made to improve TMOS throughput.  Make sure to copy the document links from the beginning of Lab 2 for further review.

This concludes Lab 2