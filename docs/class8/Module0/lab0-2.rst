Working With Components
=======================

UDF refers to lab elements as Components.  For the labs, you will access a BIG-IP, an Ubuntu client and a Windows 11 client.  You can choose to access the BIG-IP and Ubuntu client from the UDF **Access** dropdown for each Component or RDP to the Windows 11 client and do the work from there.  


1. Click on the **Deployment** tab in UDF to see the components and their status.  The lab deployment will be fully operational when there is a green triangle next to each component. Each component within the deployment has at least one connection method from the **Access** dropdown as you'll see in the instructions below.
   
   The following image shows a deployment in the startup phase - it should take a few minutes for startup to complete.

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


5. For command-line access to BIGIP01, select **Web Shell** from the Access dropdown

   .. image:: ../images/udf_bigip01_webshell.png
       :width: 350px

 
6. For the Ubuntu-Client, you only need to use **Web Shell**

   .. image:: ../images/udf_client_webshell.png
       :width: 452px


   Once connected enter the following commands::

      su ubuntu
      cd ~

      
7. For Windows-Client, you will use RDP.  An RDP file will save to your browser's download location.

   .. image:: ../images/udf_RDP_option.png
       :width: 500px


9. Find the .RDP file and click to open with your RDP client
10. Use the following credentials:

    | User: labUser
    | Password: lab.F5demo.com

    .. image:: ../images/udf_RDP_login.png
        :width: 500px


13. Click continue to the connection prompt

    .. image:: ../images/udf_RDP_continue.png
        :width: 600px


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