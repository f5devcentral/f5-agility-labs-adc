Lab 1: Prompt and Response Scanning
==========================================================================================

Business problem. You will learn how to protect AI applications from various prompt attacks, such as jailbreaks, prompt injections, and data exfiltration attempts. 
These attacks can lead to unauthorized access, data breaches, and compromised AI model integrity

Following the tasks in the prior **Introduction** Section, you should now be able to access the
UDF lab environment and have received an email from the Calypso SaaS platform.

Task 1: Create project
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After logging into the platform you access the Dashboard. From here click on Projects on the left-hand menu.

.. image:: ../_static/lab1-dashboard.png
   :align: center
   :alt: Dashboard

The create project dialog will appear. 
Click the radio button to the left of CalypsoAI Chat and click the Create button.

.. image:: ../_static/lab1-create-project.png
   :align: center
   :alt: Create Project

Create the new chat project with the following details:

========================== ======================== 
Chat Name                  Model                    
========================== ======================== 
**First inital Last name** genai-azure-openai         
========================== ======================== 
      
.. image:: ../_static/lab1-chat.png
   :align: center
   :alt: Chat Setup

You should now be in your new chat project.

.. image::_static/lab1-chat-project.png
   :align: center
   :alt: Chat Project

Task 2: Real-time protection for prompts and responses
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Here you will be sending prompts to the model you just connected with-in your project.  
These prompts will be of the safe and unsafe varieties. You will observe the results and explore the logs regarding those prompts.
