DSC Lab Network Setup
---------------------

In the interest of focusing as much time as possible configuring your
application delivery controller, we have provided some resources and
basic setup ahead of time. These are:

-  Cloud-based lab environment complete with a Windows workstation, three
   virtual BIG-IPs (VE),  and a back-end banking application running on a
   Linux web server.

-  The virtual BIG-IP has been pre-licensed

If you wish to replicate these labs in your office you will need to
perform these steps accordingly.

To access the lab environment, you will require a web browser and
Remote Desktop Protocol (RDP) client software. The web browser will be
used to access the lab training portal. The RDP client will be used to
connect to a Windows workstation, where you will be able to access the
BIG-IPs (HTTPS, SSH).

You class instructor will provide additional lab access details.

Login info for lab jump box
===========================

| Use RDP to connect to the DNS host provided to you in the Agility Portal.  The user and password are:
|
| Username: ``student``
| Password: ``agility``
|
|
| Bigip1: ``10.128.1.245``
| Bigip2: ``10.128.1.246``
| Bigip3: ``10.128.1.247``
|

+------------------------------+------------------+
|BIG-IP Usernames              | BIG-IP Passwords |
+==============================+==================+
| admin                        | admin            |
+------------------------------+------------------+
| root                         | default          |
+------------------------------+------------------+


Timing for DSC Labs
^^^^^^^^^^^^^^^^^^^

The time it takes to perform each lab varies and is mostly dependent on
accurately completing steps. This can never be accurately predicted but
we strived to derive an estimate among several people each having a
different level of experience. Below is an estimate of how long it will
take for each lab:

+------------------------------------------------------+------------------+
| LAB Name (Description)                               | Time Allocated   |
+======================================================+==================+
| Lab 1 – Active/Active ScaneN Clustering Exercise     | 55 minutes       |
+------------------------------------------------------+------------------+
| Lab 2 – Sync-Only Groups Exercise                    | 30 minutes       |
+------------------------------------------------------+------------------+
| Lab 3 – Device Service Cluster Automation            | 60 minutes       |
+------------------------------------------------------+------------------+

