# Security Orchestration, Automation and Response (SOAR)

Notes from the SANS Institute Whitepaper: 'Learn How SOAR Helps You Streamline Security While Improving Your Defenses Against Cyber Attacks', Cristopher Crowley, Temi Adebambo, August 2022


Cybersecurity deals with systems that are behaving unexpectedly and deviating from intended or 
authorised actions. SOAR tools are designed to help cyber security professionals develop 
automations to handle these events. 

Cybersecurity staff need to:

- verify signals requiring a response
- collect data from multiple systems
- aggregate related data
- perform repeatable tasks and 
- work through complicated sequences to make the correct decisions and take appropriate actions

## Workflow Development with SOAR

SOAR is intended to help improve performance accuracy and precision to cybersecurity operations. 
A SOAR should provide staff with repetitive and repeatable tasks (workflows) to respond to security 
incidents. Workflows will consist of modules: interconnected sequences of actions. Workflows 
are developed by identifying possible scenarios and responses (use cases), and 
identifying what supporting information is required in each scenario. Workflows can be tested
to identify any additional requirements. The workflow can then be added to the SOAR tool. 
Workflows can be updated by working through additional use cases. 

The SOAR tool develops into a knowledge management system through the repetition and 
workflow enhancement cycle. 

Development of alerting systems involves use case development or detection engineering. The workflow
development pipeline will typically involve:

- select a scenario of interest and decompose to a story line
- this should result in one or more use cases
- identify data artifacts and detection opportunities
- data enrichment through correlation with related internal or external (threat intelligence) data
- implement detections in systems
- perform assessment, review, revision or retirement

Use case development and detection engineering are fundamental in security operations because they
maximise true positive alerts. 

DevSecOps is an important to instill confidence in the automation provided by SOAR tools, and 
involves the implementation of rigourous testing. 

## Definitions

- Security
    - the restriction of a system to its intended use and
    - the protection of the confidentiality, integrity and availability of that system
- Orchestration
    - the automated configuration, coordination and management of computer systems and software
- Automation
    - the technology by which a process or procedure is performed with minimal human assistance
- Response
    - initial verification
    - short-term mitigation
    - fundamental root-cause analysis
    - business continuity in the face of actual incidents
    - long term enhancements to systems to be more resilient to issues
