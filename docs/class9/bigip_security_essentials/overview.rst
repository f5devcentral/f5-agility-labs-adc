Overview
========

The F5 Control-Plane Security Onion model provides a layered approach to securing the BIG-IP
control plane, which includes management and administrative interfaces such as:

* TMUI (Configuration Utility)
* SSH
* iControl REST and SOAP
* Related management services and daemons (big3d, bigd)

This guide focuses exclusively on protecting these control-plane components.
Data-plane protections (Virtual Servers, WAF, AFM, DoS profiles, and traffic processing)
are outside the scope of this framework.

---

Control-Plane Definition
------------------------

According to F5 security best practices, the control plane includes all methods for managing
a BIG-IP system or deployment, including:

* Web-based management (TMUI)
* Programmatic management (iControl REST and SOAP)
* Command-line access (SSH and tmsh)
* Synchronization and monitoring services

Because the control plane has full administrative authority over the system,
it must be protected with the highest priority.

---

Security Onion Model
--------------------

The Control-Plane Security Onion applies layered defenses from the outside inward.
Each layer builds upon the previous one, providing defense-in-depth for administrative access.

::

        +-----------------------------+
        |        CORE LAYER           |
        |  Appliance Mode, RBAC, HSM  |
        |  Continuous Monitoring      |
        +-----------------------------+
                  ▲
        +-----------------------------+
        |       MIDDLE LAYER          |
        |  AAA, SSH Hardening, NTP,   |
        |  Management DMZ             |
        +-----------------------------+
                  ▲
        +-----------------------------+
        |        OUTER LAYER          |
        |  IP Restrictions, Lockdown |
        |  Timeouts, Password Policy |
        +-----------------------------+

Each layer increases security depth and operational maturity while reducing the risk
of unauthorized administrative access.

---

Design Principles
-----------------

The Security Onion model is guided by the following principles:

* **Least privilege:** Only required users and systems can access the control plane.
* **Defense in depth:** Multiple independent security controls are layered together.
* **Visibility:** All control-plane activity is logged and monitored.
* **Validation:** Security posture is continuously verified using automated diagnostics.
* **Resilience:** Recovery procedures are defined and tested.

---

Layer Summary
-------------

Outer Layer – Network Access Controls
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The outer layer focuses on quick-win protections with the highest immediate impact:

* Restrict management interface access by IP address
* Configure Self IP port lockdown
* Enforce session timeouts and login banners
* Eliminate default credentials
* Apply strong password policies

This layer provides the first and most critical barrier against unauthorized access.

---

Middle Layer – Enhanced Control-Plane Security
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The middle layer introduces enterprise-grade controls and protocol hardening:

* Centralized authentication (LDAP, RADIUS, TACACS+)
* Multi-factor authentication (where supported)
* SSH and management protocol hardening
* Secure time synchronization (NTP)
* Secure SNMP configuration
* Management network segmentation (management DMZ)

This layer improves accountability, auditability, and operational security.

---

Core Layer – Advanced Control-Plane Security
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The core layer delivers maximum security and compliance readiness:

* Role-Based Access Control (RBAC)
* Appliance mode enforcement
* Cryptographic protection of secrets (HSM / SecureVault)
* Break-glass account management
* Continuous monitoring and alerting
* Incident response and recovery procedures

This layer is intended for high-security and regulated environments.

---

Validation with iHealth
-----------------------

F5 iHealth provides automated verification of control-plane security settings.

Key validation capabilities include:

* Security Best Practices panel review
* Automated diagnostics for misconfigurations
* Detection of vulnerabilities affecting management interfaces
* Identification of indicators of compromise
* Continuous posture assessment through QKView uploads

Throughout this guide, iHealth diagnostics are referenced as validation mechanisms
for each security layer.

---

Scope and Boundaries
--------------------

This framework applies only to control-plane security.

It does not cover:

* Data-plane traffic protection
* Application security (WAF, bot defense)
* DDoS mitigation
* Virtual server security policies
* NAT and SNAT configuration

These areas should be addressed through separate security architectures and controls.

---

Next Steps
----------

Proceed through the guide in the following order:

1. Review prerequisites and environment requirements
2. Follow the implementation roadmap
3. Study each Security Onion layer
4. Complete the hands-on labs
5. Validate results using iHealth
6. Apply the operational checklist

Continue to:

