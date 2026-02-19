Task 3: Use iRules To Log Traffic Data
======================================

If you do not have AVR provisioned and do not have the ablility to enable AVR since reprovisioning requires a full service restart on the BIG-IP, you can use iRules to log traffic data.  In most cases, these iRules would be just used temporarily as they have the potential for creating a lot of log traffic.  With iRules you have flexibility on what you want to log and where you want to apply the logging.  The iRules in the lab are only simple examples of what can be done.  The traffic generated in the lab is not very complex nor a high number of client connections so the iRules shown won't have a lot of client-specific filtering that may be required in an environment with more traffic.  You will create iRules and assign them to Virtual Server(s) then remove the iRule assignment after collecting the needed log data.
    
#. From the left menu, select Local Traffic > Virtual Servers > iRules > iRule List and click the '+' to create a new iRule

  .. image:: ../images/iRule_Create_plus.png

    Use 'lab_ContentLength_rule' for the iRule name.  This rule will log the content-length of the HTTP responses passing through the system.  We will use this information later when tuning TCP profiles. You won't need to write down any of the specific sizes but just see what is happening - a lot of small, medium or large files.  Is it a mix of different size files?  Is is a lot of large files?

  Enter the following code to log HTTP content-length anf the URL to local LTM log (/var/log/ltm)::
  **use this format (no linenos needed?)**

   .. code-block:: tcl
      :linenos:

  | when HTTP_REQUEST {
  |   set reqURI [HTTP::uri]
  | }
  |
  | when HTTP_RESPONSE {
  |   log local0. "Content-Length (Bytes): [HTTP::header value Content-Length] - URI: $reqURI" 
  | }

  .. image:: ../images/iRule_Content-Length.png

  Click the Update button at the bottom of the page to save the changes.  The iRule needs to be applied to a Virtual Server before it can log the content-length.

#. From the left menu, select Local Traffic > Virtual Servers > Virtual Server List

  * Click on web01_vs1 then go to the the Resources tab where the iRule will be assigned.

  .. image:: ../images/iRule_vs_resources.png

  * Click on the Manage button to the right of the iRule section
  
  .. image:: ../images/iRule_manage_button.png  

  * Select rule lab_ContentLength_rule from the list on the right the click "<<" in the middle to assign the iRule to the Virtual server.
  
  .. image:: ../images/iRule_Content-Assigment.png
  
  * Click Finished button at the bottom of the page to save the change
  
#. View the new log data from the BIG-IP command-line

  * Open an SSH session to the BIG-IP from the UDF Components page.
  
  .. image:: ../images/udf_bigip01_access.png
  
  * Monitor the logs to see the content-length of the HTTP responses::
  
      tail -f /var/log/ltm | grep Content-Length
  
  The files names in the lab include the file size but that is most likely not going to be the case in other environments so the Content-Length header shows the size in bytes.  This traffic mix contains files 16KB to 3MB::

  Feb 12 13:04:41 bigip01.f5tcp.lab info tmm1[16963]: Rule /Common/lab_contentLength_rule <HTTP_RESPONSE>: Content-Length (Bytes): 3145728 - URI: /file_3mb.txt
  Feb 12 13:04:41 bigip01.f5tcp.lab info tmm2[16963]: Rule /Common/lab_contentLength_rule <HTTP_RESPONSE>: Content-Length (Bytes):  - URI: /index.nginx-debian.html
  Feb 12 13:04:41 bigip01.f5tcp.lab info tmm3[16963]: Rule /Common/lab_contentLength_rule <HTTP_RESPONSE>: Content-Length (Bytes): 1048576 - URI: /file_1mb.txt
  Feb 12 13:04:41 bigip01.f5tcp.lab info tmm[16963]: Rule /Common/lab_contentLength_rule <HTTP_RESPONSE>: Content-Length (Bytes): 16384 - URI: /file_16kb.txt
  Feb 12 13:04:41 bigip01.f5tcp.lab info tmm2[16963]: Rule /Common/lab_contentLength_rule <HTTP_RESPONSE>: Content-Length (Bytes): 262144 - URI: /file_256kb.txt
  Feb 12 13:04:41 bigip01.f5tcp.lab info tmm[16963]: Rule /Common/lab_contentLength_rule <HTTP_RESPONSE>: 

  * Type <ctrl-c> to exit the tail command

#. Detach the iRule from the Virtual Server

  * Since you are connected to BIG-IP01 via SSH, modify the web01_vs1 Virtual Server from tmsh
  
    type tmsh and <enter> at the command prompt
    Enter the following commands to remove the lab_ContentLength_rule iRule::

      modify ltm virtual web01_vs1 rules none
      save sys config

  * Type 'q' and <enter> to exist tmsh
  * Verify the iRule is no longer logging.  You may see a few log entries but it should not be scrolling::

      tail -f /var/log/ltm | grep Content-Length

  * Type <ctrl-c> to exit the tail command

