Key Signing Key
############################################

Navigate to: **DNS  ››  Delivery : Keys : DNSSEC Key List**

.. image:: /class2/media/dnssec-keys-create.png

Create a key signing key according to the following table:

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   Name, example.com_ksk
   Type, Key Signing Key
   Key Management, Manual
   Certificate, default.crt
   Private Key, default.key

.. image:: /class2/media/dnssec-ksk.png

TMSH commands for Key Signing key creation:

.. admonition:: TMSH

 tmsh create ltm dns dnssec key example.com_ksk key-type ksk certificate-file default.crt key-file default.key


