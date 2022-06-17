Decrypt with tcpdump --f5 ssl
=============================

Beginning with v15.x of BIG-IP there is a tcpdump option that has been added that removes the requirement for an iRule to create a Pre Master Secret file.  A Pre Master Secret file is used to decrypt the PCAP data in a packet capture.  It can be imported into Wireshark to decrypt the data within each packet.  In order to do this do the following:

#. SSH using Putty into the BIG-IP01 box.   

#. Enable the **tcpdump.sslprovider** db varialbe.

   .. code-block:: bash

      tmsh modify sys db tcpdump.sslprovider value enable

#. Now when you take a packet capture you need to add **--f5 ssl** to the end of your command like this:

   .. code-block:: bash
      
      tcpdump -nni 0.0:nnn -s0 -w /var/tmp/hackazon-ssl.pcap host 10.1.20.103 --f5 ssl 

   .. NOTE:: Notice that we've got a warning message because Master Secret will be copied to tcpdump capture itself, so we need to be careful with who we share such capture with.

#. Now that the packet capture is running open Chrome and click on the Hackazon link and browse around the website.

#. Stop the packet capture with `Ctrl + C`.  

#. Start WINSCP from the jumpbox and copy the hackazon-ssl.pcap file from the /var/tmp directory.

#. Open Wireshark and open the hackazon-ssl.pcap file.

#. Once we have the packet capture we will also need to enable the F5 TLS protocols in Wireshark:

   a. Go to Analyze, Enable Protocols

      .. image:: /_static/class4/enable-protocols.jpeg
         :scale: 60%
   
   b. Search for F5 and check F5 TLS:

      .. image:: /_static/class4/enable-f5tls.jpeg
         :scale: 60%

#. Apply the following filter in Wireshark: `f5ethtrailer.tls.keylog`

#. Now expand the F5 Ethernet Trailer Protocol, then F5 TLS on any of the packets.

#. If you right click the Keylog entry and then select copy, and then value, this will put the keylog value into your clipboard and you can manually build a Pre Master Secret Log file:
    
   .. image:: /_static/class4/keylogvalue.png
      :scale: 80 %

#. Now you would open a notepad document and paste the clipboard value into the doc and save it as session.pms.  You would need to do this for every packet that has the f5ethtrailer.tls.keylog value which could take a long time.

#. The Pre Master Secret file will look similar to this after creating:

   .. image:: /_static/class4/presecretfile.png
      :scale: 80 %

Automate Pre Master Secret File Creation
----------------------------------------

#. Open a Command Prompt in your RDP Session.  Run the following command:

   .. code-block:: bash
      
      "c:\Program Files\Wireshark"\tshark.exe -r c:\users\user\Documents\hackazon-ssl.pcap -Y "f5ethtrailer.tls.keylog" -T fields -e f5ethtrailer.tls.keylog >> c:\users\user\Documents\session.pms

   .. NOTE:: The command is in the format of: **"tshark.exe -r <packet capture file> -Y "f5ethtrailer.tls.keylog" -T fields -e f5.ethtrailer.tls.keylog >> <file to write to>"**
             
             The -Y sets a display filter, the -T says to look for Field values, -e pulls tha values from the fields.

#. However you created the Pre Master Secret file it can now be used in Wireshark to decrypt the traffic following instructions on the next page.

