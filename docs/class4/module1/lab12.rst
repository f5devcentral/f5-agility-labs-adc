Appendix
========

The Appendix is reference material for other ways to accomplish SSL decryption.

Decrypt SSL with iRule
----------------------

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

Decrypt traffic using Client
----------------------------

If you have access to the client you can use the following modifications for Windows or Ubuntu:


SSL Decrypt from Windows Client
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. To use the client to decrypt you must add a System Variable to log the session key data for decryption.  On a windows client you would go into the Environment Variables and add a SSLKEYLOGFILE value to a text file on the machine as in the following image.

   .. image:: /_static/class4/system_variables.png
      :height: 500px

#. In our lab environment there is a shortcut on the desktop to connect to the environment variables.

#. Now start the tcpdump on the F5 box similar to: 'tcpdump -nni 0.0:nnnp -s0 -w /var/tmp/ssl.pcap host 10.1.20.103'

#. Once the system variable has been put in place you can then launch a web browser and start the traffic that you want to analyze.

#. Once the traffic has been captured you will import the capture file into Wireshark and configre the SSL options to use the pre-master key file defined in the system variables. 


SSL Decrypt from Linux Client
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. On an Ubuntu client open a terminal window and run the following commands:

   .. code-block:: bash
      :linenos:

      touch session-key.log
      export SSLKEYLOGFILE=/home/f5student/session-key.log
      chromium-browser

#. This will launch the Chrome browser and once you close the browser it will stop logging the SSL Session key data.  

#. Once the traffic has been captured you will import the capture file into Wireshark and configure the SSL options to use the session-key.log file.

