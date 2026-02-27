Post-quantum Crypto (PQC) and F5
================================

Welcome
-------

Quantum computing is one of today's most exciting technological advancements, poised to transform industries with its unparalleled processing power. Unlike classical computers, which encode data using binary bits (zeroes and ones), quantum computers leverage quantum bits, or qubits. Qubits can exist in a superposition of zero and one, meaning they can be a zero, a one, or a combination of both at the same time. This capability enables quantum computers to solve certain types of problems exponentially faster than today's classical computers.

While quantum computing won't necessarily benefit everyday tasks like web surfing, its true strength lies in addressing complex, data-intensive challenges, such as accelerating medical and pharmaceutical research, optimizing AI algorithms, improving network designs, and advancing cryptography.

PQC's key benefits include future-proofing data security against powerful quantum computers, ensuring long-term data confidentiality (defeating "harvest now, decrypt later"), enabling seamless migration with hybrid modes, meeting regulatory compliance, and providing enhanced authentication and data protection for critical systems like finance, government, and communications, all while maintaining compatibility with current infrastructure.  

Cryptography is embedded into the core of the F5® Application Delivery and Security Platform (ADSP) via the F5 BIG-IP® Traffic Management Microkernel (TMM), a foundational element of BIG-IP solutions. 

F5 ADSP plays a pivotal role in network and application architectures by managing traffic between clients and servers. As a result, it is strategically positioned to facilitate PQC readiness and adapt seamlessly to evolving encryption standards.

NGINX provides PQC support using the Open Quantum Safe provider library for OpenSSL 3.x (oqs-provider). This library is available from the Open Quantum Safe (OQS) project. The oqs-provider library integrates the post-quantum algorithms from the OQS project into network protocols (e.g., TLS) for applications that use OpenSSL 3. All ciphers/algorithms provided by oqs-provider are supported by NGINX. 

[F5 Post-quantum cryptography readiness](https://www.f5.com/pdf/solution-overview/f5-pqc-readiness.pdf)

[Understanding PQC Standards and Timelines](https://www.f5.com/company/blog/understanding-pqc-standards-and-timelines) 

Objectives:
-----------

This lab provides hands-on experience with PQC across modern application delivery architectures. It will build a practical understanding of where PQC fits in the TLS stack, how it is implemented on F5 and NGINX, and how to validate the traffic behavior. By the end of the lab, you will be able to:

- Understand core PQC concepts and standards
- Configure PQC enabled TLS profiles on F5 BIG-IP for client-side encryption
- Explore PQC implementation in NGINX using OpenSSL and the Open Quantum Safe (OQS) provider
- Validate PQC cipher negotiation using Chrome developer tools

Lab & Tasks:
------------

The next page (Introduction) will cover the lab environment, access, and lab variables. The lab will be using a shared AWS account where we have deployed the following resources:

This lab environment contains three parts:

- BIG-IP Version 17.5.1.3.0.0.19
- NGINX Open Source Server
- Windows-client

During the lab exercises we will explore different methods of protecting and exposing object storage services
that are included in the lab

