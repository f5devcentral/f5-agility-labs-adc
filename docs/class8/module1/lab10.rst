SSL Decryption
~~~~~~~~~~~~~~

There are two ways to decrypt the SSL traffic. Both ways require that you perform one of the following tasks before you take the TCP Capture.  The first method is to modify the client and in most cases is more difficult since you won't always have access to the client.  If you do then client modifications for Windows or Ubuntu follow:

In our lab environment you can do either way.  We will focus on capturing from the BIG-IP since that will be the more commonly available.

SSL Decrypt from Windows Client
-------------------------------

#. To use the client to decrypt you must add a System Variable to log the session key data for decryption.  On a windows client you would go into the Environment Variables and add a SSLKEYLOGFILE value to a text file on the machine as in the following image.

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

Login to the BIG-IP from the jumpbox by launching the Chrome browser and clicking on the BIG-IP01 shortcut.  Login as:

user: 'admin'
password: 'admin.F5demo.com'

#. Configure an iRule as follows (ours is named 'ssl-decrypt.ir'):

.. code-block:: tcl

   when CLIENTSSL_HANDSHAKE {
      log local0. "RSA Session-ID:[SSL::sessionid] Master-Key:[SSL::sessionsecret]"
   }

   when SERVERSSL_HANDSHAKE {
      log local0. "RSA Session-ID:[SSL::sessionid] Master-Key:[SSL::sessionsecret]"
   }

|image4|

#. Apply this new iRule to the LAMP SSL virtual server.

#. You can now start a tcpdump and surf the website.

#. After you have stopped the tcpdump, you will now need to SSH to the F5 and run the following command:

   .. code-block:: bash

      grep Session-ID /var/log/ltm | sed 's/.*\(RSA.*\)/\1/' > /var/tmp/session.pms

#. Now the session.pms file can be pulled from the F5 and put into Wireshark.

.. |image4| image:: images/image4.PNG
   :width: 6.32107in
   :height: 4.33645in
