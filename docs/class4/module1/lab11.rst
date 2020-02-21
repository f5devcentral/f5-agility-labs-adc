Decrypting SSL in Wireshark
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now you need to have your pre-master key file and your capture moved to your local box.  To do this do the following:

#. Open WinSCP on the Windows jumpbox and connect to Bigip01.

#. Change local directory to Documents.

#. Pull from the remote directory the session.pms and your pcap files.

#. Now open Wireshark.  

#. Once Wireshark is open go to Edit/Preferences.

#. Expand on the left side, Protocols, then select TLS.

   .. image:: /_static/class4/premaster-session.png
      :height: 500px

#. Browse to the pre-master session key file and click on save.

#. Open in Wireshark the .pcap file you pulled down from the F5 BIG-IP.  You can use the original .pcap.

#. Right click on one of the SSL packets and select Follow, TLS Stream.

   .. image:: /_static/class4/follow-ssl-stream.png
      :height: 500px

#. You will now see unencrypted SSL data in the capture as follows:

   .. image:: /_static/class4/ssl-decrypted-data.png
      :height: 500px
