Notes on DevOps drawn mostly from <cite>Implementing Azure Devops Solutions, Henry Been, Maik van der Gaag, Packt Publishing, 2020</cite><p>

# Devops Defined
Traditionally development and operations teams were isolated from each other
and worked towards different goals: developers add value to by delivering new
features to products, requiring changes to infrastructure; operations teams
add value by maintaining the stability of deployed infrastructure, requiring 
strict controls on any changes being implemented to the deployed infrastructure
DevOps removes this conflict of interest by creating teams that are each 
responsible for both the development of new features and the delivery of these 
features. DevOps teams consist of both developers and operations staff working 
together on a common goal or product, to deliver new features to production 
in a fast and reliable manner. 
This closely aligns with Agile metholdology where the focus is on reducing the 
time to delivery of new business value. The introduction of DevOps is often 
associated with the adoption of Agile.
When adopting DevOps teams will attempt to standarise on a common set of tools
and standards for source control, development and architecture deployment. As the
pipelines for infrastructure deployment mature, self-service capabilities become
available where developers become able to create and destroy environments 
on-demand through a published API. 

## Key Metrics for Devops
The key metrics for measuring the success of your DevOps teams are:
<dl>
  <dt>Lead Time</dt>
  <dd>The time between requesting a feature and starting the implementation of the feature</dd>
  <dt>Cycle Time</dt>
  <dd>The time between starting work on a feature and users being able to use the feature</dd>
  <dt>Time to Market</dt>
  <dd>The sum of Lead Time and Cycle Time</dd>
  <dt>Amount of Work in Progress</dt>
  <dd>DevOps should focus on the flow of value to the user. This implies that instead of switching from one incomplete task to another, each team member should be working on no more than one task at a time</dd>
  <dt>Mean Time To Recovery</dt>
  <dd>DevOps shifts the focus from Mean Time Between Failures to Mean Time To Recovery. Mean Time Between Failures encourages resistance to change. Less change means less failures. By focussing on Recovery Time, teams are not dis-incentivised to implement change</dd>
  <dt>Change Rate</dt>
  <dd>Increasing the number of implemented changes implies that you are delivering value more rapidly</dd>
  <dt>Change Failure Rate</dt>
  <dd>Focussing on the failure rate for changes encourages the implementation of smaller changes: increasing the Change Rate and reducing the likelihood of failure</dd>
</dl>

# Seven Pracitices of Devops
The seven practices of DevOps are:
<dl>
  <dt>Configuration Management or Configuration As Code</dt>
  <dd>Configuration for the applcation and its components is stored as JSON/YAML in a version control repository and serves as input for deployment tools. Configuration describes the desired state of the application and should be applied regularly to ensure that there is no configuration drift. Re-applying configuration will revoke manual changes made outside of configuration management.</dd>
  <dt>Release Managment</dt>
  <dd>Release management is about controlling which versions of the software is deployed to which environments. Versions of the software and its configuration are stored as immutable artifacts in a repository and release management tools are used to manage the deployment of versions.</dd>
  <dt>Continuous Integration</dt>
  <dd>Continuouse integration requires each developer to integrate their work with that of every other developer at least once a day. A continuous integration build ensures that it compiles and passes unit tests. Frequent integrations ensures that changes are minor and less likely to result in problems. When problems arise they should be easier to fix.</dd>
  <dt>Continuous Deployment</dt>
  <dd>Continuous deployment is the automated deployment of each new version to prodution as long as it passes quality tests. The automated deployment pipeline involves deploying each version through a hierarchy of stages where it must pass the tests for that stage before proceeding to deployment to the next stage. If a release fails at any stage it is completely revoked. This ensures that any fixes to a broken release pass through all stages of testing before hitting production.</dd>
  <dt>Infrastructure As Code</dt>
  <dd>Infrastructure to run the application is also stored in a source control repository. Infrastructure code describes the desired state and should be idempotent: no matter how many times you deploy it, the result should be the same. To avoid configuration drift, the desired state for infrastructure should be applied regularly to each environment using continuous deployment pipelines and is often done before new software versions are released to the environments.</dd>
  <dt>Test Automation</dt>
  <dd>Continuous deployment involves frequent releases and therefore manual testing becomes impractical. Each deployment stage will require different types of tests that are triggered automatically when the release hits that stage. Manual testing should be reduced to the bare minimum, and should occur only at the final stages before deployment to production.</dd>
  <dt>Application Performance Monitoring</dt>
  <dd>Involves monitoring performance and usage of the software in production, to gather intelligence about the value of the features in each release.</dd>
