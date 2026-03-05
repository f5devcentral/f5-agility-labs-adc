iHealth Integration and Validation
==================================

F5 iHealth provides automated diagnostics and best-practice analysis for BIG-IP systems.
Within the Control-Plane Security Onion framework, iHealth serves as the continuous validation
mechanism to verify that control-plane security controls are properly implemented and maintained.

This section explains how iHealth is used to assess, validate, and sustain control-plane security
across all security layers.

---

Objective
---------

The objectives of iHealth integration are to:

* Automatically detect control-plane security misconfigurations
* Identify vulnerabilities affecting management interfaces
* Validate implementation of Security Onion controls
* Provide continuous posture assessment
* Support audit and compliance reporting
* Enable early detection of indicators of compromise

iHealth transforms configuration review into a repeatable and automated security process.

---

Key Concepts
------------

iHealth operates by analyzing a QKView file collected from a BIG-IP system.

A QKView contains:

* System configuration data
* Software and hardware information
* Log excerpts
* Diagnostic command output
* Security-relevant settings

iHealth compares this data against:

* F5 Security Best Practices
* Known vulnerability patterns
* Platform and version-specific diagnostics
* Control-plane hardening guidance

The results are presented through diagnostics, heuristics, and the Security Best Practices panel.

---

iHealth Capabilities for Control-Plane Security
-----------------------------------------------

Security Best Practices Panel
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Security Best Practices panel provides a centralized view of security-related findings.

Capabilities include:

* Identification of control-plane misconfigurations
* Highlighting of high-risk management interface exposure
* Grouping of findings by severity
* Filtering diagnostics by security category
* Prioritization of remediation actions

This panel should be reviewed after every major configuration change.

---

Automated Control-Plane Diagnostics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

iHealth includes heuristics specifically aligned to control-plane security.

Key diagnostics include:

* **D006068** – Login banner configuration
* **D009908** – Idle logout settings (TMUI, SSH, console, CLI)
* **D012815** – Appliance mode status
* **D015632** – Admin and root account status
* **H444724** – Management interface public access
* **H494013** – Password policy strength
* **H726514** – NTP server configuration and reachability

These heuristics map directly to the Outer, Middle, and Core security layers.

---

Vulnerability and Exposure Detection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

iHealth identifies:

* Vulnerabilities specific to BIG-IP version and platform
* Insecure protocol configurations
* Weak authentication and access control practices
* Misconfigurations affecting management services
* Indicators of compromise within control-plane logs

This provides an early-warning system for emerging security risks.

---

Operational Workflow
--------------------

Collecting a QKView
~~~~~~~~~~~~~~~~~~~

Generate a QKView from the BIG-IP system:

* Via TMUI (Support > QKView)
* Via tmsh command line
* Via BIG-IQ automation (if deployed)

Ensure the QKView includes:

* Full configuration data
* Log files
* Diagnostic outputs

---

Uploading to iHealth
~~~~~~~~~~~~~~~~~~~~

Upload the QKView to:

https://ihealth.f5.com

After upload:

* Select the Security Best Practices panel
* Filter diagnostics by "Security_best_practices"
* Review control-plane related findings

---

Review and Remediation
~~~~~~~~~~~~~~~~~~~~~~

For each finding:

1. Identify the affected control-plane control (Outer, Middle, or Core layer)
2. Review the recommended remediation guidance
3. Implement corrective configuration changes
4. Re-collect a QKView
5. Re-upload and confirm findings are cleared

This creates a closed-loop validation cycle.

---

Mapping iHealth to Security Onion Layers
----------------------------------------

Outer Layer Validation
~~~~~~~~~~~~~~~~~~~~~~

Relevant diagnostics include:

* H444724 – Management interface public access
* D009908 – Idle logout settings
* D006068 – Login banner configuration
* D015632 – Admin and root account status
* H494013 – Password policy strength

These verify perimeter and credential hygiene controls.

---

Middle Layer Validation
~~~~~~~~~~~~~~~~~~~~~~~

Relevant diagnostics include:

* H726514 – NTP configuration and reachability
* Authentication and logging configuration findings
* Protocol hardening-related heuristics
* AAA integration warnings

These validate enterprise authentication and protocol security.

---

Core Layer Validation
~~~~~~~~~~~~~~~~~~~~~

Relevant diagnostics include:

* D012815 – Appliance mode status
* RBAC and privileged access findings
* Secret protection and credential storage diagnostics
* Monitoring and alerting gaps

These confirm advanced security posture and compliance readiness.

---

Automation and Continuous Assessment
------------------------------------

iHealth validation should be performed regularly.

Recommended cadence:

* After major configuration changes
* After software upgrades
* Quarterly security reviews
* After security incidents
* As part of compliance audits

Automation options include:

* Automated QKView collection via BIG-IQ
* Scheduled uploads
* Integration with configuration snapshot workflows
* Documentation of findings for audit purposes

This enables continuous control-plane security assessment.

---

Operational Considerations
--------------------------

When using iHealth for security validation:

* Do not upload QKViews containing sensitive data to unauthorized accounts
* Restrict access to iHealth results to security and operations teams
* Track findings over time to identify regression
* Correlate iHealth results with SIEM alerts and logs
* Maintain historical reports for compliance evidence

iHealth should complement, not replace, real-time monitoring systems.

---

Expected Outcomes
-----------------

With proper iHealth integration:

* Control-plane misconfigurations are detected early
* Security Onion controls are continuously validated
* Vulnerabilities are identified by platform and version
* Audit and compliance reporting is simplified
* Security posture improves over time

iHealth becomes the assurance mechanism for the entire framework.

---

Next Steps
----------

After integrating iHealth validation:

* Apply the Security Onion controls in each layer
* Complete the hands-on lab exercises
* Use the operational checklist to confirm full implementation
* Establish sustainment and review procedures

Continue to:


