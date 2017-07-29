Final Notes
===========

Here are a few helpful Solution Articles which helped us create this lab:

**SOL 13250 - Overview of Port Lockdown Behavior**

- TMOS v10.x - v11.x: https://support.f5.com/csp/article/K13250
- TMOS v12.x - v13.x: https://support.f5.com/csp/article/K17333

Even if you specify **Port Lockdown** as ``None`` this Solution states
the following: *ICMP traffic is always allowed, and if the BIG-IP
systems are configured in a redundant pair, ports that are listed as
exceptions are always allowed from the peer system.*

**SOL 13946 - Troubleshooting ConfigSync and device service clustering issues (11.X)**

https://support.f5.com/csp/article/K13946

Ever see “Awaiting Initial Sync,” “Sync Failure,” or “Changes
Pending” and don’t know how to go about resolving the issue? This is
a guide to keep handy if you’re running in HA mode with 11.X and
above. It will give you simple tests do diagnose the issues you will
face and how to resolve them.

**Version 13.0 Relase Notes**

https://support.f5.com/kb/en-us/products/big-ip_ltm/releasenotes/product/relnote-ltm-13-0-0.html

**New Built-in TCP Profiles, auto Nagle, auto Buffer tuning**

Default TCP profiles are becoming read only for "troubleshooting purposes."

If you are using the default TCP profile, it is not optimal and can be less
productive for your applications.  Please see:
https://devcentral.f5.com/articles/stop-using-the-base-tcp-profile

"When the latency penalty in using Nagle's algorithm is small, the feature
assembles data into full-size packets; when it is high, it stops doing that."

Instead of analyzing traffic captures and setting send buffer/receive buffer
sizes manually, customers can now select the auto option for send/receive buffer
sizes and high/low watermarks for the TCP Proxy buffers