</dl>


# Seven Habits of Devops
The seven habits of DevOps are:
<dl>
  <dt>Team Autonomy and Enterprise Alignment</dt>
  <dd>Each team should be working autonomously towards their current goal, however controls will need to be in place to ensure teams stay aligned to organisational goals.</dd>
  <dt>Rigourous Management of Technical Debt</dt>
  <dd>Technical debt is the cost associated with delay in addressing issues that affect the development pipelines. Fixing bugs and architectural issues should be prioritised and kept to a minimum.</dd>
  <dt>Focus on Flow of Customer Value</dt>
  <dd>Teams should be focussed on the delivery of features that are used by the customer.</dd>
  <dt>Hypothesis-driven Development</dt>
  <dd>Minimal implementations are released to production to determine whether they deliver actual value to the product based on usage patterns. If the hypothesis proves popular, the feature should be completed.</dd>
  <dt>Evidence Gathered In Production</dt>
  <dd>Performance measurements should be gathered from the production environment and compared to previous measurements.</dd>
  <dt>Live-Site Culture</dt>
  <dd>The production environment is prioritised above all others: issues or changes affecting production or release to production are tackled before all other tasks.</dd>
  <dt>Manage Infrastructure as a Flexible Resource</dt>
  <dd>Infrastructure is deployed when needed and discarded when no longer needed.</dd>
</dl>

# Source Control
Source control can either be centralised or decentralised. In a centralised
system, all changes and branches are stored in a single location. Developers only
receive the latest version of the code when they checkout the repository. To view
history or other branches, they have to access the central server. Centralised
source control reduces the amount of storage that is required locally and can 
allow for better access control on branches, directories and files.
In a decentralised source control repository, each developer takes a full copy 
of the repository by cloning a local copy. This means each developer can view the
full history of changes on their local copy without being connected to the central
server.
Azure DevOps supports Git as a decentralised source control system. Git is 
optimised for working with text files. If your project also needs to track 
images or binary files then you will need to use Git LFS (Large File Storage).
Further information on <a href="https://docs.microsoft.com/en-gb/learn/paths/intro-to-vc-git/" target="#">Git</a> and <a href="https://docs.microsoft.com/en-gb/azure/devops/repos/git/manage-large-files?view=azure-devops" target="#">Git LFS</a>. It 
is worth noting that Azure Repos do not support ssh in repositories with Git LFS 
tracked files

## Branching Strategies
There are three popular branching strategies:
<dl>
  <dt><a href="https://docs.github.com/en/get-started/quickstart/github-flow" target ="#">GitHub Flow</a></dt>
  <dd>In GitHub Flow there is one <b>main</b> branch that is always in a deployable state. New features are started in a new branch and are not merged back to the <b>main</b> branch until complete.</dd>
  <dt><a href="https://datasift.github.io/gitflow/IntroducingGitFlow.html" target ="#">GitFlow</a></dt>
  <dd>GitFlow involves creating a branch from <b>main</b> called <b>develop</b> whenever you start work on new features. From <b>develop</b>, new branches are created for each new feature being developed. When the new feature is complete it is merged back into <b>develop</b>. When you want to release a new version of the application, you create a <b>release</b> branch from the develop branch. Once testing and bug fixes are applied to the <b>release</b> branch, this branch is then merged into <b>main</b> and tagged with the version number. Changes from testing the release are also merged back to the <b>develop</b> branch for the next round of development. Urgent bug fixes are carried out on a new branch from <b>main</b>, and are merged back to both <b>main</b> and <b>develop</b> branches when ready.</dd>
  <dt><a href="https://devblogs.microsoft.com/devops/release-flow-how-we-do-branching-on-the-vsts-team/" target="#">Release Flow</a></dt>
  <dd>Involves working with short-lived topic branches just as with GitFlow. However, the <b>main</b> branch is never released to production. Instead when a new version is to be deployed, a new branch from the <b>main</b> branch is created named <b>release-{version}</b> and this is pushed to production. Urgent bug fixes can be merged from the <b>main</b> branch to the current <b>release</b> branch.</dd>
