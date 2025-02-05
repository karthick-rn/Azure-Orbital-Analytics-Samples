# Overview of Infrastructure deployment and configuration

We are using bicep templates to deploy the infrastructure resources.
In a default setup we will be using following azure services:
- Azure Batch Account
- Azure Synapse Analytics
- Azure Keyvault
- Azure Storage Account
- Azure Networking
- Azure Log Analytics

# Prerequisites

The deployment script uses following tools, please follow the links provided to install the suggested tools on your computer using which you would execute the script.

- [git](https://github.com/git-guides/install-git) 
- [az cli](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [jq](https://stedolan.github.io/jq/download/)
- The scripts are executed on bash shell, so if using a computer with windows based operating system, install a [WSL](https://docs.microsoft.com/windows/wsl/about) environment to execute the script.
- Install bicep on az cli using command  `az bicep install` and ensure the bicep version is >=0.8.9.
Alternatively, one can use Azure Cloud Bash to deploy this sample solution in their Azure subscription.

## RBAC requirement

The user performing the deployment of the bicep template and the associated scripts should have `Owner` role assigned at the subscription to which the resources are being deployed. This is needed in order to grant IAM roles to managed identities in bicep templates.

> [!NOTE]
If you have started the deployment with a different role Ex: `Contributor`, and the deployment failed due to insufficient access. Please change the role to `Owner` and refresh the credentials by re-logging before attempting to deploy again.

## Infrastructure Deployment

Get the repository to find the scripts. Clone the repository using following command.

```bash
git clone git@github.com:Azure/Azure-Orbital-Analytics-Samples.git
```

Before executing the script, please login to azure using `az` cli and set the subscription in which you want to provision the resources.

```bash
az login
az account set -s <subscription_id>
```

The following command will install, configure and generate the custom vision model package.

```bash
./deploy/scripts/setup.sh <environmentCode> <location>
```
- `environmentCode` used as a prefix for Azure resource names. It allows only alpha numeric(no special characters) and must be between 3 and 8 characters.
- `location`is a valid Azure region.

For eg.
```bash
./deploy/scripts/setup.sh aoi eastus
```

[setup.sh](./scripts/setup.sh) executes tasks in 3 steps
- installs the infrastructure using [install.sh](./scripts/install.sh) script.
- configures the infrastructure for setting up the dependecies using [configure.sh](./scripts/configure.sh) script.
- packages the pipeline code to a zip file using [package.sh](./scripts/package.sh) script.

After the script has run successfully, please check the batch-account pool created is created successfully.

- You can find the batch account named as `batchact<10-character-random-string>` under the resource group `<environment-code>-orc-rg`.
- Go to the Batch Account and switch to the pools blade. Look for one or more pools created by the bicep template. Make sure the resizing of the pool is completed without any errors. 
- Resizing may take a few minutes. Pools that are resizing are indicated by `0 -> 1` numbers under dedicated nodes column. Pools that have completed resizing should show the number of dedicated nodes. 
- Wait for all pools to complete resizing before moving to the next steps.

Error while resizing the pools are indicated by red exclamation icon next to the pool. Most common issues causing failure are related to the VM Quota limitations.

## Executing Synapse Pipeline

We have published the Pipeline on Synapse gallery, follow the following steps to find the Synapse pipeline and execute.

- Find your Synapse workspace named as `synws<10-character-random-string>` in resource group `<environment-code>-pipeline-rg`.
- Click on `Workspace web URL` to reach the synapse workspace portal homepage.
- Click on `Knowledge center`.
- Click on `Browse gallery`.
- Click on `Pipelines` in the top menu.
- In the search box type `space` to find the pipeline named `Spaceborne Data Analysis Master Pipeline`.
- Click on pipeline name to select it, and press `Continue` button at the end of the page.

Follow the steps in [instructions.md](./gallery/instructions.md) to configure and execute the pipeline.

## Cleanup Script

We have a cleanup script to cleanup the resource groups and thus the resources provisioned using the `environmentCode`.
As discussed above the `environmentCode` is used as prefix to generate resource group names, so the cleanup-script deletes the resource groups with generated names.

Execute the cleanup script as follows:

```bash
./deploy/scripts/cleanup.sh <environmentCode>
```

For eg.
```bash
./deploy/scripts/cleanup.sh aoi
```

If one wants not to delete any specific resource group and thus resource they can use NO_DELETE_*_RESOURCE_GROUP environment variable, by setting it to true

```bash
NO_DELETE_DATA_RESOURCE_GROUP=true
NO_DELETE_MONITORING_RESOURCE_GROUP=true
NO_DELETE_NETWORKING_RESOURCE_GROUP=true
NO_DELETE_ORCHESTRATION_RESOURCE_GROUP=true
NO_DELETE_PIPELINE_RESOURCE_GROUP=true
./deploy/scripts/cleanup.sh <environmentCode>
```

## AI Model

Follow the [document](./docs/using-ai-model-in-pipeline.md) to understand and use [Custom Vision Model](/src/aimodels) or bring different ai-model for processing with the pipeline.
