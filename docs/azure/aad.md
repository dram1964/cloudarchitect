# AAD Features

## Key concepts

1. Identity - an object that can be authenticated: users, applications or servers
2. Account - an identity with associated data
3. Azure AD Account - an identity created through AAD
4. Azure tenant - an instance of Azure AD
5. Azure Subscription - used to pay for Azure cloud services. Each suscription is joined to a single tenant

Azure Active Directory is Microsoft's multi-tenant cloud-based directory and identity 
management service. AAD is used by Azure, Microsoft 365, Microsoft Intune and Microsoft Dynamics 365.
When a company purchases one of these services they are assigned a default directory. The default
directory will hold the users and groups with access to the services purchased. The tenant refers to 
the organisation and the default directory assigned to it, and therefore the tenant is also referred
to as the directory. 

Subscriptions are both a billing entity and a security boundary. 
Each subscription has a single account owner and is associated with a single 
AAD, although a single AAD instance may be used for many subscriptions.
Management Groups help you manage access, policy and compliance by grouping multiple subscriptions
together.

AAD provides:

- Secure SSO to cloud and on-premises applicatons
    - Users can sign-in with the same set of credentials to access all of their apps
    - SSO available for M365 and thousands of SaaS apps
- Ubiquitous device support
    - Company portals and web-based access can allow users to connect to apps using existing work credentials
- Secure remote access for on-prem web applications. Includes:
    - MFA
    - Conditional Access Policies
    - Group-based access management
- Cloud-extensibility
    - On-premises directories can be connected to AAD
    - AAD can then be used to manage access across environments
- Sensitive-data protection
    - identity protection available in P2 subscriptions
    - provides advanced security reporting, notifications, recommendations and risk-based policies
- Self-service options:
    - self-service password resets
    - delegation of group management to non-admin users

Active Directory Domain Services (AD DS) is the deployment of Windows Server-based Active Directory
on a physical or virtual server. Windows Active Directory is a suite of technologies. Apart from 
Directory Services, it also includes:

- Certificate Services (AD CS)
- Lightweight Directory Services (AD LDS)
- Federation Services (AD FS)
- Rights Management Services (AD RMS)

Using AAD is different from deploying an AD domain controller and adding it to your on-premises 
domain. AAD is primarily an identity solution and is designed for internet-based applications. AAD 
cannot be queried using LDAP: instead AAD provides a REST API over http/https. AAD does not use 
Kerberos/NTLM for authentication: instead it uses protocols such as SAML, WS-Federation, 
OpenID Connect for authentication and OAuth for authorisation. Federation Services allow use of 
third-party authentication services. There are no OUs or GPOs in AAD.

Azure Active Directory comes in four editions:

1. Free: User and Group management for up to 500,000 objects, SSO for Azure and Microsoft 365, Identity and Access Management, B2B Collaboration, on-prem directory synchronisation.
2. Microsoft 365 Apps: adds IAM for Microsoft 365 applications (which includes MFA, group access management and self-service password reset for cloud users)
3. Premium P1: adds Premium features, Hybrid Identities allowing users to access cloud or on-prem resources, Advanced Group Management (dynamic groups, self-service group management), Conditional Access, self-service password resets (SSPR) for on-prem users
4. Premium P2: adds Identity Protection (risk-based conditional access) and Identity Governance

All Azure AD editions support SSPR. SSPR uses a security group to limit which users have SSPR 
priviledges. All users accounts in your organisation require a valid licence to use SSPR. Azure Administrators can always reset their passwords. Authentication methods used in SSPR include:

1. email notification
2. text message
3. security code sent to a phone
4. security questions

# Azure AD Join

Azure AD Join is intended for organisations that do not have an on-prem AD.

Azure AD Join allows you to join devices directly to Azure AD without the need to join to an 
on-prem AD. It provides SSO, Enterprise state roaming across joined devices, access to an inventory
of pre-approved applications (Microsoft Store for Business), access to on-prem resources when the 
device also has access to on-prem domain controllers. 

Devices can be either:

1. registered 
    - provides the device with an identity used to authenticate the device
    - the device identity can be used to enable or disable the device
2. joined 
    - provides an identity and allows sign-in to the device using an organisational identity (AAD)
    - changes the local state of the device

# Users and Groups

AAD defines users in three ways:

- Cloud Identities: users defined in AAD only
- Directory-synchronised Identities: defined in an on-prem AD.
- Guest Users: accounts defined in other cloud providers

The `bulk create` option in the Azure Portal allows you to download a CSV template to use 
to add users in bulk.

Security groups can be assigned access to resources, and any members of the group will also 
inherit those permissions. Users can be assigned directly to the group or membership rules can be
defined so that users are assigned or removed from the group (dynamic user) or devices can be
assigned or removed from the group (dynamic device).

Administrative units can be defined in AAD and an administrative role can be created with 
administrator rights on the adminstrative unit. Individual users within the administrative unit
can then be assigned the adminstrative role to manage membership of the unit

Administrator roles are used to control who is allowed to do what. Member users are intended
for users who are considered internal to the organisation. Guest users are intended for 
individuals who are external to your organisation but need to access resources in your tenant
as part of a collaboration. Guest users sign-in using their own organisation accounts. 
Member users can invite Guest users unless this has been disabled by a User Administrator. 

If you have multiple tenants, you can use Guest accounts to give users from one tenant 
access to resources in another tenant. With Azure B2B collaboration, you don't have to 
manage external user identities - this is left to their originating organisation. B2B 
collaboration using Guest accounts is simpler than Federation. With Federation you need
to establish a trust relationship between different AD servers possibly using on-prem AD FS. 
Users authenticating outside the internal network will need to go through a Web Application 
proxy to authenticate to the AD FS server. 

Azure AD-roles are for managing access to AAD resources, like:

1. Users
2. Groups
3. Billing
4. Licencing
5. Application Registration

Access rights can be assigned directly to a user or group or assigned based on rules (requires a 
premium licence). 

# RBAC

Azure role-based access control (RBAC) roles are for managing access to Azure Resources and is built
on Azure Resource Manager. Roles can be assigned at the following scopes: 

1. Management Group
2. Subscription
3. Resource Group
4. Resource

RBAC is assigned from the IAM pane for each object scope. RBAC uses an 'allow' model - roles 
assigment defines what is allowed for the assignee. The definition of each role contains both
an 'Actions' and 'Not Actions' section. 

You can check role assignment activity in Activity Logs

