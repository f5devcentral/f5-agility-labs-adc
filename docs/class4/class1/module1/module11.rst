Decrypting SSL in Wireshark
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now you need to have your pre-master key file and your capture moved to your local box.  To do this do the following:

#. Open Terminal on the Ubuntu Jump Box.

#. Change directory to Documents by typing: 'cd Documents'.

#. Run the following commands:

   .. code-block:: bash
      :linenos:

      sudo scp root@10.1.1.245:/var/tmp/session.pms ./
      sudo scp root@10.1.1.245:/var/tmp/hackazon.pcap ./

After each of these commands you will be prompted to accept the SSH keys.  Type yes to continue. Then you will be prompted for the F5 root password.  Type that in as well.
 
#. Now open Wireshark.  

#. Once Wireshark is open go to Edit/Preferences.

#. Expand on the left side, Protocols, then select SSL.

   .. image:: /_static/class4/premaster-session.png
      :height: 500px

#. Browse to the pre-master session key file and click on save.

#. Open in Wireshark the pcap file you pulled down from the F5 BIG-IP. 

#. Right click on one of the SSL packets and select Follow, SSL Stream.

   .. image:: /_static/class4/follow-ssl-stream.png
      :height: 500px

#. You will now see unencrypted SSL data in the capture as follows:

   .. image:: /_static/class4/ssl-decrypted-data.png
      :height: 500px
