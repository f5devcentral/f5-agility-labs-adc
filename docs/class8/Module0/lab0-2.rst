Working With Components
=======================

UDF refers to lab elements as Components.  For the labs, you will access a BIG-IP, an Ubuntu client and a Windows 11 client.  You can choose to access the BIG-IP and Ubuntu client from the UDF **Access** dropdown for each Component or RDP to the Windows 11 client and do the work from there.  


1. Click on the **Deployment** tab in UDF to see the components and their status.  The lab deployment will be fully operational when there is a green triangle next to each component. Each component within the deployment has at least one connection method from the **Access** dropdown as you'll see in the instructions below.
   
   The following image shows a deployment in the startup phase with spinning yellow gears next to some components.  It should take a few minutes for startup to complete.

   .. figure:: ../images/udf_deployment_tab.png
      :width: 900px


2. For BIGIP01, you will use TMUI and Web Shell options during the labs

3. For BIGIP01 UI, select **TMUI** from the Access dropdown

   .. image:: ../images/udf_bigip01_access.png
       :width: 350px


4. Login with the following credentials:

   | User: admin
   | Password: admin.F5demo.com


   .. image:: ../images/udf_bigip01_auth.png
       :width: 250px


5. For command-line access to BIGIP01, select **Web Shell** from the Access dropdown.  Web Shell will open as a new web browser tab.

   .. image:: ../images/udf_bigip01_webshell.png
       :width: 350px

 
6. For the Ubuntu-Client, you only need to use **Web Shell**. Web Shell will open as a new web browser tab.

   .. image:: ../images/udf_client_webshell.png
       :width: 300px


   Once connected to Ubuntu-Client enter the following commands::

      su ubuntu
      cd ~



7. For Windows-Client, you will use **RDP**.  If RDP access is blocked on your computer you can use **Console**.  


Option 1: Windows RDP
---------------------

1. From the Windows-Client Access dropdown, click on the RDP screen resolution you prefer.  An RDP file will save to your browser's download location as device ID.rdp similar to *78bd595f-5239-4b93-8d04-1bc3ab2582c5.rdp*

   .. image:: ../images/udf_windows_rdp_options.png
       :width: 350px


2. Find the .RDP file in your wed browser's download location and double-click to open with your RDP client
3. Use the following credentials:

    | User: labUser
    | Password: lab.F5demo.com


   .. image:: ../images/udf_RDP_login.png
       :width: 500px


4. Click continue to the connection prompt

   .. image:: ../images/udf_RDP_continue.png
       :width: 600px

Option 2: Windows Console
-------------------------

1. From the Windows-Client Access dropdown, click on **Console** to open a noVNC connection to the client in your web browser

   .. image:: ../images/udf_windows_console.png
       :width: 350px


2. You may need to click on the screen to get the login prompt to appear. The background image may be different during the lab session

   .. image:: ../images/udf_windows_console_screen.png
       :width: 500px


3. Click on **Lab User** from the account selection on the bottom left

   .. image::  ../images/udf_windows_user_select.png
       :width: 600px

4.  Use the following password to login

    | Password: lab.F5demo.com



**Note:** Throughout the labs you should leave a single connection open to the following as they will be used for multiple labs/tasks:

* BIGIP01 TMUI
* BIGIP01 Web Shell
* Ubuntu-Client Web Shell

Recap
-----
You now have the following:

* Logged into the UDF portal
* A working UDF deployment
* Access to the key Lab components