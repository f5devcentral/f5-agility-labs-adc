Decrypt with tcpdump --f5 ssl
=============================

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

#. You can also automate this by doing the following:

#. Open your packet capture in Wireshark, and set the following display filter: 'f5ethtrailer.tls.keylog'

#. Click on File, Export Packet Dissections, As JSON:

   .. image:: /_static/class4/exportpacket.png
      :scale: 50 %

#. In the Packet Range select Displayed and All Packets, give the file a name and click on Save.

#. Now load the JSON file onto a linux system (your BIG-IP would work) and run the following command:

   .. code-block:: bash
      :linenos:

      cat <json file> | jq -r .[]._source.layers.f5ethtrailer'."f5ethtrailer.tls"."f5ethtrailer.tls.keylog"' >> /var/tmp/session.pms

#. However you created the Pre Master Secret file it can now be used in Wireshark to decrypt the traffic following instructions on next page.
