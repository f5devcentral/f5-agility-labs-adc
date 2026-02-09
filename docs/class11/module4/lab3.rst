Signed Zone
###################

Navigate to: **DNS  ››  Zones : DNSSEC Zones : DNSSEC Zone List**

.. image:: /class2/media/dnssec-zone.png
   :scale: 50

Create DNS Express zone signed by DNSSEC

.. csv-table::
   :header: "Setting", "Value"
   :widths: 15, 15

   Name, example.com
   Zone Signing Key, example.com_zsk
   Key Signing Key, example.com_ksk

.. image:: /class2/media/dnssec-new-zone.png
   :scale: 50


TMSH commands for DNSSEC signed zone creation:

.. admonition:: TMSH

 tmsh create ltm dns dnssec zone example.com keys add { example.com_ksk example.com_zsk }
