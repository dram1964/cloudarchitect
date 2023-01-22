# Security

## Threat Modeling
The Threat Priority Model can be used to identify your threat priorities. It starts by identifying the Impact of a threat occuring on a resource, then identifies the probability
of this happening by defining the vulnerability and identifying any existing counter-measures (mitigations). The Impact and Probability together define the Threat Priority
## Zero Trust
The attack chain (or kill chain) begins with a compromised account that is then used to gain elevated 
priviledges to mount an attack. Zero Trust uses the principle of <i>Never Trust: Always Verify</i>
## Defence In Depth

A defence-in-depth strategy addresses security at the following layers:

    1. Physical Security - protecting access to buildings and hardware
    2. Identity and Access - authentication and authorisation controls
    3. Perimeter - protecting the network perimeter using DDoS protection and firewalls
    4. Network - limiting communication between and to resources
    5. Compute - secure access to VMs and endpoints
    6. Application - securing applications, remediating vulnerabilities
    7. Data - securing access to data



- Microsoft Defender for Cloud (previously Azure Security Centre and Azure Defender)
    - Monitor security posture for Azure and on-prem resources. Security Posture (<strong>CIA</strong>):
        - Confidentiality - sensitive data must be kept protected and accessed only by those who should have access through the principle of least priviledge
        - Integrity - confidence that data has not been altered or tampered with
        - Availability - data and systems should be available to those that need them
    - Automatically apply security settings to new resources
    - Provides recommendations
    - Continuously monitor and assess vulnerabilities
    - Block malware with machine learning
    - Define rules for allowed applications (adaptive application controls)
    - Threat detection
    - Just-in-time access control for network ports
    - Secure score reflects compliance to assigned governance controls
    - The Regulatory Compliance dashboard provides overall compliance score and the number of failing assessments
    - Adaptive network hardening monitors network activity compared to NSGs and makes recommendations
    - File integrity monitoring - monitor important files
    - Use Workflow Automation (Logic Apps) to respond to security alerts
  
- Microsoft Sentinel (previously Azure Sentinel)
    - Microsoft Managed (SaaS), cloud-based Security Information and Event Management (SIEM) and Security Orchestration, Automation and Response ([SOAR](/operations/soar)) system
    - Aggregates security data from many different sources, using open-standard logging format
    - Threat detection uses built-in rules (AI) provided as templates or custom rules
    - Threat response can be automated with Azure Monitor Workbooks, to set alerts or send emails. Email will include link to Block or Ignore threat
    - Utilises Microsoft's analytics and threat intelligence
    - Several methods exist for connecting security data sources to Sentinel, including native support for M365, AAD, Syslog, CEF, REST
    - Data is stored in Log Analytics
  
- Azure Key Vault
    - Secure, centralised storage for application secrets
    - Monitor and control access to secrets
    - Simplify management and renewal of certificates
    - Integrates with other Azure services - storage accounts, container registries, event hubs, etc
    - Available in Standard and Premium SKUs
    - Premium adds support for HSM-protected Keys
    - Resources must be in the same region and subscription to access key vault secrets
  
- Azure Dedicated Host
    - VMs hosted on physical servers that are not shared with other customers
    - Host groups provide for high availability
    - Charge is per dedicated host - not per VM running on the host
  
Further Reading:

- <a href="https://docs.microsoft.com/en-us/learn/modules/resolve-threats-with-azure-security-center/" target="#">Resolve Security Threats</a>
- <a href="https://docs.microsoft.com/en-us/learn/modules/design-monitoring-strategy-on-azure/" target="#">Security Monitoring Plan</a>
- <a href="https://docs.microsoft.com/en-us/learn/modules/manage-secrets-with-azure-key-vault/" target="#">Manage App Secrets</a>
- <a href="https://docs.microsoft.com/en-us/learn/modules/configure-and-manage-azure-key-vault" target="#">Manage Key Vault Secrets</a>
- <a href="https://docs.microsoft.com/en-us/learn/paths/implement-resource-mgmt-security/" target="#">Implement Resource Management Security</a>
- <a href="https://docs.microsoft.com/en-us/learn/paths/secure-your-cloud-data/" target="#">Secure Cloud Data</a>
- <a href="https://docs.microsoft.com/en-us/learn/paths/az-400-develop-security-compliance-plan/" target="#">Develop a Security and Compliance Plan</a>
- <a href="https://docs.microsoft.com/en-us/learn/paths/manage-security-operations/" target="#">Manage Security Operations in Azure</a>

