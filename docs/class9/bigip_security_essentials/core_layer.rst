Core Layer – Advanced Control-Plane Security
===========================================

The core layer of the Control-Plane Security Onion represents the highest level of protection
for the BIG-IP control plane. These controls are designed for high-security and compliance-driven
environments and provide maximum defense-in-depth.

This layer requires the greatest implementation effort and ongoing maintenance, but delivers
the strongest security posture.

---

Objective
---------

The objectives of the core layer are to:

* Enforce least-privilege administrative access
* Protect cryptographic secrets and sensitive credentials
* Restrict system-level access through appliance mode
* Enable continuous monitoring and threat detection
* Prepare for incident response and recovery

This layer assumes that attackers may reach the control plane and focuses on limiting damage,
detecting compromise, and enabling rapid recovery.

---

Key Concepts
------------

The core layer is guided by the following principles:

* **Least privilege:** Administrators receive only the access required for their role.
* **Strong isolation:** System-level access is tightly controlled.
* **Secret protection:** Cryptographic material and credentials are protected from theft.
* **Continuous monitoring:** All activity is logged and analyzed in near real time.
* **Operational readiness:** Recovery procedures are documented and tested.

These principles ensure that control-plane compromise is difficult, detectable, and recoverable.

---

Control-Plane Services in Scope
-------------------------------

This layer applies to advanced control-plane protections affecting:

* Administrative roles and permissions
* System shell and appliance mode behavior
* Cryptographic key storage and secrets
* Logging, monitoring, and alerting systems
* Incident response and recovery mechanisms

These controls govern how administrators interact with the system at the deepest level.

---

Controls and Best Practices
---------------------------

Role-Based Access Control (RBAC)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Administrative privileges should be restricted according to job function.

Best practices include:

* Define granular administrative roles
* Separate configuration, monitoring, and audit duties
* Limit shell and advanced system access to approved users
* Review and recertify access regularly
* Remove unused or dormant accounts

RBAC reduces the risk of misuse or compromise of privileged accounts.

---

Appliance Mode Enforcement
~~~~~~~~~~~~~~~~~~~~~~~~~~

Appliance mode restricts BIG-IP behavior to appliance-like operation.

Best practices include:

* Enable appliance mode where operationally appropriate
* Restrict root shell and system-level access
* Prevent unauthorized modification of the underlying operating system
* Enforce stricter operational boundaries

Appliance mode reduces attack surface and limits post-compromise actions.

---

Cryptographic Protection of Secrets
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sensitive control-plane data must be protected from unauthorized access.

Best practices include:

* Deploy Hardware Security Modules (HSM) or SecureVault
* Protect SSL private keys and certificates
* Encrypt stored credentials for AAA and SNMP
* Secure API tokens and authentication secrets
* Implement credential rotation policies

Secret protection is critical for regulatory and compliance requirements.

---

Break-Glass Account Management
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An emergency administrative account must exist for recovery scenarios.

Best practices include:

* Create a dedicated break-glass account
* Secure credentials in an enterprise password vault
* Restrict use to emergency scenarios only
* Enable enhanced logging for break-glass access
* Test break-glass procedures regularly

Break-glass accounts ensure recoverability without weakening daily operations.

---

Continuous Monitoring and Threat Detection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All control-plane activity must be continuously monitored.

Best practices include:

* Forward all control-plane logs to a SIEM
* Monitor TMUI, SSH, and API access events
* Alert on suspicious login behavior
* Track configuration changes and privilege escalation
* Enable SNMP traps for system and security events

This enables early detection of compromise and misuse.

---

Incident Response and Recovery Planning
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Organizations must be prepared to respond to control-plane compromise.

Best practices include:

* Document incident response procedures
* Define steps for isolating affected systems
* Maintain automated configuration backups
* Establish credential revocation processes
* Perform periodic incident response exercises

Preparedness reduces downtime and limits damage during security incidents.

---

Validation with iHealth
-----------------------

Use F5 iHealth to verify core-layer controls.

Relevant diagnostics include:

* **D012815** – Appliance mode status
* Identification of weak credential storage practices
* Findings related to RBAC and privileged access
* Security Best Practices panel review

Upload a QKView and verify that advanced security findings are resolved.

---

Operational Considerations
--------------------------

When implementing core-layer controls:

* Coordinate with security, identity, and compliance teams
* Validate break-glass access before restricting normal admin access
* Test appliance mode in non-production environments first
* Ensure monitoring and alerting are operational before enforcement
* Document all role definitions and recovery procedures

Improper configuration at this layer can significantly impact operations if not planned carefully.

---

Expected Outcomes
-----------------

After implementing the core layer:

* Administrative access follows least-privilege principles
* Secrets and cryptographic materials are protected
* Appliance mode enforces system-level restrictions
* Continuous monitoring and alerting are active
* Incident response procedures are documented and tested
* iHealth reports minimal or no high-risk findings

This layer provides maximum protection for the control plane.

---

Next Steps
----------

After completing the core layer:

* Validate the entire Security Onion implementation using the operational checklist
* Complete the hands-on lab exercises
* Begin sustainment and continuous improvement activities

Continue to:


