Lab 3 - Red Team – Campaign Exploration

**Objective:** Review results from a pre-run Red Team campaign and
understand how to communicate findings.

**Note:** Red Team campaigns can take anywhere from 30 minutes to several
hours tocomplete and incur model usage costs.


For this lab, we will not be launching any new attack campaigns.
Instead, we’ve prepared a pre-run campaign so we can focus on reviewing and interpreting the
results without waiting for execution or consuming additional resources.



**Steps:**

1. Open the Red Team reports by clicking on the Reports in the
      navigation menu on theleft.

2. Observe that All Attack Campaign Run reports are displayed. At a quick glance 
   you will see the high level details including which campaign was run as
   well as the CASI score. Click into the ‘gpt-5-nano-azure-guardrails’ report.

3. This will bring up an overview of the results. Let’s investigate the
   details. Click on ‘View Raw Data’ in the upper right handcorner.

4. Here you will see all the details of every attack prompt that was
   sent against the target. Let’s just filter on the successful attacks. Click on *Filter*,
   and select *True*.

5. Inspect one failed attack chain — note why it succeeded.

6. Click the Download link to Export results (CSV.)

**Reflection:**

-  Which attack category had the highest failure rate? Why do you think
      that is?

-  Which findings would you escalate as “must-fix before production”
      vs.“track andmonitor”?
