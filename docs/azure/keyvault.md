# Key Vault

## Key Concepts
[Key Vault Documentation](https://learn.microsoft.com/en-us/azure/key-vault/)

Azure Key Vault provides secure storage containers called *(vaults)* for 
managing application secrets. Key Vault supports certificate management. 

Each Key Vault is a collection of cryptographic keys and cryptographically
protected data *(secrets)*, representing logical groups that need to be
managed together. 

Secret access and vault management is accomplished via a REST API that is
supported by all Azure management tools and client libraries. 

Keys are cryptographic assets stored in key vaults. Keys are accessed 
indirectly by applications calling cryptography methods on the Key
Vault and the Key Vault performs the requested operation internally. 
Applications do not have direct access to the keys. Key Vault supports
assymetric keys *RSA 2048*. 

Key Vault supports both Hardware-protected and Software-protected keys:

1. Hardware-protected keys use HSMs (hardware security modules) for processing
and key generation. All actions are confined to the HSM boundary. HSM keys
can also be imported from your own HSMs - known as *bring your own key* 
or BYOK. The Microsoft Azure Dedicated HSM service can also be used to 
protect HSM-protected applications. 
2. Software-protected keys use software-based algorithms: RSA and ECC. Keys 
are encrypted at rest using HSMs and remain isolated from the application. 

Hardware-protected keys provide FIPS 140-2 Level 2 assurance and are 
therefore recommended for production environments. HSM-protected keys
require a Premium SKU for the Key Vault.

Azure services integrate directly with Azure Key Vault and can decrypt
secrets without knowledge of the encryption keys. Secrets are encrypted
when added to a Key Vault and decrypts them automatically when you 
read them. The encryption key used is unique to each Key Vault.

Secrets are small data blobs (less than 10K) protected by HSM-generated
key created when the Key Vault is created. Secrets are used to protect
sensitive application-data

Key Vault also provides certificate services to provision, manage and 
deploy public and private SSL/TLS certificates. Certificates can be 
requested and renewed through partnerships with Certificate Authorities. 

Key Vault is designed to store configuration settings for server
applications. User data should be stored elsewhere, such as in 
Storage Accounts or Azure SQL Database. Secrets used by your application 
to access that data can be stored in Key Vault. 

## Best Practice

Use RBAC to grant access to Key Vaults. Access to Key Vaults is controlled
at two different levels:

1. Management Plane - access to Key Vault properties but not Key Vault 
data
2. Data Plane - controls access to the Key Vault data but not the Key Vault configuration

Both the Management and Data Plane use AAD for authentication. The 
Management Plane uses RBAC for authorisation and the Data Plane uses 
Access Policies (or the newly added RBAC) for authorisation. Once a 
user or application authenticates, they aquire a token for the 
resouce endpoint in the corresponding plane. The token can be used to 
access a resource in the Key Vault using a REST request. 

Users with Contributor access to the Key Vault can grant themselves access
to the Data Plane by setting a Key Vault access policy. Contributor access
to Key Vaults should be tightly controlled. To restrict access to just the 
managment plane, it is recommended that the **Key Vault Contributor** role
is used instead. Additionally, the **Key Vault deploy** role allows 
deployed VMs to access secrets from the Key Vault.

Reading and Writing data to the Key Vault is controlled via the Key Vault 
Access Policy. The Access Policy assigns a permission-set to an identity. 

Network access can be restricted to:

1. Specific Azure Vnets
2. Specific IP Addresses
3. Trusted Microsoft Services

Use separate Key Vaults per application per environment. The names for each 
secret can be the same across environments, meaning that developers will only 
need to change the vault URL in the application to access secrets across 
environments. 

Turn on logging and alerts for your Key Vaults

Apps and Users authenticate to Key Vault using an AAD token. To get a AAD token
requires the app or user to hold a secret or certificate. To avoid having to 
hold this secret/certificate outside of Key Vault, it is recommended to use a 
system-assigned managed-identity to authenticate to Key Vault. When you enable
a system-assigend managed-identity on your app, Azure activates a token-granting
REST service specifically for the app. The App Service requires a secret to 
access the REST service, but this is integrated into the App Service environment. 
Managed identities also get registered in AAD when the resource is created, 
and are automatically deleted when the resource is deleted or the managed 
identity is disabled.

[Azure Key Vault Developers Guide](https://learn.microsoft.com/en-us/azure/key-vault/general/developers-guide)



## Manage Certificates

Key Vault can be used to manage X.509 certificates from various sources. 
Self-signed certificates can be generated directly in the Portal. This 
process generates a public/private key pair and signs the Certificate
with its own key. 

Alternatively you can generate a X.509 certificate signing request (CSR). 
This creates a public/private key-pair and a CSR. The CSR can be passed 
to a CA. The signed Certificate returned by the CA can then be merged with 
the stored key-pair to finalise the certificate in the Key Vault. 

You can also connect your Key Vault with a trusted certificate issuer. You
can then request to create a certificate and the Key Vault will interact
directly with the CA to fulfill the request. The returned certificate is
merged with the Key Vault key-pair. This scenario supports the lifecycle
management of the Certificate including renewal and revocation. 

Existing certificates, in PFX or PEM format, can also be imported with the 
private key.

Certificates in Azure Key Vault can then be associated to your Web App 
in the same subscription. Once the certificate is in place, you can 
associate your custom domain with the certificate.

When a Key Vault certificate is created, an addressable key and secret 
with the same name is also created. The key allows key operations and 
the secret allows retrieval of the certificate value as a secret. If the
policy used to create the certificate indicates that it is exportable, then 
the certificate value will include the private key when retrieved as a 
secret. If the certificate is flagged as non-exportable, then the secret
will not include the private key when retrieved.

A certificate policy indicates how to create and manage the certificate
life cycle. If the certificate is imported, then the policy is read
from the certificate. If the certificate is created in Key Vault, then 
the policy must be supplied. 

A certificate policy contains the following items: 

1. X509 properties - including subject names and aliases and other properties used to create an x509 certificate request
2. Key properties - key type, key length, exportable and reuse key fields
3. Secret properties - including content type of addressable secret
4. Lifetime Actions - includes both a trigger and action for each lifetime action
5. Issuer - information about the certificate issuer
6. Policy attributes - attributes associated with the policy

Before creating a certificate issuer in a Key Vault, the organisation 
administrator must onboard the company with a CA provider and generate
credentials used with the provider for requesting new and renewal 
certificates.

## Protecting Key Vault
If a key vault is deleted, any data that is protected using keys from the 
Key Vault become inaccessible. Use soft delete and purge protection settings 
to protect Key Vaults from inadvertant or malicious deletion. 

Each key, secret and certificate stored in Key Vault can be backed-up
manually using the 'Download Backup' link on each item. The backup acts as
an offline copy of the secret.

Backups should be made on each update/delete/create event in the Key Vault

## Key Rotation

A rotation strategy should be implemented for the values stored in Key Vault
secrets. The strategy can be implemented: 

- as part of a manual process
- via REST API calls
    - an example of this would be using Event Grid in response to an expiry event. The Event Grid can call a Function App generates a new secret and updates the application using the secret
- using Azure Automation

## Key Vault CLI Commands

    az keyvault create --name $KV_NAME --group $RG_NAME --location $LOCATION
    az keyvault secret set --vault-name $KV_NAME --name "Example Password" --value $SECRET_VALUE
    az keyvault secret show --vault-name $KV_NAME --name "Example Password"
    az keyvault set-policy --secret-permissions get list --name $KV_NAME --object-id $SP_ID
