Roadmap
=======

This roadmap provides a structured implementation sequence for
Control-Plane Security Onion deployment.

Implementation Order
--------------------

1. Outer Layer – Establish boundary protections
2. Middle Layer – Enforce authentication and encryption controls
3. Core Layer – Apply least-privilege, service exposure reduction,
   and monitoring controls

Each layer should be validated before proceeding to the next.

Deployment Phases
-----------------

Phase 1 – Boundary Hardening  
Isolate management access and restrict network reachability.

Phase 2 – Identity and Encryption Enforcement  
Integrate centralized AAA, enforce MFA, and harden TLS configurations.

Phase 3 – Control-Plane Surface Reduction  
Apply RBAC refinement, Self IP Port Lockdown, appliance mode,
and logging integration.

Phase 4 – Operational Validation  
Validate configurations using iHealth and perform access testing.

Operational Readiness Checkpoints
----------------------------------

Before advancing layers:

* Confirm administrative access continuity
* Validate HA and ConfigSync stability
* Test break-glass recovery
* Verify logging and alerting functionality

Maturity Model
--------------

Basic – Outer Layer only  
Intermediate – Outer + Middle  
Advanced – Full Outer + Middle + Core  

Enterprise environments should implement all three layers.
