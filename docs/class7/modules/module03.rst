Lab 3: SSL Offload and Security
===============================

In this Lab we will configure client side SSL processing on the BIG-IP.

Objective:

-  Create a self-signed certificate

-  Create a client SSL profile

-  Modify your HTTP virtual server to use HTTPS

-  Add addition security to your HTTPS web server using the HTTP profile

We will create a self-signed certificate and key and a SSL client
profile to attach to our virtual server.

Creating a Self-signed certificate and key
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Go to **System > Certificate Management > Traffic Certificate
   Management > SSL Certificates** **List** and select **Create**

.. image:: ../images/image52.png
   :width: 5.83333in
   :height: 1.83891in

This will take you to **Local Traffic >> SSL Certificates >> New SSL
Certificate…**

a. NOTE: The default key size is **2048**, you can save SSL resources on
   the **server-side** by lowering this key size.

.. image:: ../images/image53.png
   :width: 2.60099in
   :height: 2.75581in

b. Enter:

   i.   **Name**: my-selfsigned-cert

   ii.  **Issuer**: Self

   iii. **Common Name**: www.f5demo.com

   iv.  Fill out the rest any way you would like and then click Finished.

Creating SSL Client Profile
~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Go to **Local** **Traffic>Profiles>SSL>Client** menu and select
   **Create**.

.. image:: ../images/image54.png
   :alt: C:\Users\RASMUS~1\AppData\Local\Temp\SNAGHTMLf292a2.PNG
   :width: 2.15302in
   :height: 1.94805in

a. Under **General Properties**

   i. **Name**: my_clientssl_profile

b. Under **Configuration** in the **Certificate Key Chain** section,
   select the **Custom** box and hit **Add**.

   i.  In the **Add SSL Certificate to Key Chain** pop-up select:

       1. **Certificate**:         my-selfsigned-cert

       2. **Key**:                     my-selfsigned-cert

   ii. Select **Add**

.. image:: ../images/image55.png
   :width: 2.23377in
   :height: 1.08439in

c. Hit **Finished.**

Building our New Secure Virtual Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Go to **Local Traffic>Virtual Servers** and hit the Create button or
   hit the “+” next to Virtual Servers

   a. **Name**: secure_vs

   b. **Destination Address/Mask**: 10.1.10.105

   c. **Port**: 443 or HTTPS

   d. **SSL Profile (Client)**: my_clientssl_profile (the profile you
      just created)

   e. **Source Address Translation**: Auto Map (remember why we need
      this?)

   f. **Default Pool**: www_pool

   g. Default all other settings. (Notice you did not require an HTTP
      profile)

   h. **Finish**

2. Testing our secure server. 
      - **UDF** - Go to your RDP session and browse to **secure_vs** at **https://10.1.10.105**

   a. If you want to watch member traffic, go to the **www_pool** and
      reset the statistics.

   b. Browse to your secure virtual server

   c. What port did your pool members see traffic on?

Securing web applications with the HTTP profile
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Let’s begin by creating a custom HTTP profile.

   a. Go to **Local Traffic> Profiles>Services**, by default you are in HTTP profiles, click Create and generate a
      new profile

   b. Under **General Properties**

      i. **Name**: secure-my-website

   c. Under **Settings** use Custom settings and modify the following:

      i.   Set the **Fallback Host**: https://www.f5.com *(this will take you an alternate site)*

      ii.  **Fallback on Error Codes**: 404 *(fallback site if a 404 error is received)*

      iii. **Response Headers Allowed**: Content-Type Set-Cookie
           Location

      iv.  **Insert XForwarded For**: Enabled *(to save the original client ip)*

         .. image:: ../images/image56.png
            :alt: C:\Users\RASMUS~1\AppData\Local\Temp\SNAGHTML566674e6.PNG
            :width: 3.25in
            :height: 3.44635in

   v. Select **Finish**

   vi. Attach your new HTTP Profile to your secure_vs (HTTPS) virtual server by going to the virtual server, selecting the **HTTP Profile** and changing it from **None** to **secure-my-website**.

1. Browse to your secure virtual server in RDP session.

   a. Do web pages appear normal?

   b. Now browse to a bad page.

   c. Using your browser Inspect window check the server response.  Altenately you can to the **Request and Response Headers** from the **Demo** drop-down menu to view the headers. You should see your sanitized server response and the original client IP address in the **x-forward-for** header, which is itself new.

   d. You can compare the headers by accessing your HTTP virtual server your unsecure **www_vs** and looking at the responses.

.. note:: 
   Even though the data is encrypted between your browser and the virtual server, the LTM can still modify the data (i.e. resource cloaking) because the data is unencrypted and decompressed within TMOS.

