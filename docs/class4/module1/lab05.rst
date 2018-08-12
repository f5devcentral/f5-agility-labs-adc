Taking a Capture from the F5
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let's take the information we have gathered so far and take a packet capture from the F5.  

#. Start Putty and launch the bigip01 SSH session.

#. Login as root user.  Password is 'default'.

#. List the destination address of the virtual on the F5 using the following command:

   .. code-block:: bash
      :linenos:    
      
      tmsh list ltm virtual hackazon.f5demo.com.app/hackazon.f5demo.com_vs destination

#. Now take the destination address and compose a tcpdump command to track the traffic coming to this virtual server:

   .. code-block:: bash
      :linenos:
      
      tcpdump -nni 0.0:nnn -s0 -w/var/tmp/hackazon.pcap host 10.1.10.201

#. After starting the capture, start Chrome and click on the Hackazon bookmark.  Browse around the site following a couple links.  Next go to the address bar and type in: "https://hackazon.f5demo.com:8080".  Then stop the capture in the putty session by using 'Ctrl+c'.

#. Open a terminal screen on the Ubuntu jumpbox.  Change Directory to the Downloads folder. 

#. Run the following command: 

   .. code-block:: bash
      :linenos:
  
      sudo scp root@10.1.1.245:/var/tmp/hackazon.pcap ./

#. You will be prompted to save the SSH keys. Type yes and hit enter, then enter the root password for the F5.  

#. Now open Wireshark and open the hackazon.pcap file you just copied from the F5.

#. If you run into issues copying the hackazon.pcap file to the jumpbox you can use the already created file in the Downloads folder hackazon2.pcap.
