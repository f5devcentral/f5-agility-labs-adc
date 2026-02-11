Decrypting SSL in Wireshark
===========================

Using the pre-master key file created in the previous step and the f5_ssl.pcap file saved to the Desktop, you will decrpyt the contents.  Use the following steps:


#. Open Wireshark (if not already open).  

#. Go to Edit/Preferences.

#. Expand Protocols from the left side list, then select TLS.

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