Create an iRule
############################

Navigate to: **DNS  ››  Delivery : iRules : iRules List**

.. image:: /_static/class2/irule-new.png 

https://router01.branch01.example.com/tmui/Control/jspmap/tmui/dns/rule/list.jsp

Create new iRule, copy the content below and paste it.

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   Name, IPI_irule

.. code-block:: tcl

   when DNS_RESPONSE 
   {  
      # If Query type was A and response is an answer.  
      if { ([DNS::question type] eq "A") and ([DNS::ptype] == "ANSWER") } 
      {  
          set rrs [DNS::answer]  
          foreach rr $rrs 
          {  
             if { [DNS::type $rr] eq "A" } 
             {  
	   if {[llength [IP::reputation [DNS::rdata $rr]]] != 0} 
	   {  
	       # Bad IP Reputation for destination detected  
	       log local0. "$rr: \"[IP::reputation $ip]\", count: [llength [IP::reputation $rr]]" 
     	   }  
             }  
          }  
      }
   } 

.. admonition:: TMSH

   tmsh create ltm rule IPI_irule

