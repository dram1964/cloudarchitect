# App Service Plans
An app always runs in an App Service Plan. An App Service Plan defines a set of compute resources 
for the web app to run. App Service Plans can run multiple web apps or Azure Functions
App Service Plans define:

- Region
- Number of VM instances
- Size of VM instances
- Pricing Tier

Pricing Tier Categories

- Shared Compute (Free and Shared): resource pools are shared with other customers. Intended for development and testing only. Apps are assigned CPU quotas (in minutes) and cannot scale out
- Dedicated Compute (Basic, Standard, Premium, PremiumV2, PremiumV3): run on dedicated VMs. Only apps in the same Service Plan share resources. Higher Tiers provide more VMs for scale-out
- Isolated: adds dedicated VNet to dedicated VMs. Provides maximum scale-out capabilities
- Consumption: only available to Function Apps

The App Service Plan is the scale unit for the apps. Apps in the Service Plan run on all VM instances configured by the Plan or the autoscale (scale-out) settings. Scaling up or down is achieved by changing the pricing tier of the plan.

# App Deployment
App Service supports both automated (continuous integration) or manual deployment. Automated 
deployments can run from Azure DevOps, GitHub or BitBucket. Azure DevOps allows you to:

- build your code in the cloud
- run the tests
- generate a release
- push your code to an Azure Web App

Github deployments will automatically deploy when you push to the production branch
Manual Deployment Methods include:

- Git: configure a Git URL for the web app and deploy by pushing to the Git URL
- CLI: use the `az webapp up` command to create and package your app and deploy it
- Zip: send a zip of your application files using `curl` or similar command
- FTPS: code can be loaded to the App Service using FTP or FTPS

You can deploy a static html site from your current working directory using:
```
az webapp up -g $RESOURCE_GROUP -n $APP_NAME --html
```
This command will:

- Create the resource group if one is not specified
- Create a default App Service Plan
- Create an app with the specified name
- Zip deploy the files from the current working directory

# Authentication and Authorisation
App Service provides built-in authentication and authorisation otherwise you can use the
features provides by your web-application framework. App Service uses federated identity supporting
AAD, Facebook, Twitter, Google, or any OpenID provider. The federated providers are configured by 
setting the 'sign-in endpoint' for your app. Multiple providers can be used.
The authentication flow can be configured as a server-directed flow where the user is redirected
to the providers sign-in page or using the provider SDK, where the application handles the sign-in and
returns the authentication token to the app.
App Service can be configured to 'allow unauthenticated requests'. Unauthenticated requests can 
then be handled by your application code. The App Service can also be configured to 'require authentication': in which case you can redirect all unauthenticated requests to the appropriate handler.

# Networking
By default, apps hosted in App Service are accessible via the internet and can communicate with 
internet endpoints. The Isolated SKU hosts apps in an dedicated Azure VNet, but all the other
tiers are hosted on a multi-tenant network. For this reason, you cannot connect the multi-tenant 
app services directly to your network.
App Service apps are distributed applications: incoming http requests are handle by front-end roles; 
application workload is handled by worker roles. Different features are available depending on whether 
traffic is incoming or outgoing for your app:

- Inbound Features
    - App-assigned address
    - Access restrictions
    - Service endpoints
    - Private endpoints
  
- Outbound Features
    - Hybrid Connections
    - Gateway-required virtual network connection
    - Virtual network integration
  
The Free, Shared, Basic, Standard and Premium plans all use the same worker VM Type. Premium V2 and 
Premium V3 each use a different worker type. When the worker type is changed during a scale-up or down, 
the outbound IP address is changed. The current outbound IP for your app can be listed in the 
Properties for the app in the portal or by using the following command:
```
az webapp show -g $RG -n $NAME --query outboundIpAddress
```
To find all possible IP addresses for your app, use:
```
az webapp show -g $RG -n $NAME --query possibleOutboundIpAddresses
```

# Configuring Application Settings
In App Service, application settings are passed as environment settings to the application code. 
For Linux apps and Custom Containers, App Service uses the '--env' flag to pass these settings to the
app. Application settings are configured in the Portal via 'Configuration > Application Settings'. 
This way you can have one environment when running the app locally and another environment for 
your app when running in Azure
In App Service, application settings are passed as environment settings to the application code. 
For Linux apps and Custom Containers, App Service uses the '--env' flag to pass these settings to the
app. Application settings are configured in the Portal via 'Configuration > Application Settings'. 
This way you can have one environment when running the app locally and another environment for 
your app when running in Azure. The settings in the app configuration are encrypted.
Click 'New Application Setting' to add a new app setting. For nested JSON keys use an underscore
instead of a colon for the name. By clicking 'Advanced' you can add settings in bulk using a 
JSON array.
```
[
  {
    "name": "",
    "value": "",
    "slotSetting": false
  },
  {
    "name": "",
    "value": "",
    "slotSetting": false
  },
  ...
]
```
For ASP.NET developers, connection strings can also be cofigured in the apps Configuration page. 
For other languages, you may need to set the connection string within the codebase to ensure
correct formatting. However, certain database types are only backed-up with the app if the 
connection is configured in the configuration settings of the app. Connection strings 
can also be bulk-inserted using JSON:
```
[
  {
    "name": "",
    "value": "",
    "type": "SQLServer
    "slotSetting": false
  },
  {
    "name": "",
    "value": "",
    "type": "PostgreSQL"
    "slotSetting": false
  },
  ...
]
```

