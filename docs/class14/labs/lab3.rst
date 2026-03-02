
Lab 3: NGINX Setup
==================

NGINX utilizes the OpenSSL package installed on the host operating system for the SSL library. The OpenSSL 3.5 package, SSL certificate, SSL Key, and NGINX configuration are already complete. Review the settings for understanding.

In this section, you will primarily perform the following tasks:

- Review PQC config on NGINX
- Inspect PQC listener settings

1. From the UDF lab page, access the NGINX Web Shell  

    >Note: Do not disconnect or close your existing Windows-client connection

    ![webshell-access](images/image28.png)  
    <br>


2. View the NGINX configuration file from the prompt with: 

    `cat /opt/nginx/nginx.conf`

    ![nginx-conf](images/image29.png)

<br>

3. The nginx conf file includes another location for the PQC listener. From the prompt, view the included PQC listener: 

    `cat /opt/nginx/conf.d/pqc.conf`

    ![nginx-pqc-conf](images/image30.png)
    <br>

In this configuration, we can see the listening port, the certificate and key used, and the SSL options for exchange, ML-KEM.
<br>
<br>  

### NGINX Chrome PQC settings

The Chrome browser has experimental features that enable Kyber and ML-KEM. However, as mentioned earlier, these features have been removed from the current version of Chrome due to a security gap. In this section we will enable this feature to test PQC

Enable the security features in Chrome to use the Kyber settings, but prefer ML-KEM settings

In this section, you will primarily perform the following tasks:

- Validate TLS Negotiation

1. Open the Chrome browser and browse to `chrome://flags/`
<br>

2. Change the experimental settings to enable "TLS 1.3 post-quantum key agreement", and enable "Use ML-KEM in TLS 1.3", and relaunch the browser  

    >Note: Use find (ctrl-f) to quickly locate the `"tls"` settings above  
    <br>

    ![pqc-nginx-browser-settings](images/image27.png)  
<br>  
<br>  


### NGINX PQC Listener Validation

With Chrome, check the version of TLS negotiation and the ciphers used.

1. Open Chrome and browse to ```https://10.1.10.30``` the listener address on NGINX with the PQC SSL options

    ![nginx-ssl-error](images/image32.png)
<br>

2. Proceed to the website

    ![nginx-ssl-proceed](images/image31.png)
<br>


3. NGINX is hosting a page with the negotiated SSL curve version listed

    ![nginx-cipher](images/image33.png)
<br>

4. Open the Chrome browser developer tools 

    ![developer-tools](images/image34.png)
<br>  

5. Scroll the developer tools to the left, exposing Privacy and security to show the TLS negotiation

    ![tls-kyber](images/image35.png)
<br>
<br>

That concludes the lab, you should have learned why PQC is a strategic security priority and how PQC is implemented in BIG-IP and NGINX.
## Lab Complete