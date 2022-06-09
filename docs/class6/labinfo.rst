Getting Started
---------------

You will be using our Unified Demo Framework (UDF) environment to complete the tasks in this lab.  All configuration can be completed from a Web browser.  Chrome is the preferred browser.

Lab Topology Diagram
--------------------
.. image:: images/image164.png

The following components have been included in your lab environment:

#. F5 BIG-IP VE (bigipA)

#. F5 BIG-IP VE (bigipB)

#. Windows Jumphost

#. Ubuntu LAMP Server

Lab Components
---------------

The following table lists the management IP addresses and credentials for all components:

.. list-table:: 
   :widths: auto
   :align: center
   :header-rows: 1

   * - Host
     - Management IP
     - username:password
   * - bigipA
     - 10.1.1.5
     - **admin**:admin.F5demo.com *and/or* **root**:default
   * - bigipB
     - 10.1.1.6
     - **admin**:admin.F5demo.com *and/or* **root**:default
   * - Windows Jumpbox
     - 10.1.1.4
     - **external_user**:admin.F5demo.com
   * - Ubuntu LAMP Server
     - 10.1.1.7
     - N/A; access via Webshell from UDF