# Configuring General Settings
General settings for the app can be configured via 'Configuration > General Settings'. These 
include:

- Stack settings: language, major and minor versions. For container apps you can also set a startup command
- Platform settings:
    - 32-bit or 64-bit
    - Websocket Protocol
    - Always On: keep the app loaded even when there's no requests. Required by WebJobs or WebJobs triggered by a cron expression. If disabled, app is unloaded after 20 minutes of inactivity
    - IIS pipeline mode. Set to 'Classic' if you require a legacy version of IIS
    - HTTP version
    - ARR affinity: set to 'On' to ensure that the client is routed to the same worker for the duration of the session. Set to 'Off' for stateless applications
  
- Debugging: used to enable debugging for 48 hours for an ASP or Node app
- Incoming Client Certificates: require client certificates for mutual authentication

# Configuring Path Mappings
Use 'Configuration > Path Mappings' to configure handler mappings, and virtual application and 
directory mappings. In Windows apps, handler mappings can be used to add custom scripts to handle
specific file extensions. Default root-path settings can be changed in Path Mappings. For Linux apps, 
you can add custom storage

# Diagnostic Logging
Application logging tracks logs generated by the application code. Deployment logging tracks 
information concerning app deployment. Application and Deployment logging is available for both 
Windows and Linux apps. Additional logs are available for Windows apps only:
- Web server logging: raw HTTP request data
- Detailed error logging: copies of the .html error pages that would have been sent to the client
- Failed request tracing: detailed tracing information
Application logging for Windows allows you to log to File (for temporary logging over a 12 hour period) or Blob and lets you choose the log level (Disabled, Error, Warning, Information, Verbose). For Linux
apps you set the logging to file system and choose the retention period
Stream logs can be used by writing to the '/LogFiles' directory using '.htm', '.txt', '.log' 
file extensions
Configured log files can be accessed from the Storage Account. For logs stored in the App Service
file system use: https://<app-name>.scm.azurewebsites.net/api/logs/docker/zip (for Linux) or 
https://<app-name>.scm.azurewebsites.net/api/dump (for Windows).

# Configuring Certificates
Azure App Service comes with tools for managing private and public certificates. A certificate
uploaded to an app is stored in a deployment unit that is bound to the apps resource-group and 
region combination (referred to as the 'webspace'). The certificate is then accessible to other
apps in the same webspace.
There are various options for adding certificates in App Service:

- Create a free App Service managed certificate: a private certificate to secure your custom domain in App Service
- Purchase an App Service certificate: a private certificate managed by Azure. Provides automated certificate management and renewal
- Import a certificate from Key Vault
- Upload a private certificate
- Upload a public certificate: can't be used with custom domains

Use the TLS/SSL settings for your app to enforce HTTPS only

# Feature Flags
Azure App Service supports the use of feature flags to enable or disable features in an 
application. 

# Autoscaling
Autoscaling performs scale-in or scale-out according and can be triggered by:

- A schedule
- CPU usage
- Memory occupancy
- Number of incoming requests
- A combination of factors

Autoscaling monitors the resource metrics of a web app as it runs and responds to changes by
adding or removing web servers and load balancing between them. Autoscaling does not affect the 
resources available to the web servers: only the number of web servers. Autoscaling responds to 
rules that define thresholds to trigger an autoscale event.
Care should be taken when defining autoscale rules: scaling-up in response to a DDoS attack
would be pointless and expensive. Instead, DDoS protection should be enabled to filter such 
attacks.
Autoscaling improves availability and fault-tolerance by ensuring enough servers are available to 
service requests and that crashed servers are replaced. Autoscaling might not help cope with 
resource-instensive requests: scaling-up might be a better response in this case. Autoscaling 
does take time to spin up new instances, and so availability and response time may be affected 
during periods of surge in demand
Autoscale rules can combine both schedules and metric-based rules. Metrics include:

- CPU Percentage
- Memory Percentage
- Disk Queue Length
- Http Queue Length
- Data In
- Data OUt

Metrics can also be used from other Azure services, e.g Service Bus queue length. Metrics are 
aggregated over a period of time known as a time grain (usually 1 minute). The autoscale event is 
only triggered if the rule crosses the threshold over a time duration (usually 10 minutes). Several
aggregations are available. If the aggregation is MAX, then a single time grain exceeding the 
threshold will be enough to trigger the autoscale. If the aggregation is AVG, then multiple
time grains will need to exceed the threshold to trigger the autoscale for the duration. 
Autoscaling can either trigger an incremental increase or decrease or the trigger can set the 
number of instances explicitly. Autoscale rules should be paired: setting the value for when 
to scale-out and when to scale-in in response to a specific metric.
Scaling rules can combine multiple conditions. Scale-out will occur if any scale-out rule is
met: scale-in will occur only if all scale-in rules are met. If you want to scale-in when only
one scale-in rule is met, then you should define the rules separately
Once auto-scaling is enabled for an app, scale conditions can be defined. The default 
condition can be use to define a default scale for the app (e.g. one instance). The default
condition will be applied when no other condition is met, or when there are no metrics
available due to a system crash. Additional 
conditions can be used to define the scale according to a schedule or in response to metrics. 
The conditions allow you to set minimum, maximum and default scales for the app.
Autoscaling activity can be monitored in the 'Run History' tab of the autoscale blade
Thresholds are calculated across instances. Flapping can occur if there is not an adequate
margin between scale-out and scale-in conditions. For instance consider a rule to scale out if 
queue length is <80 and scale-in if queue length is >80. When three instances are running 
with a queue length of 75 if scale-in were to occur the queue length would be >80 for the 
two instances (75 x 3 / 2), so the app would immeadiately have to scale-out again. To avoid 
flapping, autoscale doesn't scale-in when it detects that the scale-out threshold will be 
met if it scales-in. By setting an adequate margin between scale-out and scale-in conditions, 
flapping can be avoided.
Autoscaling logs activity and these can be used to trigger alerts or used by webhook or 
email notifications. Autoscaling logs the following:

- issue a scale operation
- successfully complete a scale action
- failed scale action
- metrics not available
- metrics available again


# Deployment Slots
Standard, Premium and Isolated Service Plans provide Deployment Slots for your application, allowing
the same app to run in different environments. Each slot has it's own hostname. Slots can be swapped, 
allowing you to test a new version of the app before swapping it into production. Slot swapping 
provides the following benefits:

- validate changes in a development slot before swapping it into the production slot
- swapping a development slot into production, means that all instances of the slot are warmed-up before being swapped, thus speeding up the deployment into production
- The previous production slot is preserved in the slot is was swapped with, enabling quick failback if errors are found in the new production slot

Standard Plans provide 5 deployment slots whereas Premium and Isolatd plans provide up to 20 slots.
New deployment slots have no content, even if you clone them from an existing environment slot. 
You can deploy to the slot from a repository branch or a different repository
When slots are swapped, App Service ensures that the target slot does not experience downtime as 
follows:

- Apply the following settings from the target to the source slot:
    - slot-specific application settings and connection strings
    - continuous deployment settings
    - app service authentication settings
- Restart source slot instances. If using 'swap with preview' the swap is paused to allow you to validate this phase of the swap. If an instance fails to restart, the changes are reverted and the swap is discontinued
- initialise local caches on each instance
- trigger application warm-up on each instance in the source slot
- swap the two slots by switching the routing rules for the two slots
- re-apply the process to the source slot, so that it has a copy of the previous app

Since all the activities are applied to the source slot, the target slot experiences minimal 
downtime.

- Settings that are NOT swapped (slot specific):
    - Publishing endpoints
    - Custom domain names
    - non-public certificates and TLS/SSL settings
    - Scale settings
    - WebJobs schedulers
    - IP restrictions
    - Always On
    - Diagnostic log settings
    - Cross-Origin Resource Sharing (CORS)
  
- Settings that are swapped:
    - General settings
    - Handler settings
    - Public certificates
    - WebJobs content
  
- Settings that are swapped but can be configured to NOT swap:
    - App settings
    - Connection strings
    To configure the settings to not swap, select 'deployment slot setting' next to the setting in the configuration page for that slot
  
- Setting that are currently swapped but planned to be NOT swapped:
    - Hybrid connections
    - Virtual network integration
    - Service Endpoints
    - Azure Content Delivery Network
  
Auto swap can be configured for a slot, so that every time code is pushed to a slot, App Service 
will automatically swap the slot into the target slot. Auto swap is not currently available for 
web apps on Linux
By default, all traffic to the app's production URL are routed to the production slot. You can 
route traffic to another slot by setting the percentage of traffic to be routed to the new slot. 
Information about the slot a request is routed to, can be inspected in the HTTP header by checking
the value of 'x-ms-routing-name'. You can also manually route traffic to a different slot by 
including an 'x-ms-routing-name' link on your website page. New slots are automatically assigned 
a routing percentage of zero. If you set this value to zero explicitly, then developers can 
access the slot using 'x-ms-routing-name' for testing purposes

