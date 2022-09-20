# AAD Features

Azure Active Directory is Microsoft's multi-tenant cloud-based directory and identity 
management service. AAD is used by Azure, Microsoft 365, Microsoft Intune and Microsoft Dynamics 365.
When a company purchases one of these services they are assigned a default directory. The default
directory will hold the users and groups with access to the services purchased. The Tenant refers to 
the organisation and the default directory assigned to it. Subscriptions are both a billing entity
and a security boundary. Each subscription has a single account owner and is associated with a single 
AAD, although a single AAD instance may be used for many subscriptions.
Management Groups help you manage access, policy and compliance by grouping multiple subscriptions
together.

AAD provides a secure SSO to cloud and on-premises applicatons. On-premises 
directories can be connected to AAD. Self-service password resets available.
Tenants and AAD are closely linked: a Tenant is a single instance of AAD representing a single organisation.

Active Directory Domain Services (AD DS) is the deployment of Windows Server-based Active Directory
on a physical or virtual server. Windows Active Directory is a suite of technologies. Apart from 
Directory Services, it also includes:

- Certificate Services (AD CS)
- Lightweight Directory Services (AD LDS)
- Federation Services (AD FS)
- Rights Management Services (AD RMS)

Using AAD is different from deploying an AD domain controller and adding it to your on-premises 
domain. AAD is primarilly an identity solution and is designed for internet-based applications. AAD 
cannot be queried using LDAP: instead AAD provides a REST API over http/https. AAD does not use 
Kerberos for authentication: instead it uses protocols such as SAML, WS-Federation, OpenID Connect for 
authentication and OAuth for authorisation. Federation Services allow use of third-party authentication
services. There are no OUs or GPOs in AAD.

Azure Active Directory comes in four editions:

1. Free: User and Group management for up to 500,000 objects, SSO for Azure and Microsoft 365, Identity and Access Management, B2B Collaboration, on-prem directory synchronisation.
2. Microsoft 365 Apps: adds IAM for Microsoft 365, MFA, group access management and self-service password reset for cloud users.
3. Premium P1: adds Premium features, Hybrid Identities allowing users to access cloud or on-prem resources, Advanced Group Management (dynamic groups, self-service group management), Conditional Access, self-service password resets for on-prem users
4. Premium P2: adds Identity Protection and Identity Governance
Azure Administrators can always reset their passwords

# Azure AD Join

Azure AD Join allows you to join devices directly to Azure AD without the need to join to an 
on-prem AD. It provides SSO, Enterprise state roaming across joined devices, access to an inventory
of pre-approved applications (Microsoft Store for Business), access to on-prem resources when the 
device also has access to on-prem domain controllers. Devices can be either registered (provided 
with an identity in AAD) or joined (provides an identity and allows sign-in to the device 
using an organisational identity). Azure AD Join is intended for organisations that do not 
have an on-prem AD.

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
