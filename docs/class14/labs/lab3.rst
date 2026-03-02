
Lab 3: NGINX Setup
==================

NGINX utilizes the OpenSSL package installed on the host operating system for the SSL library. The OpenSSL 3.5 package, SSL certificate, SSL Key, and NGINX configuration are already complete. Review the settings for understanding.

In this section, you will primarily perform the following tasks:

- Review PQC config on NGINX
- Inspect PQC listener settings
| 

1. From the UDF lab page, access the NGINX Web Shell  

   **Note:** Do not disconnect or close your existing Windows-client connection

   .. image:: ../_static/image28.png
    


2. View the NGINX configuration file from the prompt with: 

   ``cat /opt/nginx/nginx.conf``

   .. image:: ../_static/image29.png



3. The nginx conf file includes another location for the PQC listener. From the prompt, view the included PQC listener: 

   ``cat /opt/nginx/conf.d/pqc.conf``

   .. image:: ../_static/image30.png
    

In this configuration, we can see the listening port, the certificate and key used, and the SSL options for exchange, ML-KEM.

| 

NGINX Chrome PQC settings
-------------------------

The Chrome browser has experimental features that enable Kyber and ML-KEM. However, as mentioned earlier, these features have been removed from the current version of Chrome due to a security gap. In this section we will enable this feature to test PQC

Enable the security features in Chrome to use the Kyber settings, but prefer ML-KEM settings

In this section, you will primarily perform the following tasks:

- Validate TLS Negotiation

1. Open the Chrome browser and browse to ``chrome://flags/``


2. Change the experimental settings to enable "TLS 1.3 post-quantum key agreement", and enable "Use ML-KEM in TLS 1.3", and relaunch the browser

   **Note:** Use find (ctrl-f) to quickly locate the ``"tls"`` settings above  
   .. image:: ../_static/image27.png    

| 

NGINX PQC Listener Validation
-----------------------------

With Chrome, check the version of TLS negotiation and the ciphers used.

1. Open Chrome and browse to **https://10.1.10.30** the listener address on NGINX with the PQC SSL options

   .. image:: ../_static/image32.png


2. Proceed to the website

   .. image:: ../_static/image31.png



3. NGINX is hosting a page with the negotiated SSL curve version listed

   .. image:: ../_static/image33.png


4. Open the Chrome browser developer tools 

   .. image:: ../_static/image34.png
  

5. Scroll the developer tools to the left, exposing Privacy and security to show the TLS negotiation

   .. image:: ../_static/image35.png



That concludes the lab, you should have learned why PQC is a strategic security priority and how PQC is implemented in BIG-IP and NGINX.  

| 

Lab Complete
------------