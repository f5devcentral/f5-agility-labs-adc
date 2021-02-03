Decrypt SSL with iRule
======================

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

Now Follow this link **instead of clicking on next** in order to see how to import the PreMasterSecret in Wireshark:

.. toctree::
   :maxdepth: 1
   :glob:

   lab11