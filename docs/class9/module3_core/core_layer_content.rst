Core Layer – Privilege, Survivability, and System Integrity
============================================================

The Core Layer of the Control-Plane Security Onion represents the
deepest level of control-plane protection. These controls assume that
an attacker may have reached the management interface and focus on
restricting privilege, protecting sensitive material, and ensuring
operational survivability.

Executive Summary
-----------------

   The Core Layer enforces least privilege, protects cryptographic
   assets, constrains system-level access, and enables monitoring
   and recovery. These controls align with Zero Trust principles,
   NIST SP 800-53, and enterprise privileged access governance models.

Objective
---------

The objectives of the Core Layer are to:

* Enforce least-privilege administrative access
* Restrict system-level capabilities
* Protect cryptographic secrets and credentials
* Enable continuous monitoring and detection
* Prepare for incident response and recovery

Threat Model
------------

The Core Layer assumes an adversary may:

* Obtain valid administrative credentials
* Access the TMUI or SSH interface
* Attempt privilege escalation
* Attempt extraction of cryptographic material
* Modify system configuration for persistence

Controls at this layer focus on limiting blast radius and ensuring
rapid detection and recovery.

Key Concepts
------------

The Core Layer is guided by the following principles:

* **Least privilege:** Administrators receive only required capabilities.
* **Capability restriction:** System-level access is tightly controlled.
* **Secret protection:** Cryptographic material is safeguarded.
* **Monitoring and detection:** Administrative activity is logged and analyzed.
* **Operational resilience:** Recovery mechanisms are documented and tested.

Control-Plane Services in Scope
-------------------------------

This layer applies to deep administrative and system-level controls:

* Role-Based Access Control (RBAC)
* System shell and appliance mode enforcement
* Cryptographic key storage and protection
* Break-glass account governance
* Logging, monitoring, and SIEM integration
* Incident response and recovery readiness

Controls and Best Practices
---------------------------

Role-Based Access Control (RBAC)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Administrative privileges should be restricted according to job function.

Best practices include:

* Define granular administrative roles
* Separate configuration, monitoring, and audit duties
* Limit advanced system and shell access
* Periodically review and recertify role assignments
* Remove unused or dormant accounts

RBAC limits privilege escalation and reduces insider risk.

Appliance Mode Enforcement
~~~~~~~~~~~~~~~~~~~~~~~~~~

Appliance mode restricts BIG-IP behavior to controlled operational boundaries.

Best practices include:

* Enable appliance mode where operationally appropriate
* Restrict root shell access
* Prevent modification of the underlying operating system
* Validate compatibility before enabling in production

Appliance mode reduces post-compromise capability.

Cryptographic Protection of Secrets
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sensitive control-plane data must be protected from unauthorized access.

Best practices include:

* Deploy Hardware Security Modules (HSM) or SecureVault
* Protect SSL private keys and certificates
* Encrypt stored credentials for AAA and SNMP
* Secure API tokens and authentication secrets
* Implement credential rotation policies

Secret protection is critical for regulatory compliance and risk reduction.

Break-Glass Account Management
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An emergency administrative account must exist for recovery scenarios.

Best practices include:

* Create a dedicated break-glass account
* Secure credentials in an enterprise password vault
* Restrict use to documented emergency procedures
* Enable enhanced logging for break-glass usage
* Test break-glass procedures regularly

Break-glass accounts ensure operational continuity.

Continuous Monitoring and Threat Detection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Administrative activity must be continuously monitored.

Best practices include:

* Forward control-plane logs to a centralized SIEM
* Monitor TMUI, SSH, and API access events
* Alert on suspicious authentication patterns
* Track configuration changes and privilege escalation
* Enable SNMP traps for system and security events

Monitoring ensures early detection of misuse or compromise.

Incident Response and Recovery Planning
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Organizations must prepare for control-plane compromise.

Best practices include:

* Document incident response procedures
* Define system isolation steps
* Maintain automated configuration backups
* Establish credential revocation processes
* Conduct periodic response exercises

Preparedness reduces downtime and limits operational impact.

Validation with iHealth
-----------------------

Use F5 iHealth to verify Core Layer controls.

Relevant diagnostics include:

* **D012815 – Appliance mode status**
* Findings related to RBAC configuration
* Privileged account diagnostics
* Security Best Practices panel review

Upload a QKView and confirm that high-risk privilege findings are resolved.

Operational Considerations
--------------------------

When implementing Core Layer controls:

* Coordinate with identity and security governance teams
* Validate access continuity before restricting privileges
* Test appliance mode in non-production environments
* Ensure logging pipelines are operational before enforcement
* Document all role definitions and recovery procedures

Improper configuration at this layer may disrupt administrative operations.

Expected Outcomes
-----------------

After implementing the Core Layer:

* Administrative access follows least-privilege principles
* System-level capabilities are constrained
* Cryptographic materials are protected
* Monitoring and alerting are operational
* Incident response procedures are validated
* iHealth reports minimal high-risk privilege findings

This layer ensures survivability even if earlier defenses fail.

Next Steps
----------

Return to the Operational Security Checklist
or proceed to hands-on lab validation exercises.