</dl>

## Pull Requests
Pull requests are a way to ensure that a merge only occurs on a branch when the
appropriate approval processes have been completed. Pull requests allow other team
members to review the changes in the branch and add comments before the merge
Once a pull request is approved there are three methods available for the 
merge commit:
- Merge Commit - retains full visibility of the changes on both the source and target branch. Commit history will reference both the source and target branches.
- Squash Commit - all the changes from the source branch are combined into one new commit on the target branch
- Rebase - changes from the source branch are temporarily stored elsewhere while changes from the master branch are applied to the source. Then the source branch has its own commits re-applied. Once the source is rebased, the commits can then be applied to the master branch. Rebasing retains all the change history on a single branch - the master branch

# Continuous Integration
Continuous integration involves combining your changes with those of other
developers on your project and testing whether the combined code works as 
expected. Frequent code merges means that the changes are smaller and less likely
to result in merge conflicts. Continuous integration adds value by testing the
resulting integrated work for the team.
Azure DevOps uses Azure Pipelines to create a continuous integration 
build, with YAML pipelines being the preferred method. Storing your build 
definitions in YAML, means that you can include this definition in the source 
control for your project and it is then available on all branches and will be
subject to the same policies applied to changing the code.
After saving a YAML file in the repository, you can create a build definition
from it.
The default YAML pipeline file in Azure DevOps is called 'azure-pipelines.yml'. 
The pipeline definition should begin with trigger actions that can be 
monitoring either pull requests (<b>pr</b>) or commits (<b>trigger</b>):
```
pr:
  branches:
    include: ["main", "release", "develop"]

trigger:
- main
- release
- develop
- feature/*
```

Pipelines are configured to run on an agent using the <b>pool</b> block:
```
pool:
  vmImage: ubuntu-18.04
```

Variables can be configured for later use in the pipeline:
```
variables:
  folder_context: $(System.DefaultWorkingDirectory)/Terraform/environments
  terraform_version: 1.1.7
  is_main: $[eq(variables['Build.SourceBranch'], 'refs/heads/main')]
  is_test: $[eq(variables['Build.SourceBranch'], 'refs/heads/release')]
  is_dev: $[eq(variables['Build.SourceBranch'], 'refs/heads/develop')]
  is_feature: $[startsWith(variables['Build.SourceBranch'], 'refs/heads/feature/')]
```

The rest of the pipeline will define the various stages to be executed. 
Typically this will include:
<ol>
- a build stage, where the new code is tested and
- deployment stages, where the code is deployed to various environments
</ol>
Each stage will define a series of jobs, with each job executing one or more
steps:
```
stages:
- stage: terraform_build
  jobs:
  - job: terraform_plan
    steps:
    - task: AzureKeyVault@1
      inputs:
        azureSubscription: $(subscription_id)
        KeyVaultName: $(keyvault_name)
        SecretsFilter: 'state-resource-group, state-storage-container, state-storage-account, state-file-name, state-sas-token, client-id, client-secret, subscription-id, tenant-id'
      displayName: 'Get key vault secrets as pipeline variables'
    - bash: |
        ...
        ---
      
```
Combining the above snippets will define a build pipeline that
will be triggered by commits to <b>main</b>, <b>develop</b>, 
<b>release</b> and any branch that begins with <b>feature/</b>
For IaC, the deployment stages should be executed conditionally: deployments
should only occur to each environment when the corresponding source-code branch
is updated:
```
- stage: development_env
  condition: eq(variables.is_dev, 'true')
  jobs:
  - deployment: development_environment
    displayName: development_environment
    pool:
      vmImage: Ubuntu-18.04
    environment: dev
    strategy:
      runOnce:
        deploy:
          steps:
          - checkout: self
          - task: AzureKeyVault@1
            inputs:
              azureSubscription: $(subscription_id)
              KeyVaultName: $(keyvault_name)
              SecretsFilter: 'state-resource-group, state-storage-container, state-storage-account, state-file-name, state-sas-token, client-id, client-secret, subscription-id, tenant-id'
            displayName: 'Get key vault secrets as pipeline variables'

          - bash: |
            ...
            ...

```
Adding the above code to our pipeline will mean that any pull
request to the <b>develop</b> branch will result in both the build 
stage (Continuous Integration) and the deployment to the development environment (Continuous Delivery) being executed
Stages can be added for both the testing environment and the production environment to conditionally execute after pull 
requests to their respective branches: <b>release</b> and <b>main</b>. 
<a href="https://about.gitlab.com/product/continuous-integration/" target="#">GitLab</a> and <a href="https://jenkins.io" target="#">Jenkins</a> offer tools to build continuous integration pipelines
See <a href="https://docs.microsoft.com/en-gb/learn/modules/create-a-build-pipeline/" target="#">Create a Build Pipeline with Azure Pipelines</a> and <a href="https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/?view=azure-pipelines" target="#">YAML Schema for Azure Pipelines</a>

# Continuous Deployment
Continuous Deployment is not the same as Continuous Delivery. Continuous 
Delivery is about building artefacts that are tested and ready to be deployed
to production. Continuous Deployment, takes this one step further and actually
includes deployment to production.
In Azure Devops delivery and deployment are realised using releases. Release
definitions are created from Pipelines and often begin with an artefact that 
triggers deployment. Then the enviroment (or stage) to deploy to is identified.
As with CI, YAML pipelines are the recommended means to define your CD 
pipelines
Continuous deployment could become challenging to users who have to deal 
with the challenges of new releases, potential bugs and possible rollbacks. To 
minimise the impact a suitable deployment strategy should be chosen. Potential
deployment strategies are:
<ol>
- Blue-Green Deployments. Two identical groups of servers are maintained: one to run the production instance and a second to act as the target for new deployments. A load balancer is used to direct users to the production environment. When a new release is deployed to the other group it is tested and when authorised as ready, the load balancer is used to switch users to the new group. The original environment can be used for rollback should problems occur or can be used as the target for the next release if no problems are found
- Immutable Servers - a variation of blue-green deployments, where new servers are configured for each deployment and the original production environment is discarded after a grace period (in case rollback is required).
- Canary deployments - initially only a selected group of users are exposed to the new environments, with further groups added if no problems are found
- Ring-based deployments - several production environments are maintained for different groups of users. New deployments are issued to one group at a time until the release is propogated to all groups in the ring
- Feature Flags - new versions are deployed but with the new features turned-off. Feature flags are used to slowly enable the new features. If the features passes quality acceptance, then the flag can be removed from the code base, and the feature becomes part of production environment
- Fail forward - instead of rolling back when issues are encountered in new releases, bug fixes can be applied to fix the issues.
</ol>
Further reading: <a href="https://docs.microsoft.com/en-gb/azure/devops/pipelines/?view=azure-devops" target="#">Azure Pipeline Docuemntation</a>

# Dependancy Management
To reduce build times, shared libraries can be pre-built for re-use across 
multiple projects. Azure Artifacts can be used to manage the library of 
re-usuable components (packages and artifacts) across projects. Further reading: 
<a href="https://docs.microsoft.com/en-gb/learn/modules/manage-build-dependencies/" target="#">Azure Artifacts</a>

