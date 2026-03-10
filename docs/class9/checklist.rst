Operational Security Checklist
==============================

Use this checklist to validate complete implementation of the
Control-Plane Security Onion.

Outer Layer
-----------

☐ Management interface isolated to dedicated network

☐ TMUI access restricted to approved source IP ranges

☐ Self IP Port Lockdown configured (Allow Custom on data VLANs)

☐ SSH access restricted to approved source IP ranges

☐ Firewall rules prevent internet exposure of control-plane services

☐ SNMP restricted to authorized monitoring systems

Middle Layer
------------

☐ Centralized AAA integration configured

☐ Role mapping verified

☐ MFA enforced for administrative access

☐ TLS 1.0 and 1.1 disabled

☐ Weak cipher suites removed

☐ API access restricted and monitored

Core Layer
----------

☐ RBAC roles defined and reviewed

☐ Unused accounts removed

☐ Appliance mode evaluated and enabled if appropriate

☐ Secrets protected (HSM or SecureVault as applicable)

☐ Break-glass account secured and tested

☐ Control-plane logs forwarded to SIEM

☐ Configuration backup automation verified

Validation
----------

☐ iHealth QKView uploaded

☐ No high-risk control-plane findings remain

☐ HA and ConfigSync verified functional

☐ Incident response procedures documented


This checklist should be reviewed quarterly or after major configuration changes.
