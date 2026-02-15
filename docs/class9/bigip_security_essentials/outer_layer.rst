Outer Layer – Network Access Controls
====================================

The outer layer of the Control-Plane Security Onion focuses on the fastest and most effective
controls for reducing risk to the BIG-IP control plane.

These controls are:

* Easiest to implement
* Lowest maintenance
* Highest immediate security impact

This layer establishes a hardened perimeter around management and administrative interfaces.

---

Objective
---------

The objectives of the outer layer are to:

* Prevent unauthorized network access to the control plane
* Limit exposure of management services
* Enforce automatic session termination
* Eliminate default credentials and weak authentication practices

Successful implementation of this layer significantly reduces the attack surface of TMUI,
SSH, and iControl interfaces.

---

Key Concepts
------------

The outer layer is based on three fundamental principles:

* **Restrict access:** Only trusted networks and systems can reach management interfaces.
* **Enforce time limits:** Idle sessions are automatically terminated.
* **Harden credentials:** Default and weak passwords are eliminated.

These controls provide foundational security that all environments should implement,
regardless of deployment size or industry.

---

Control-Plane Services in Scope
-------------------------------

This layer applies to the following control-plane services:

* Configuration Utility (TMUI)
* SSH and tmsh access
* iControl REST and SOAP APIs
* Synchronization and monitoring services (big3d, bigd)

These services must never be exposed to untrusted or public networks.

---

Controls and Best Practices
---------------------------

Restrict Management Interface Access
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Management interfaces must be reachable only from trusted IP ranges.

Best practices include:

* Restrict TMUI access by source IP
* Restrict SSH access by source IP
* Configure the management interface firewall (BIG-IP 14.1+)
* Avoid exposing management interfaces to the internet
* Use a dedicated management interface or management VLAN
* Isolate management traffic from data-plane traffic

Self IP Port Lockdown
~~~~~~~~~~~~~~~~~~~~~

Self IP addresses should be configured with a default-deny posture.

Best practices include:

* Set Self IPs to **Lockdown None** by default
* Explicitly allow only required services (for example, big3d on port 4353)
* Use **Allow Default** only for dedicated management or HA VLANs
* Restrict access to specific ports whenever possible

This prevents unintended exposure of control-plane services.

---

Session and Access Timeouts
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Idle administrative sessions increase the risk of unauthorized access.

Configure the following:

* TMUI idle logout
* SSH inactivity timeout
* CLI (tmsh) session timeout
* Serial console session timeout
* BIG-IQ idle session timeout (if applicable)

These controls ensure sessions are automatically terminated when inactive.

---

Login Banners
~~~~~~~~~~~~~

Login banners provide legal notice and inform users that activity is monitored.

Configure:

* Pre-login banner (legal and security notice)
* Post-login banner (maintenance or policy message)

Banners should state that all control-plane activity is logged and subject to monitoring.

---

Account and Password Hygiene
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Eliminate default and weak credentials.

Best practices include:

* Change default admin and root passwords
* Disable or restrict root account where policy allows
* Enforce password complexity requirements
* Enable password history
* Configure maximum failed login thresholds
* Apply policy consistently to all administrative accounts

Strong password hygiene is critical for control-plane protection.

---

Validation with iHealth
-----------------------

Use F5 iHealth to verify outer-layer controls.

Relevant diagnostics include:

* **H444724** – Management interface public access
* **D009908** – Idle logout settings
* **D006068** – Login banner configuration
* **D015632** – Admin and root account status
* **H494013** – Password policy strength

Upload a QKView and review the Security Best Practices panel to confirm findings are cleared.

---

Operational Considerations
--------------------------

When implementing outer-layer controls:

* Always confirm an alternate access path before applying restrictions
* Test changes from an approved management network
* Document approved IP ranges and access policies
* Coordinate with operations teams before restricting services
* Avoid making changes during production hours without change control

Loss of management access can require console or break-glass recovery procedures.

---

Expected Outcomes
-----------------

After implementing the outer layer:

* Management interfaces are no longer exposed to untrusted networks
* Idle sessions are automatically terminated
* Default credentials are eliminated
* Password policies are enforced
* iHealth shows no critical outer-layer findings

This establishes a secure foundation for higher-layer controls.

---

Next Steps
----------

After completing the outer layer:

* Proceed to the middle layer for enterprise authentication and protocol hardening
* Begin hands-on implementation using the lab exercises

Continue to:


