Install the F5 Wireshark Plugin 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Wireshark version 2.2.14 is installed on the jumpbox.

You can download the F5 Wireshark plugin from devcentral.f5.com here:  https://devcentral.f5.com/d/wireshark-plugin?lc=1.  In the lab the plugin is already downloaded to /home/f5student/Downloads/wireshark/.

#. Start Wireshark by double clicking the shortcut on the desktop.

#. Click on Help and then About Wireshark.

#. Click on the plugins tab and check to see what directory the plugins are installed to.

#. Open the plugin directory in file explorer.

#. Copy the F5 wireshark plugin that has been copied to /home/f5student/Downloads/wireshark/
   
   a. The jumpbox client is Ubuntu 64 bit.

   #. Open the Linux64-2.2.0 folder and copy the f5ethtrailer.so to the plugin directory determined in step 3.

   #. In order to move the file you will need to elevate permissions.  The easiest way to do this is from the command line terminal.  Use the command: 

      .. code-block:: bash
         :linenos:

         sudo cp ./Downloads/wireshark/Linux64-2.2.0/f5ethtrailer.so /usr/lib/x86_64-linux-gnu/wireshark/plugins/2.2.6/

#. Depending on your OS and Wireshark version, you will need the correct plugin files from the correct folder.

#. Shutdwon Wireshark and restart it.

#. Click on Help and then About Wireshark.

#. Check the plugins tab again and make sure the F5 plugin is installed.

   .. image:: /_static/class4/wireshark-plugin.png
      :scale: 50 %
