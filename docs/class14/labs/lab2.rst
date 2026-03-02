
Lab 2: BIG-IP Setup
===================

BIG-IP has version 17.5.1.3.0.0.19 installed. In version 17.1, PQC for client-side SSL profiles was introduced, which we will explore in this lab. In version 17.5.1, PQC for server-side SSL profiles was added, and both client and server-side ciphers were updated to the NIST standards at the time of publication.

BIG-IP supports both Kyber and ML-KEM, in this lab we will demonstrate Kyber, though ML-KEM is more widely adopted.

In this section, you will primarily perform the following tasks:

- Review BIG-IP cipher rules and cipher groups
- Review BIG-IP client ssl profile
- Verify the virtual server configuration

**Note:** We will not demonstrate server-side SSL PQC in this lab; however, the environment supports it, for exploration.  
 

1. Log into the BIG-IP to verify access and configuration

   From the Chrome browser, open the BIG-IP TMUI: `https://10.1.1.6`  

   **User:** admin -- **Password:** admin

   .. image:: ../_static/image08.png  
     

   .. image:: ../_static/image09.png
     

   .. image:: ../_static/image10.png
 

2. Post-quantum crypto configuration

   BIG-IP utilizes SSL Profiles for client and server-side TLS negotiations. Within the SSL Profile, attached cipher groups manage the cipher rules for negotiation.

   The BIG-IP Configuration has already been completed for the lab. If you would like to configure and familiarize yourself with a new SSL profile, please use the following knowledge article as a reference:

   `K000149577: Enable post-quantum cryptography in F5 BIG-IP TMOS <https://my.f5.com/manage/s/article/K000149577>`_
 

3. Navigate to BIG-IP cipher rules

   .. image:: ../_static/image11.png
 

4. The ``TMSH_PQC`` PQC profile has been created for you using TMSH.  Please review it using the TMUI.

   .. image:: ../_static/image12.png
 

5. Explore the TMSH_PQC rule, and verify the setup

   .. image:: ../_static/image13.png
 

6. Navigate to BIG-IP cipher groups

   .. image:: ../_static/image14.png
 

7. Explore the TMSH_PQC group, and verify the setup

   .. image:: ../_static/image15.png
 

8. Navigate to SSL Client profiles

   .. image:: ../_static/image16.png
 

9. Explore the TMSH_PQC client SSL profile, and verify the setup

   .. image:: ../_static/image17.png
     
    
   .. image:: ../_static/image18.png  
     

10. Navigate to the BIG-IP virtual servers  

|   .. image:: ../_static/image19.png  
     

11. Explore the pqc_vs virtual server, and verify the setup

|   .. image:: ../_static/image20.png
 
 

BIG-IP Chrome PQC settings
--------------------------

The Chrome browser, on the Windows-client has experimental features that enable Kyber and ML-KEM. However, as mentioned earlier, these features have been removed from the current version of Chrome due to a security gap. In this section you will enable the required settings in chrome to implement PQC

Enable the security features in Chrome to use the Kyber settings and disable the ML-KEM settings

1. Open the Chrome browser and browse to chrome://flags/

2. Change the experimental settings to enable "TLS 1.3 post-quantum key agreement", and disable "Use ML-KEM in TLS 1.3", and relaunch the browser  

   **Note:** Use find (ctrl-f) to quickly locate the `"tls"` settings above  
       

   .. image:: ../_static/image21.png

### BIG-IP PQC Virtual Server Validation

With Chrome, check the version of TLS negotiation and the ciphers used.

1. Open Chrome and browse to `https://10.1.10.100` the virtual server address on the BIG-IP with the PQC SSL Client profile attached  

   .. image:: ../_static/image22.png 
     

2. Proceed to the website  

   .. image:: ../_static/image23.png 
     

3. The loaded page is the NGINX default page

   .. image:: ../_static/image24.png  
     

4. Open the Chrome browser developer tools 

   .. image:: ../_static/image25.png)  
     

5. Scroll the developer tools to the left, exposing Privacy and security to show the TLS negotiation

   .. image:: ../_static/image26.png)  
 
 

### View PQC Statistics on BIG-IP  

1. Log into TMSH on BIG-IP using the `Web Shell`  

   **Note:** Do not disconnect or close your existing TMUI connection  
       

   .. image:: ../_static/image50.png
    
   .. image:: ../_static/image51.png

 
 

2. View the `TMSH_PQC` profile statistics 

    `tmsh show ltm profile client-ssl TMSH_PQC`

   **Note:** the **protocol** and **DH group** 
    
        

   .. image:: ../_static/image52.png

 

3.  View the `pqc_vs` virtual server statistics

    `tmsh show ltm virtual pqc_vs`  

       

   .. image:: ../_static/image53.png

 
 