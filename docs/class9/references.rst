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

* `**K13092 – Overview of securing access to the BIG-IP system**  <https://my.f5.com/manage/s/article/K13092>`
  Recommended approaches for protecting BIG-IP management interfaces.

* `**K53108777 – Hardening your F5 system**  <https://my.f5.com/manage/s/article/K53108777>`
  Comprehensive platform hardening guidance.

* `**K27404821 – Using F5 iHealth to diagnose vulnerabilities**  <https://my.f5.com/manage/s/article/K27404821>`
  Overview of security diagnostics and vulnerability detection.

* `**K9970 – F5 Security Notifications and Advisories**  <https://my.f5.com/manage/s/article/K9970>`
  Subscription guidance for vulnerability alerts and software updates.

Outer Layer – Boundary and Access Isolation
-------------------------------------------

These articles support network-level access control and management-plane isolation:

* `**K5380 – Restricting SSH access to trusted IP addresses**  <https://my.f5.com/manage/s/article/K5380>`
* `**K13309 – Restricting access to the Configuration utility (TMUI)**  <https://my.f5.com/manage/s/article/K13309>`
* `**K17333 – Self IP port lockdown recommendations**  <https://my.f5.com/manage/s/article/K17333>`
* `**K9908 – Configuring idle logout for TMUI sessions**  <https://my.f5.com/manage/s/article/K9908>`
* `**K6068 – Configuring login banners**  <https://my.f5.com/manage/s/article/K6068>`
* `**K13121 – Changing default passwords**  <https://my.f5.com/manage/s/article/K13121>`
* `**K15497 – Configuring secure password policies**  <https://my.f5.com/manage/s/article/K15497>`
* `**K15632 – Disabling or restricting the root account**  <https://my.f5.com/manage/s/article/K15632>`

These references reinforce explicit allowlisting and account hygiene controls.

Middle Layer – Authentication and Protocol Hardening
----------------------------------------------------

These references support centralized authentication and encryption controls:

* `**K8811 – Configuring TACACS+ authentication**  <https://my.f5.com/manage/s/article/K8811>`
* `**K11072 – Configuring LDAP and Active Directory authentication**  <https://my.f5.com/manage/s/article/K11072>`
* `**K17403 – Configuring RADIUS authentication**  <https://my.f5.com/manage/s/article/K17403>`
* `**K11719 – SSH brute-force mitigation**  <https://my.f5.com/manage/s/article/K11719>`
* `**K80425458 – Hardening SSH ciphers and algorithms**  <https://my.f5.com/manage/s/article/K80425458>`
* `**K3122 – Configuring NTP servers**  <https://my.f5.com/manage/s/article/K3122>`
* `**K10240 – NTP configuration best practices**  <https://my.f5.com/manage/s/article/K10240>`
* `**K14120 – Preventing time-based attacks against BIG-IP**  <https://my.f5.com/manage/s/article/K14120>`

These documents support enterprise identity integration and protocol hardening.

Core Layer – Privilege, Survivability, and System Integrity

These references align with advanced least-privilege and recovery mechanisms:

* `**K12815 – Enabling Appliance Mode**  <https://my.f5.com/manage/s/article/K12815>`
* `**K12173 – Role-Based Access Control (RBAC)**  <https://my.f5.com/manage/s/article/K12173>`
* `**K73034260 – Hardware Security Modules (HSM) and SecureVault**  <https://my.f5.com/manage/s/article/K73034260>`
* `**K11438344 – Control-plane incident response planning**  <https://my.f5.com/manage/s/article/K11438344>`

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

* `**NIST SP 800-53 – Security and Privacy Controls**  <https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final>`
* `**NIST SP 800-207 – Zero Trust Architecture**  <https://csrc.nist.gov/pubs/sp/800/207/final>`
* `**CIS Benchmarks – Network Device Hardening**  <https://www.cisecurity.org/benchmark/network_devices/>`
* `**NIST SP 800-61 – Incident Response Guide**  <https://csrc.nist.gov/publications/detail/sp/800-61/rev-1/final>`
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
