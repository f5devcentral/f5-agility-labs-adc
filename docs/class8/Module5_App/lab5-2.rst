Automate Pre Master Secret File Creation
========================================

#. Open a Command Prompt in your RDP Session.  Run the following command:

   .. code-block:: bash
      :linenos:

      "c:\Program Files\Wireshark"\tshark.exe -r c:\users\labUser\Desktop\ssl_f5.pcap -Y "f5ethtrailer.tls.keylog" -T fields -e f5ethtrailer.tls.keylog >> c:\users\labUser\Desktop\session.pms

   .. NOTE:: The command is in the format of: **"tshark.exe -r <packet capture file> -Y "f5ethtrailer.tls.keylog" -T fields -e f5.ethtrailers.tls.keylog >> <file to write to>"**
             
             The -Y sets a display filter, the -T says to look for Field values, -e pulls tha values from the fields.

#. However you created the Pre Master Secret file it can now be used in Wireshark to decrypt the traffic following instructions on the next page.

