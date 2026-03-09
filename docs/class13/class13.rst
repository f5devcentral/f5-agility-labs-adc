AI Data Delivery: Secure and Reliable Object Storage
========================================================

Welcome
-------

In this lab, attendees will be introduced to using F5 BIG-IP as an intelligent Application Delivery Controller (ADC) 
for MinIO AIStor.  

Objectives:
-----------

- Implement high availability and efficient load distribution for an S3 cluster
- Enforce fine-grained policy controls
- Migrate S3 buckets to new object storage cluster
- Route object storage traffic intelligently based on node and cluster health characteristics

Lab & Tasks:
------------

The next page (Introduction) will cover the lab environment, access, and lab variables. The lab will be using a shared AWS account where we have deployed the following resources:

- BIG-IP VE (LTM provisioned)
- MinIO Clusters

During the lab exercises we will explore different methods of protecting and exposing object storage services
that are included in the lab

.. toctree::
   :maxdepth: 2
   :caption: Labs:
   :glob:

   intro*
   labs/lab*
   close*
