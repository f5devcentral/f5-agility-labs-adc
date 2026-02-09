Log Profile
##################################################

In order to Log DNS queries, responses, or both, a logging profile must be created. The log profile specifies both the formatting and destination of the log messages which is typically off the BIG-IP using High Speed Logging (HSL).  

Normally due to log volume, DNS logs would be sent off the BIG-IP, but for the purpose of the lab we will use a local syslog destination to easily see log messages. 

#. Create a "Log Publisher" for local syslog.

   Navigate to: **System  ››  Logs : Configuration : Log Publishers**

   .. image:: /class2/media/system_log_publisher_flyout.png
      :scale: 50
 
   Create a local syslog publisher as shown in the table below:

   .. csv-table::
      :header: "Setting", "Value"
      :widths: 15, 15

      "Name", "local-syslog-publisher"
      "Destinations", "local-syslog"

   .. image:: /class2/media/sys_syslog_publisher_details.png
      :scale: 50

   .. admonition:: TMSH

      tmsh create sys log-config publisher local-syslog-publisher { destinations add { local-syslog { } } }

#. Create a "Logging Profile"

   Navigate to **DNS  ››  Delivery : Profiles : Other : DNS Logging**

   .. image:: /class2/media/dns_cache_transparent_flyout_router01.png
      :scale: 50
   
   Create a DNS logging profile as shown in the table below:

   .. csv-table::
      :header: "Setting", "Value"
      :widths: 15, 15

      "Name", "example_dns_logging_profile"
      "Log Publisher", "local-syslog-publisher"
      "Log Responses", "enabled"
      "Include Query ID", "enabled"

   .. image:: /class2/media/dns_cache_transparent_create_router01.png
      :scale: 50

   .. admonition:: TMSH

      tmsh create ltm profile dns-logging example_dns_logging_profile enable-response-logging yes include-query-id yes log-publisher local-syslog-publisher