#. Create another iRule in the UI

  * Go back to the previously opened BIGIP01 TMUI browser tab
  * From the left menu, select Local Traffic > Virtual Servers > iRules > iRule List and click the '+' to create a new iRule
    Use 'lab_response_rule' for the iRule name.  This rule will log URLs resulting in HTTP 4xx response codes.

    Enter the following code

  | when HTTP_REQUEST {
  |   set reqURI [HTTP::uri]
  | }
  |
  | when HTTP_RESPONSE {
  |   set resCode [HTTP::status]
  |   if { $resCode starts_with 4 } {
  |      log local0. "Response Code: $resCode - URI: $reqURI" 
  |   }
  | }

  * Click the update button at the bottom of the page to save the changes.
  
#. From the left menu, select Local Traffic > Virtual Servers > Virtual Server List

  * Click on web01_vs1 then go to the the Resources tab where the iRule will be assigned.
  * Click on the Manage button on the right of the iRule section
  * Select rule lab_response_rule from the list on the right the click "<<" in the middle to assign the iRule to the Virtual server
  * Click Finished button at the bottom of the page to save the change
 
View the new log data from the BIG-IP command-line

  * If not still open, open an SSH session to the BIG-IP from the UDF Components page.
  
  .. image:: ../images/bigip_ssh_access.png
  
  * Monitor the logs to see the content-length of the HTTP responses::
  
    tail -f /var/log/ltm | grep "Response Code"
  
  As we saw in the AVR example, the 40x response codes are related to the URL wiith the typo - /file_16kd.txt.

  Feb 17 23:35:29 bigip01.f5tcp.lab info tmm2[17067]: Rule /Common/lab_response_rule <HTTP_RESPONSE>: Response Code: 404 - URI: /file_16kd.txt
  Feb 17 23:35:29 bigip01.f5tcp.lab info tmm[17067]: Rule /Common/lab_response_rule <HTTP_RESPONSE>: Response Code: 404 - URI: /file_16kd.txt
  Feb 17 23:35:29 bigip01.f5tcp.lab info tmm1[17067]: Rule /Common/lab_response_rule <HTTP_RESPONSE>: Response Code: 404 - URI: /file_16kd.txt
  Feb 17 23:35:30 bigip01.f5tcp.lab info tmm1[17067]: Rule /Common/lab_response_rule <HTTP_RESPONSE>: Response Code: 404 - URI: /file_16kd.txt

  * Type <ctrl-c> to exit the tail command

#. Detach the iRule from the Virtual Server

  Since you are connected to BIG-IP01 via SSH modify the web01_vs1 Virtual Server from tmsh
  * type tmsh at the command Prompt
  * Enter the following commands to remove the lab_ContentLength_rule iRule::

      modify ltm virtual web01_vs1 rules none
      save sys config

  * Type 'q' and <enter> to exist tmsh
  * Verify the iRule is no longer logging.  You may see a few log entries but it should not be scrolling::

      tail -f /var/log/ltm | grep "Response Code"

  * Type <ctrl-c> to exit the tail command








  * Use AVR to see what is happening on the system at the HTTP layer.

    Reset the view to Last hour from the top left dropdown.  Use the Response Code filter on the right side to see what cades are available.  With the lab traffic you will have at least 200, 301, and 404 and response codes available.  Let's use the filters to see what is happening beyond 200 responses.  Click on 301 and the graphs will update showing just the data related to 301 response codes.  Click on URLs above on the right to see what URLs are causing the 301 (redirects).

  .. image:: ../images/avr_http_responses.png

    For lab, 301 responses are a result of redirect.html.  Redirects may be necessary or could be left over from a previous application migration.  You can take these results back to the application team(s) to see if redirects are expected.  It could be something as simple as clients requesting /login instead of /login/ that is causing the redirects.  Unneccesary redirects add time to the client connections and uses resources on the BIGIP and/or application servers.

    Go back to the Resonse Codes filter and click 301 to deselect it, then click on 404.  Look at the URLs filter, it will show the cause of the 404's - file_64kd.txt.  In the lab this cause is simple, the URL should be /file_64k**b**.txt not k**d**.txt.  In a production environment, you may see many other URLs resulting in 404s.  They could be simple typos from the client or from bad links in the applications.  The 404 filter may also show you evidence of someone scanning your application(s).  The point of this is learn how to see what is happening with application traffic.

  .. image:: ../images/avr_http_typo.png



This completes Lab 1.