Outer Layer – Boundary and Access Isolation
===========================================

The Outer Layer of the Control-Plane Security Onion establishes the
first defensive boundary protecting the BIG-IP control plane. These
controls prevent unauthorized network-level access before authentication
mechanisms are engaged.

Executive Summary
-----------------

   The Outer Layer enforces strict boundary controls to ensure that only
   explicitly authorized networks can reach BIG-IP control-plane services.
   These protections align with Zero Trust principles, CIS benchmarks,
   and enterprise segmentation strategies.

Objective
---------

The objectives of the Outer Layer are to:

* Prevent unauthorized network access to the control plane
* Enforce strict management-plane isolation
* Minimize exposed control-plane services
* Establish deterministic administrative access paths
* Reduce attack surface prior to authentication

Threat Model
------------

The Outer Layer assumes adversaries may:

* Scan for exposed management interfaces
* Attempt access from adjacent or internet-facing networks
* Exploit misconfigured routing or segmentation
* Leverage lateral movement within internal environments

Controls at this layer focus on preventing initial reachability
of control-plane services.

Key Concepts
------------

The Outer Layer is guided by the following principles:

* **Network isolation:** Management traffic must be segregated from data-plane traffic.
* **Explicit allowlisting:** Only approved IP ranges may access management services.
* **Service exposure reduction:** Only required control-plane services are reachable.
* **Perimeter reinforcement:** Upstream firewalls reinforce segmentation.
* **Deterministic routing:** Administrative paths are predictable and controlled.

Control-Plane Services in Scope
-------------------------------

This layer governs network-level exposure of:

* TMUI (HTTPS administrative interface)
* SSH administrative access
* iControl REST and SOAP APIs
* SNMP management interfaces
* Self IP services on data-plane VLANs
* ConfigSync and HA communications

Controls and Best Practices
---------------------------

Management Interface Isolation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Administrative access should occur over a dedicated management network whenever possible.

Best practices include:

* Use the out-of-band management interface for administrative access
* Avoid exposing management services on production data-plane VLANs
* Restrict management interface access using IP allowlists
* Ensure management routes are not reachable from untrusted networks

Perimeter Firewall Reinforcement
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Upstream network devices should reinforce control-plane isolation.

While device-level protections are critical, perimeter firewalls
provide an additional enforcement layer.

Best practices include:

* Permit management access only from authorized administrative subnets
* Block management ports from internet-facing networks
* Log management-plane connection attempts
* Periodically review firewall rules for policy drift

.. note::

   In this module, firewall enforcement is treated as architectural
   guidance. Hands-on segmentation validation is performed using
   BIG-IP configuration and testing.

IP Allowlisting
~~~~~~~~~~~~~~~

Administrative access must be restricted by source IP.

Best practices include:

* Define allowlists for SSH and TMUI
* Restrict SNMP access to monitoring systems only
* Avoid broad internal access unless justified
* Validate unauthorized access attempts are blocked

Self IP Port Lockdown
~~~~~~~~~~~~~~~~~~~~~

Self IP Port Lockdown controls which services are reachable on each VLAN.

Even without virtual servers configured, Self IPs may respond to
control-plane services unless explicitly restricted.

Best practices include:

* Use **Allow None** on all production data-plane VLAN Self IPs
* Ensure no administrative services are exposed on DMZ or application VLANs
* Use **Allow Default** only on dedicated HA or isolated management VLANs
* Validate service exposure using network testing tools
* Confirm no unintended control-plane reachability exists

.. note::

   In advanced enterprise deployments, **Allow Custom** may be used to
   explicitly permit required services. However, for hardened production
   VLANs, a default-deny posture using **Allow None** is preferred.

.. warning::

   "Allow Default" enables multiple administrative services and should not be
   configured on internet-facing or shared production VLANs.

Control-Plane Segmentation Validation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Control-plane segmentation must be validated to ensure that
administrative services are reachable only through intended paths.

This validation confirms that:

* Management services are accessible only via the management interface
* Data-plane VLANs do not expose control-plane services
* No unintended lateral movement paths exist
* Administrative access paths are deterministic and controlled

For detailed configuration steps and validation exercises, see:

Route Domain and Segmentation Controls
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Network segmentation should prevent lateral movement toward the control plane.

Best practices include:

* Use route domains where appropriate
* Separate production and management routing contexts
* Limit administrative VLAN reachability
* Validate segmentation through testing

Validation with iHealth
-----------------------

Use F5 iHealth to verify boundary protections.

Relevant diagnostics include:

* Detection of exposed management services
* Management interface public access heuristics
* Security Best Practices panel findings

Upload a QKView and confirm that control-plane services are not broadly exposed.

Operational Considerations
--------------------------

When implementing Outer Layer controls:

* Coordinate firewall changes with network teams
* Validate management reachability before enforcement
* Avoid administrative lockout scenarios
* Document approved source networks and access paths
* Revalidate segmentation after major network changes

Expected Outcomes
-----------------

After implementing the Outer Layer:

* Control-plane services are reachable only from approved networks
* Internet exposure of management interfaces is eliminated
* Data-plane VLANs do not expose control-plane services
* Administrative access paths are documented and controlled
* Lateral movement toward the control plane is restricted

Next Steps
----------

Proceed to the Middle Layer to enforce strong authentication,
encryption hardening, and API access control.