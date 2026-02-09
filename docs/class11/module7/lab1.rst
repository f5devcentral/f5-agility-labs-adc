Create an iRule
############################

Navigate to: **DNS  ››  Delivery : iRules : iRules List**

.. image:: /class2/media/irule-new.png 
   :scale: 75

Create a new iRule by copying the content below and pasting into the iRule editor window: 

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   Name, DNS-query-filtering  

.. code-block:: tcl
 
   when RULE_INIT {
     # Set categories to block for DNS hosts
     set static::blocked_categories {
       /Common/Bot_Networks
       /Common/Spyware
       /Common/Malicious_Web_Sites
       /Common/Adult_Content
       /Common/Entertainment
     }
 
 
     # CONFIGURATION
     # Check all requests by default
     set static::request_check 1
     # If the category returns as blocked, return NXDOMAIN (1)
     # Otherwise if (0), return a statically defined IP address
     set static::request_return_nxdomain 0
     set static::request_redirect_to "10.1.20.252"
     # Toggle for debug logs
     set static::request_debug 1
   }
 
 
   when DNS_REQUEST {
     if { $static::request_check } {
       set lookup_category [getfield [CATEGORY::lookup "http://[DNS::question name]"] " " 1]
       if { [lsearch -exact $static::blocked_categories $lookup_category] >= 1 } {
         if { $static::request_debug } {
            log local0. "BLOCKED: Category $lookup_category matching [DNS::question name] is filtered."
         }
         DNS::answer clear
         if { $static::request_return_nxdomain } {
            DNS::header opcode QUERY
            DNS::header rcode NXDOMAIN
         } else {
            if { [DNS::question type] equals "A" } {
               DNS::answer insert "[DNS::question name]. 111 [DNS::question class] [DNS::question type] $static::request_redirect_to"
            }
         }
         DNS::return
    } else {
      if { $static::request_debug } {
         log local0. "Category $lookup_category matching [DNS::question name] is not filtered"
         }
       }
     }
   }
 

