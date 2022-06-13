Configure the F5 Wireshark Plugin
=================================

Wireshark version 3.2.1 is installed on the jumpbox.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Wireshark 2.6 and greater have the F5 ethtrailer plugin already installed.  You will have to update one setting in Wireshark to get it fully working:

#. With Wireshark open browse to Analyze -> Enabled Protocols -> Search for F5.

#. Select the checkbox for F5ethtrailer.

#. Select OK.

#. In Wireshark select Edit, then Preferences, Expand Protocols and scroll down to Ethernet.  Select `Never` for `Assume padding for short frames with trailer`.

   .. image:: /_static/class4/wireshark-ethernet-protocol.png
      :scale: 55 %

If you have a version before 3.0 of wireshark you will need to download and install the F5 Wireshark plugin.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can download the F5 Wireshark plugin from devcentral.f5.com here:  https://devcentral.f5.com/s/articles/getting-started-with-the-f5-wireshark-plugin-on-windows.  In the lab the plugin is already downloaded to /home/f5student/Downloads/wireshark/.

#. Start Wireshark by double clicking the shortcut on the desktop.

#. Click on Help and then About Wireshark.

#. Click on the plugins tab and check to see what directory the plugins are installed to.

#. Open the plugin directory in file explorer.

#. Copy the F5 wireshark plugin that you downloaded from devcentral.f5.com to the plugins directory you found in the Help, About Wireshark options.

#. Depending on your OS and Wireshark version, you will need the correct plugin files from the correct folder.

#. Shutdwon Wireshark and restart it.

#. Click on Help and then About Wireshark.

#. Check the plugins tab again and make sure the F5 plugin is installed.

   .. image:: /_static/class4/wireshark-plugin.png
      :scale: 50 %