References and Source Material
==============================

This guide consolidates official F5 documentation, DevCentral guidance,
and iHealth diagnostics into a layered Control-Plane Security Onion framework.

All recommendations in this guide are derived from authoritative F5
knowledge articles, product documentation, and industry security standards.

.. admonition:: Executive Context
   :class: important

   Always consult the latest F5 security advisories and product documentation
   before implementing hardening controls in production environments.

Primary F5 Documentation
------------------------

The following resources define foundational control-plane security guidance:

* **Security Best Practices for F5 Products**  
  Defines control-plane versus data-plane security boundaries and foundational hardening guidance.

* **K13092 – Overview of securing access to the BIG-IP system**  
  Recommended approaches for protecting BIG-IP management interfaces.

* **K53108777 – Hardening your F5 system**  
  Comprehensive platform hardening guidance.

* **K27404821 – Using F5 iHealth to diagnose vulnerabilities**  
  Overview of security diagnostics and vulnerability detection.

* **K9970 – F5 Security Notifications and Advisories**  
  Subscription guidance for vulnerability alerts and software updates.

Outer Layer – Boundary and Access Isolation
-------------------------------------------

These articles support network-level access control and management-plane isolation:

* **K5380 – Restricting SSH access to trusted IP addresses**
* **K13309 – Restricting access to the Configuration utility (TMUI)**
* **K17333 – Self IP port lockdown recommendations**
* **K9908 – Configuring idle logout for TMUI sessions**
* **K6068 – Configuring login banners**
* **K13121 – Changing default passwords**
* **K15497 – Configuring secure password policies**
* **K15632 – Disabling or restricting the root account**

These references reinforce explicit allowlisting and account hygiene controls.

Middle Layer – Authentication and Protocol Hardening
----------------------------------------------------

These references support centralized authentication and encryption controls:

* **K8811 – Configuring TACACS+ authentication**
* **K11072 – Configuring LDAP and Active Directory authentication**
* **K17403 – Configuring RADIUS authentication**
* **K11719 – SSH brute-force mitigation**
* **K80425458 – Hardening SSH ciphers and algorithms**
* **K3122 – Configuring NTP servers**
* **K10240 – NTP configuration best practices**
* **K14120 – Preventing time-based attacks against BIG-IP**

These documents support enterprise identity integration and protocol hardening.

Core Layer – Privilege, Survivability, and System Integrity

These references align with advanced least-privilege and recovery mechanisms:

* **K12815 – Enabling Appliance Mode**
* **K12173 – Role-Based Access Control (RBAC)**
* **K73034260 – Hardware Security Modules (HSM) and SecureVault**
* **K11438344 – Control-plane incident response planning**

These materials provide guidance on restricting administrative capability
and preparing for compromise scenarios.

iHealth Diagnostics and Validation
----------------------------------

iHealth provides automated analysis of BIG-IP configuration state.

Portal:

* **F5 iHealth Portal** – https://ihealth.f5.com

Relevant diagnostics referenced in this guide include:

* **D006068 – Login banner configuration diagnostic**
* **D009908 – Idle logout configuration diagnostic**
* **D012815 – Appliance mode diagnostic**
* **D015632 – Admin and root account diagnostic**
* **H444724 – Management interface public access heuristic**
* **H494013 – Password policy strength heuristic**
* **H726514 – NTP configuration heuristic**

These diagnostics support validation of Security Onion implementation.

DevCentral Community Resources
------------------------------

DevCentral provides community-driven operational knowledge:

* Security best-practice articles
* Hardening blogs and whitepapers
* Community lab guides
* F5 training and certification resources

DevCentral supplements official documentation with operational experience.

Industry Standards and Frameworks
----------------------------------

The layered approach in this guide aligns with widely adopted security frameworks:

* **NIST SP 800-53 – Security and Privacy Controls**
* **NIST SP 800-207 – Zero Trust Architecture**
* **CIS Benchmarks – Network Device Hardening**
* **NIST SP 800-61 – Incident Response Guide**
* Defense-in-depth architectural principles

These frameworks reinforce segmentation, least privilege, and monitoring
as foundational security controls.

Usage Notes
-----------

This guide organizes existing F5 guidance into a structured,
layered control-plane security framework.

Security recommendations evolve over time. Always:

* Review current F5 advisories
* Apply software updates promptly
* Revalidate configurations after major changes
* Perform periodic security reviews

Next Steps
----------

Return to:

* The Operational Security Checklist
* The relevant Security Onion layer
* Hands-on lab exercises for validation
