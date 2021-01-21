SSL Decryption
~~~~~~~~~~~~~~

There are two ways to decrypt the SSL traffic. Both ways require that you perform one of the following tasks before you take the TCP Capture.  The first method is to modify the client and in most cases is more difficult since you won't always have access to the client.  If you do then client modifications for Windows or Ubuntu follow:

In our lab environment you can do either way.  We will focus on capturing from the BIG-IP since that will be the more commonly available. 

SSL Decrypt from Windows Client
-------------------------------

#. To use the client to decrypt you must add a System Variable to log the session key data for decryption.  On a windows client you would go into the Environment Variables and add a SSLKEYLOGFILE value to a text file on the machine as in the following image.

   .. image:: /_static/class4/system_variables.png
      :height: 500px

#. Once the system variable has been put in place you can then launch a web browser and start the traffic that you want to analyze.

#. Once the traffic has been captured you will import the capture file into Wireshark and configre the SSL options to use the pre-master key file defined in the system variables. 


SSL Decrypt from Linux Client
-----------------------------

#. On an Ubuntu client open a terminal window and run the following commands:

   .. code-block:: bash
      :linenos:

      touch session-key.log
      export SSLKEYLOGFILE=/home/f5student/session-key.log
      chromium-browser

#. This will launch the Chrome browser and once you close the browser it will stop logging the SSL Session key data.  

#. Once the traffic has been captured you will import the capture file into Wireshark and configure the SSL options to use the session-key.log file.


SSL Decrypt from F5
-------------------

More often you will not have access to modify the client in order to capture the SSL session data.  The second option is to get the pre-master session data from the F5 itself by doing the following.

In the lab we have already configured the iRule and applied it to the Virtual Server.  If you want to validate you can login to the BIG-IP from the jumpbox by launching the Chrome browser and clicking on the BIG-IP01 shortcut.  Login as:

user: 'admin'
password: 'admin'

#. Configure an iRule as follows (ours is named 'ssl-decrypt.ir'):

   .. code-block:: tcl
      :linenos:

      when CLIENTSSL_HANDSHAKE {
         log local0. "CLIENT_RANDOM [SSL::clientrandom] [SSL::sessionsecret]"
         log local0. "RSA Session-ID:[SSL::sessionid] Master-Key:[SSL::sessionsecret]"
         log local0. "SERVER_HANDSHAKE_TRAFFIC_SECRET [SSL::clientrandom] [SSL::tls13_secret server hs]"
         log local0. "CLIENT_HANDSHAKE_TRAFFIC_SECRET [SSL::clientrandom] [SSL::tls13_secret client hs]"
         log local0. "CLIENT_TRAFFIC_SECRET_0 [SSL::clientrandom] [SSL::tls13_secret client app]"
         log local0. "SERVER_TRAFFIC_SECRET_0 [SSL::clientrandom] [SSL::tls13_secret server app]"
      }
      when SERVERSSL_HANDSHAKE {
         log local0. "SERVER_Side_IP:TCP source port:[IP::local_addr]: [TCP::local_port]"
         log local0. "CLIENT_RANDOM [SSL::clientrandom] [SSL::sessionsecret]"
         log local0. "RSA Session-ID:[SSL::sessionid] Master-Key:[SSL::sessionsecret]"
      }   

#. Apply this new iRule to the virtual server.  In our lab environment the iRule has already been created and applied to the Virtual Server.

#. You can now start a tcpdump and surf the website.  

#. After you have stopped the tcpdump, you will now need to SSH to the F5 and run the following command:

   .. code-block:: bash

      grep Session-ID /var/log/ltm | sed 's/.*\(RSA.*\)/\1/' > /var/tmp/session.pms
      grep -h -o 'CLIENT_RANDOM.*' /var/log/ltm >> /var/tmp/session.pms
      grep -h -o 'CLIENT_TRAFFIC.*' /var/log/ltm >> /var/tmp/session.pms
      grep -h -o 'SERVER_TRAFFIC.*' /var/log/ltm >> /var/tmp/session.pms
      grep -h -o 'CLIENT_HANDSHAKE.*' /var/log/ltm >> /var/tmp/session.pms
      grep -h -o 'SERVER_HANDSHAKE.*' /var/log/ltm >> /var/tmp/session.pms


#. Now the session.pms file can be pulled from the F5 and put into Wireshark as indicated on the next page. 

tcpdump --f5 ssl
----------------

Beginning with v15.x of BIG-IP there is a tcpdump option that has been added that removes the requirement for an iRule to decrypt TLS with a Pre Master Secret file.  In order to do this do the following:

#. Enable the **tcpdump.sslprovider** db varialbe.

   .. code-block:: bash
      :linenos:
      
      tmsh modify sys db tcpdump.sslprovider value enable 

#. Now when you take a packet capture you need to add **--f5 ssl** to the end of your command like this:

   .. code-block:: bash
      :linenos:

      tcpdump -nni 0.0:nnn -s0 -w /var/tmp/hackazon-ssl.pcap host 10.1.20.103 --f5 ssl 

   Notice that we've got a warning message because Master Secret will be copied to tcpdump capture itself, so we need to be careful with who we share such capture with.

#. Once we have the packet capture we will also need to enable the F5 TLS protocols in Wireshark:

   a. Go to Analyze, Enable Protocols

      .. image:: /_static/class4/enable-protocols.jpeg
   
   b. Search for F5 and check F5 TLS:

      .. image:: /_static/class4/enable-f5tls.jpeg

#. Now you can expand the F5 TLS options on any of the packets that meet this filter: 'f5ethtrailer.tls.keylog'

#. If you right click the log and copy then select value, this will put the keylog value into your clipboard and you can manually build a Pre Master Secret Log file:
    
   .. image:: /_static/class4/keylogvalue.png
      :scale: 50 %

#. Make sure to copy all of the keylog values from each instance if you want to decrypt the whole file.  Otherwise you can copy the values from the streams that you are looking for specifically.

#. The Pre Master Secret file will look similar to this after creating:

   .. image:: /_static/class4/presecretfile.png
      :scale: 50 %

#. You can also automate this doing the following:

#. Open you packet capture in Wireshark, and set the following display filter: 'f5ethtrailer.tls.keylog'

#. Click on File, Export Packet Dissections, As JSON:

   .. image:: /_static/class4/exportpacket.png
      :scale: 50 %

#. In the Packet Range select Displayed and All Packets, give the file a name and click on Save.

#. Now load the JSON file onto a linux system and run the following command:

   .. code-block:: bash
      :linenos:

      cat <json file> | jq -r .[]._source.layers.f5ethtrailer'."f5ethtrailer.tls"."f5ethtrailer.tls.keylog"' >> /var/tmp/session.pms

#. However you created the Pre Master Secret file it can now be used in Wireshark to decrypt the traffic following instructions on next page.


