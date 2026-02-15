References and Source Material
==============================

This guide is based on official F5 documentation, DevCentral guidance, and iHealth diagnostics.
The following references provide authoritative background and technical details for the
Control-Plane Security Onion framework.

These sources define best practices for securing BIG-IP management interfaces and validating
security posture.

---

Primary F5 Documentation
------------------------

* **Security Best Practices for F5 Products**  
  DevCentral article defining control-plane vs. data-plane security and foundational guidance.  

* **K13092 – Overview of securing access to the BIG-IP system**  
  Describes recommended methods for protecting BIG-IP management interfaces.  

* **K53108777 – Hardening your F5 system**  
  Comprehensive hardening guidance for BIG-IP platforms.  

* **K27404821 – Using F5 iHealth to diagnose vulnerabilities**  
  Explains how iHealth identifies security issues and vulnerabilities.  

* **K9970 – F5 Security Notifications and advisories**  
  Subscription and alerting for security bulletins and updates.  

---

Control-Plane Access and Account Security
-----------------------------------------

* **K5380 – Restricting SSH access to trusted IP addresses**  
* **K13309 – Restricting access to the Configuration utility (TMUI)**  
* **K17333 – Self IP port lockdown recommendations**  
* **K9908 – Configuring idle logout for TMUI sessions**  
* **K6068 – Configuring login banners**  
* **K13121 – Changing default passwords**  
* **K15497 – Configuring secure password policies**  
* **K15632 – Disabling or restricting the root account**  

These articles support the Outer Layer controls for access restriction and account hygiene.

---

Authentication and Protocol Hardening
-------------------------------------

* **K8811 – Configuring TACACS+ authentication**  
* **K11072 – Configuring LDAP and Active Directory authentication**  
* **K17403 – Configuring RADIUS authentication**  
* **K11719 – SSH brute-force mitigation**  
* **K80425458 – Hardening SSH ciphers and algorithms**  
* **K3122 – Configuring NTP servers**  
* **K10240 – NTP configuration best practices**  
* **K14120 – Preventing time-based attacks against BIG-IP**  

These references support the Middle Layer controls for enterprise authentication and protocol security.

---

Advanced Security and Appliance Mode
------------------------------------

* **K12815 – Enabling Appliance Mode**  
* **K12173 – Role-Based Access Control (RBAC)**  
* **K73034260 – Hardware Security Modules (HSM) and SecureVault**  
* **K11438344 – Control-plane incident response planning**  

These articles align with the Core Layer controls for least privilege, secret protection, and incident readiness.

---

iHealth Diagnostics and Validation
----------------------------------

* **F5 iHealth Portal** – https://ihealth.f5.com  
* **QKView Security Best Practices Panel documentation**  
* **D006068 – Login banner configuration diagnostic**  
* **D009908 – Idle logout configuration diagnostic**  
* **D012815 – Appliance mode diagnostic**  
* **D015632 – Admin and root account diagnostic**  
* **H444724 – Management interface public access heuristic**  
* **H494013 – Password policy strength heuristic**  
* **H726514 – NTP configuration heuristic**  

These diagnostics provide automated validation for the Security Onion framework.

---

DevCentral Community Resources
------------------------------

* DevCentral Security Best Practices articles  
* BIG-IP hardening blogs and whitepapers  
* Community lab guides and tutorials  
* F5 training and certification resources  

DevCentral serves as a knowledge-sharing platform for operational and security best practices.

---

Standards and Security Frameworks (Informational)
-------------------------------------------------

The Security Onion model aligns with common security frameworks, including:

* NIST SP 800-53 – Security and Privacy Controls  
* CIS Benchmarks – Network Device Hardening  
* Zero Trust Architecture principles  
* Defense-in-depth security strategy  

These frameworks reinforce the layered approach used in this guide.

---

Usage Notes
-----------

This guide consolidates and organizes existing F5 guidance into a layered control-plane
security framework.

Always refer to the latest official F5 documentation and security advisories for current
recommendations, software updates, and vulnerability information.

---

Next Steps
----------

Return to the operational checklist or continue through the lab exercises:


