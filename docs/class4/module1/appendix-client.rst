Decrypt traffic using Client
----------------------------

If you have access to the client you can use the following modifications for Windows or Ubuntu:


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

