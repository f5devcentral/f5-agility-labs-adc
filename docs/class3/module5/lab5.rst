Lab 5: SSL Offload and Security
===============================

In this Lab we will configure client-side SSL processing on the BIG-IP

Objective:

-  Create a self-signed certificate

-  Create a client SSL profile

-  Modify your HTTP virtual server to use HTTPS

-  Add additional security to your HTTPS web server using the HTTP
   profile

We will create a self-signed certificate and key for a client SSL
profile to attach to our virtual server

Creating a Self-signed certificate and key
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Go to **System >> Certificate Management >> Traffic Certificate
   Management >> SSL Certificate List** and select **Create**

|image1|

a. **NOTE:** The default key size is **2048**. You can save SSL
   resources on the **server-side** by lowering this key size

|image2|

a. Enter:

   i.   **Name**: my-selfsigned-cert

   ii.  **Issuer**: Self

   iii. **Common Name**: www.f5demo.com

   iv.  Fill out the rest any way you would like

Creating SSL Client Profile
~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Go to **Local** **Traffic >> Profiles >> SSL >> Client** menu and
   select **Create**

|image3|

b. Under **General Properties**

   v. **Name**: my_clientssl_profile

c. Under **Configuration** in the **Certificate Key Chain** section,
   select the **Custom** box and hit **Add**

   vi.  In the **Add SSL Certificate to Key Chain** pop-up select:

        1. **Certificate**:         my-selfsigned-cert

        2. **Key**:                     my-selfsigned-cert

   vii. Select **Add**

|image4|

d. Hit **Finished.**

Building our New Secure Virtual Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Go to **Local Traffic >> Virtual Servers** and hit the **Create**
   button or hit the **“+”** next to Virtual Servers

   e. **Name**: secure_vs

   f. **Destination Address/Mask**: 10.1.10.105

   g. **Port**: 443 or HTTPS

   h. **SSL Profile (Client)**: my_clientssl_profile (the profile you
      just created)

   i. **Source Address Translation**: Auto Map (remember why we need
      this?)

   j. **Default Pool**: www_pool

   k. Default all other settings. (Notice you did not require an HTTP
      profile)

   l. **Finish**

2. Test our secure server. Go to you **secure_vs** at
   **https://10.1.10.105**

   m. If you want to watch member traffic, go to the **www_pool** and
      reset the statistics

   n. Browse to your secure virtual server

   o. What port did your pool members see traffic on?

Securing Web Applications with the HTTP profile
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Let’s begin by creating a custom HTTP profile

   p. Go to **Local Traffic >> Profiles >> Services**, select **HTTP**
      and create a new profile

   q. Under **General Properties**

      viii. **Name**: secure-my-website

   r. Under **Settings**:

      ix.  Set the **Fallback Host**: http://10.1.1.252 *(this will take
           you an internal site)*

      x.   **Fallback on Error Codes**: 404 (fallback site if a 404
           error is received)

      xi.  **Response Headers Allowed**: Content-Type Set-Cookie
           Location

      xii. **Insert XForwarded For**: Enabled (because we talked about
           it earlier)

|image5|

s. Attach your new HTTP Profile to your secure (HTTPS) virtual server

3. Browse to your secure virtual server.

   t. Do web pages appear normal?

   u. Now browse to a bad page

      xiii. For example,

            3. What is the result?

   v. Go to the **Request and Response Headers** page. You should see a
      sanitized server response at the bottom of the web page and the
      original client IP address

   w. You can compare the headers by accessing your HTTP virtual server
      at http://10.1.10.100

   x. While you are looking at the headers, check for the
      **X-Forwarded-For** header received by the server

      NOTE: Even though the data is encrypted between your browser and
      the virtual server, the LTM can still modify the data (i.e.
      resource cloaking) because the data is unencrypted and
      decompressed within TMOS

Archive your work in a file called: **lab5_security**

.. |image1| image:: images/image1.png
   :width: 7.5in
   :height: 2.0125in
.. |image2| image:: images/image2.png
   :width: 5.21822in
   :height: 4.58333in
.. |image3| image:: images/image3.png
   :width: 3.87037in
   :height: 3.59084in
.. |image4| image:: images/image4.png
   :width: 2.91428in
   :height: 1.63542in
.. |image5| image:: images/image5.png
   :width: 4.62003in
   :height: 4.89916in
