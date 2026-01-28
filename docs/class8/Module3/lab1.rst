Appendix - Optional Packet Decryption Lab
==========================================

There may be times when you need to decrypt a capture for troubleshooting client or server behaviors.

Decrypt with tcpdump --f5 ssl
-----------------------------

Beginning with v15.x of BIG-IP there is a tcpdump option that has been added that removes the requirement for an iRule to create a Pre Master Secret file.  A Pre Master Secret file is used to decrypt the PCAP data in a packet capture.  It can be imported into Wireshark to decrypt the data within each packet.  In order to do this do the following:

#. SSH using Putty into the BIG-IP01 box.   

#. Enable the **tcpdump.sslprovider** db varialbe.

   .. code-block:: bash
      :linenos:
      
      tmsh modify sys db tcpdump.sslprovider value enable 

#. From BIGIP01, ssh window we will collect a new packet capture with the **--f5 ssl** option at the end like this:

   .. code-block:: bash
      :linenos:

      tcpdump -nni external:nnn -s0 -vv -w /var/tmp/ssl_f5.pcap host 10.1.20.104 --f5 ssl 

   .. NOTE:: Notice that we've got a warning message because Master Secret will be copied to tcpdump capture itself, so we need to be careful with who we share such capture with.

#. Now that the packet capture is running, run ./ssl_testing.sh from the unbuntu-client.  The script will connect to the web02 Virtual Server with Curl and request multiple files over a single TLS connection.  The script will loop 5 times so you willl have multiple TLS streams to review later.

#. When the script is done running, stop the packet capture on BIGIP01 with `Ctrl + C`.  

#. Start WINSCP from the Windows client and copy ssl_f5.pcap from the /var/tmp directory to the Windows desktop (default save location in WINSCP).

#. Double-click ssl_f5.pcap on the Desktop to open the file in Wireshark.

#. Once we have the packet capture we will also need to enable the F5 TLS protocols in Wireshark (only needed in Wireshark before version 4.2.  If 4.2 or newer is installed skip this step.):

   a. Go to Analyze, Enable Protocols

      .. image:: ../images/enable-protocols.jpeg
         :scale: 60%
   
   b. Search for F5 and check F5 TLS:

      .. image:: ../images/enable-f5tls.jpeg
         :scale: 60%

#. Apply the following filter in Wireshark: `f5ethtrailer.tls.keylog`. You should see around 20 packets listed since the capture contains the client and server flows.

#. Click on the 1st packet in the list

#. Now expand the F5 Ethernet Trailer Protocol, then F5 TLS.

#. If you right click the Keylog entry and then select copy, and then value, this will put the keylog value into your clipboard and you can manually build a Pre Master Secret Log file:
    
   .. image:: ../images/keylogvalue.png
      :scale: 80 %

#. Now you would open a notepad document and paste the clipboard value into the doc and save it as manual_session.pms.  You would need to do this for every packet that has the f5ethtrailer.tls.keylog value which could take a long time.

#. The Pre Master Secret file will look similar to this after creating:

   .. image:: ../images/presecretfile.png

Automate Pre Master Secret File Creation
----------------------------------------

#. Open a Command Prompt in your RDP Session.  Run the following command:

   .. code-block:: bash
      :linenos:

      "c:\Program Files\Wireshark"\tshark.exe -r c:\users\labUser\Desktop\ssl_f5.pcap -Y "f5ethtrailer.tls.keylog" -T fields -e f5ethtrailer.tls.keylog >> c:\users\labUser\Desktop\session.pms

   .. NOTE:: The command is in the format of: **"tshark.exe -r <packet capture file> -Y "f5ethtrailer.tls.keylog" -T fields -e f5.ethtrailers.tls.keylog >> <file to write to>"**
             
             The -Y sets a display filter, the -T says to look for Field values, -e pulls tha values from the fields.

#. However you created the Pre Master Secret file it can now be used in Wireshark to decrypt the traffic following instructions on the next page.

Decrypting SSL in Wireshark
---------------------------

Using the pre-master key file created in the previous step and the f5_ssl.pcap file saved to the Desktop, you will decrpyt the contents.  Use the following steps:


#. Open Wireshark (if not already open).  

#. Once Wireshark is open go to Edit/Preferences.

#. Expand on the left side, Protocols, then select TLS.

   .. image:: ../images/premaster-session.png
      :height: 500px

#. Browse to the pre-master session key file saved on the Desktop and click on Open.  Then Click OK.

#. CLick File then open and select the ssl_f5.pcap file from the Desktop.

#. Apply a display filter of **http**.  You will see all the decrypted HTTP packets

   .. image:: ..images/decryptedHTTP.png

#. Right click on one of the packets and select Follow, HTTP Stream.

   .. image:: ../images/follow-http-stream.png

#. You will now see unencrypted HTTP data in the capture as follows:

   .. image:: ../images/ssl-decrypted-data.png