# Infrastructure As Code
Infrastructure As Code (IaC) and Configuration As Code (CaC) is one of the seven
practices for DevOps. Using declarative templates for deploying your 
infrastructure means that configuration and infrastructure can be managed in the
same way that application code is managed. Several tools are available for 
implementing IaC and CaC:
<ol>
	<li><a href="https://docs.microsoft.com/en-gb/azure/azure-resource-manager/templates/" target="#">ARM templates</a> - Azure Resource Manager templates are JSON-like templates for declaring the desired state of your infrastructure, and can be deployed using Powershell, Azure CLI or Azure Pipelines
	<li><a href="https://aws.amazon.com/cloudformation/" target="#">Cloud Formation</a> - the IaC tool for AWS
	<li><a href="https://www.chef.io/" target="#">Chef</a> - a CaC tool for describing and enforcing server configuration
	<li><a href="https://puppet.com/" target="#">Puppet</a> - used for deployment and configuration of servers
	<li><a href="https://www.ansible.com/" target="#">Ansible</a> - IaC and CaC tool that can be used for both Linux and Windows servers
	<li><a href="https://www.terraform.io/" target="#">Terraform</a> - a multicloud infrastructure management tool
</ol>

# Databases
Managing schema changes in a CI/CD enviroment is challenging if downtime is 
to be minimised. ORMs are used to represent relational schemas as code: there
are two common approaches to managing schema changes with ORMs: migration-based 
and state-based.
Migration-based approaches keep an ordered set of changes that need to be applied
to the database to match the changes to ORM code. This becomes difficult when 
code is being changed on multiple branches. Where multiple migrations are needed,
they need to be applied in order to match the merge sequence
State-based approaches ignore the individual changes (migrations) that are
needed on a database and only focus on the changes required to fulfill the 
desired end-state of the database. However, such an approach may additionally 
require scripts to migrate the data as the schema is changed, involving data 
dumps and insert actions.
Both migrations and end-state approaches present challenges that need to be 
carefully managed and may result in taking a non-DevOps approach to schema changes
required by new releases. A Blue-Green approach might work, combined with ETL 
processes to migrate data from old to new servers
Using a schemaless database might be a suitable alternative to manage the 
synchronisation of data and code. As new properties are added/removed from the
objects in the schemaless database, code can be tweaked to handle objects in 
both the old and new format, until all old records are migrated to the new
format
<a href="https://docs.microsoft.com/nl-nl/ef/" target="#">Entity Framework</a> is the Microsoft ORM for .Net and supports migrations.

# Testing to Maintain Quality
Continuous Deployment requires continuous testing to ensure quality is 
maintained or improved with each release. Common Metrics for measuring quality
include:

- The percentage of integration builds that fail
- The percentage of code covered by automated tests
- The change failure rate
- The amount of unplanned work
- The number of defects that are being reported by users
- The number of known issues
- The amount of technical debt - the cost of sacrificing code quality for something else. As with financial debt, technical debt should be paid off as quickly as possible.

Automated Functional Tests fall into one of three categories:

- Unit Tests - used to test the smallest possible scope in an application, e.g. a class or method
- Integration Tests - testing that multiple units that are supposed to work together
- System Tests - run against a fully assembled and running application
Manual Functional Tests fall into one of two categories
- Scripted Test - formal test cases are prepared to test a new feature and a script is prepared of the actions to carry out the test. This script is then manually executed by the test team
- Exploratory Test - the test team explore the parts of the application they consider most at risk in the upcoming release. Records are kept of the risks that were checked and the issues that were found
Non-Functional Tests include
- Performance testing - how quickly a particular task can be performed in the fully assembled system. Can be automated and results can be compared to results from previous versions of the system
- Load testing - measuring response times as the number of requests are increased. The breaking point is the moment that response time begins to rise exponentially
- Usability testing - collecting feedback from users regarding their experience using the new release (user stories)

Different test types are executed at different stages in the CI/CD pipeline. 
Unit tests should be executed at each commit and before a merge or pull request. 
Integration tests should be done both before and after a merge or pull request. System tests and other testing will occur after deployment to a staging environment
