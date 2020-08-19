Taking a Capture from the F5
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let's take the information we have gathered so far and take a packet capture from the F5.

#. Start Putty and launch the bigip01 SSH session.

#. Login as root user.  Password is 'admin.F5demo.com'.

#. List the destination address of the virtual on the F5 using the following command:

#. tmsh list ltm virtual all

#. **note there is a virtual server named **LAMP** with a destination ip address of 10.1.10.200

#. Now take the destination IP address and compose a tcpdump command to track the traffic coming to this virtual server:

$. tcpdump -nni 0.0:nnn -s0 -w /var/tmp/lamp.pcap host 10.1.10.200

#. After starting the capture, start Chrome and click on the LAMP bookmark.  Stop the capture in the putty session by using 'Ctrl+c'.

#. Open WinSCP on the Windows jumpbox.  Download the lamp.pcap file to the local box.

#. Now open Wireshark and open the lamp.pcap file you just copied from the F5.
