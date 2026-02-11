Working With The Lab
====================

For the labs, you will access the BIG-IP, an Ubuntu client and a Windows 11 client.  You can choose to access the BIG-IP and Ubuntu client from the UDF access option for each device or RDP to the Windows 11 client and do the work from there.In order to begin you will need to go to F5's Unified Development Framework (UDF) site and launch the appropriate course. The following will walk you through that process.

Option 1 - Access from UDF
--------------------------
Each component within the UDF deplyment has multiple connection methods from the **Access** dropdown.  Here is an example of the Access options for BIGIP01

   .. image:: ../images/bigip_access.png

* For BIGIP01, you will use TMUI and SSH during the labs.  TMUI password is:
  
     User: admin
   | Password: admin.F5demo.com
   SSH login does not require a password

* For Ubuntu-Client, you will use SSH and no password is needed
* For Windows-Client, you will use RDP.  An RDP file will save to your browser's download location.

   User: labUser
   Password: lab.F5demo.com

   #. Open a web browser and head to the following link `Unified Demo Framework <https://udf.f5.com/>`_.
   #. Click on the Non-F5 Users selection.
   #. Login with your existing F5 account or create a new account.
   #. In the left navigation bar select Courses.
   #. Search for "Instance Manager Templates" in the search bar under the Content tab.
   #. Select "AppWorld 2025 - NGINX One - Instance Manager Templates" 
   #. Select the Details button on the title screen.
   #. Click the Details button on the blueprint in the lower right.
   #. Select the Deploy option.
   #. Select the Start option.
   #. In the pop up window set the Autostop Duration to 4 hours.
   #. Leave everything else default and click start.

The UDF deployment will take a few minutes to stand up. In the meantime feel free to read through the Documentation section of the deployment.

Recap
-----
You now have the following:

   - Logged into the UDF portal
   - A working UDF deployment
   - Access to the key Lab components