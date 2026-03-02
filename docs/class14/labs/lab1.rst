Lab 1: Windows-client
=====================

From the Windows-client we will be able to access the BIG-IP TMUI, NGINX, and the websites protected with PQC profiles/OpenSSL. 

| In this section, you will primarily perform the following tasks:
- Log in to the Windows client and set the Google Chrome as the default browser

| 
1. Navigate to the details button of the Windows-client. Select either RDP or Console to access the Windows-client.  
    - MAC users should connect with RDP 
    - Windows users: If the RDP session does not display properly or cannot be resized to a larger screen, please connect using the Console instead
   

   **Note:** For RDP, select the appropriate resolution format -- This will initiate the rdp file download which then can be used to access the Windows-client.
      

   **Note:** To run the RDP session in "windowed" mode, choose a screen size from the drop-down list

   .. image:: ../_static/image01.png


2. Windows-client admin credentials - **uQpLYwsLUrsyz**

   .. image:: ../_static/image02.png

3. Accept the RDP certificate error if prompted

   .. image:: ../_static/image03.png

4. Open the Chrome browser and skip the sign-in process.  **DO NOT** attempt to reinstall Chrome or finish the update.

   **Note:** Kyber level PQC was an early access feature in Chrome, it has been removed from the current Chrome release. We will be using an older version of Chrome in  this lab.
       

   .. image:: ../_static/image05.png

5. Set Chrome as the default browser. This will need to be done in two places, the browser itself and in Windows System properties.  

6. Set Chrome as the default browser in Chrome

   .. image:: ../_static/image06.png
       

   .. image:: ../_static/image06-2.png

       

7. Set Chrome as the default browser in Windows System properties - Settings > Apps > Default apps > Google Chrome  
         

   .. image:: ../_static/image07.png

       
       