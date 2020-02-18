Taking a Capture from the F5
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let's take the information we have gathered so far and take a packet capture from the F5.

#. Start Putty and launch the bigip01 SSH session.

#. Login as root user.  Password is 'P@ssw0rd!'.

#. List the destination address of the virtual on the F5 using the following command:

   .. code-block:: bash
      :linenos:

      tmsh list ltm virtual /Sample_04/A1/serviceMain |grep destination

#. Now take the destination IP address and compose a tcpdump command to track the traffic coming to this virtual server:

   .. code-block:: bash
      :linenos:

      tcpdump -nni 0.0:nnn -s0 -w /var/tmp/hackazon.pcap host 10.1.20.103

#. After starting the capture, start Chrome and click on the Hackazon bookmark.  Browse around the site following a couple links.  Next go to the address bar and type in: "http://10.1.20.15:8443".  Then stop the capture in the putty session by using 'Ctrl+c'.

#. Open WinSCP on the Windows jumpbox.  Download the hackazon.pcap file to the local box.

#. Now open Wireshark and open the hackazon.pcap file you just copied from the F5.

#. If you run into issues copying the hackazon.pcap file to the jumpbox you can use the already created file in the My Documents folder hackazon2.pcap.
