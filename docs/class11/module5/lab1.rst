Trust Anchors
#############

Next, create a trust anchor to validate DNS payloads in a DNSSEC response.

Begin by connecting to the BIG-IP via a web shell and run the commands shown below:

.. code-block:: bash

   dig dnskey . | grep 257 > /root/dnskey.txt

   dnssec-dsfromkey -f /root/dnskey.txt .

.. image:: /class2/media/trusted-anchors-cli.png

Navigate to: **DNS  ››  Caches : Cache List  ››  validating-resolver_cache : Trust Anchors**

Select the validating-resolver_cache and click "Trust Anchors"

.. image:: /class2/media/selet_validating_resolver.png

.. image:: /class2/media/trust-anchor.png

For each *DS* record, enter them as trust anchors: 

.. image:: /class2/media/trust-ancor-1.png

.. image:: /class2/media/trusted-anchors-done.png

When using TMSH, enter the DS records, each surrounded by quotes (" "), and use the entire keys above for *<key 1>* and *<key 2>*

.. code-block:: tcl
   :linenos:

   tmsh modify ltm dns cache validating-resolver validating-resolver_cache trust-anchors replace-all-with { "<key 1>" "<key 2>" }
