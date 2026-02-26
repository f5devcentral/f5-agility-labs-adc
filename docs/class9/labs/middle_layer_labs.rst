Middle Layer Labs – Enhanced Control-Plane Security
==================================================

Objective
---------
Integrate centralized authentication and harden control-plane protocols.

Lab 1: Enterprise AAA Integration
---------------------------------

Steps
~~~~~
1. Configure AAA authentication (TACACS+, LDAP/AD, or RADIUS).
2. Ensure local accounts remain available for break-glass scenarios (per policy).
3. Forward authentication and audit logs to a central syslog/SIEM target.

Validation
~~~~~~~~~~
::

   tmsh list auth
   tmsh list sys syslog

Expected Results
~~~~~~~~~~~~~~~~
* Admin logins are centrally authenticated.
* Control-plane auth events are exported to SIEM/syslog.

Lab 2: Protocol Hardening (SSH / NTP / SNMP)
--------------------------------------------

Steps
~~~~~
1. Harden SSH configuration (ciphers/MACs/KEX) per your security baseline.
2. Configure at least 3 reliable NTP servers and verify reachability.
3. Secure SNMP (prefer SNMPv3) and restrict access to monitoring systems.

Validation
~~~~~~~~~~
::

   tmsh list sys sshd
   tmsh list sys ntp
   tmsh list sys snmp

iHealth Verification
~~~~~~~~~~~~~~~~~~~~
* H726514 – NTP server configuration and reachability
