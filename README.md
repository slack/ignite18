# Introduction

Welcome to this hands-on lab for Azure Kubernetes Service (AKS). This lab will help you create an AKS cluster, deploy a microservices application, and explore basic Kubernetes concepts.

The latest content for this lab can be found at [https://aka.ms/containerfest/aks-lab](https://aka.ms/containerfest/aks-lab)

## Getting Started

For this lab, you will need access to an Azure subscription, the Azure web portal, and Azure Cloud Shell.

If you do not have access to an active Azure subscription you may obtain an Azure Pass. The Azure Pass provides free Azure credit, obtain a redemption code and visit [https://www.microsoftazurepass.com/](https://www.microsoftazurepass.com/) to activate your subscription.

Make sure to follow the [redemption instructions](https://www.microsoftazurepass.com/Home/HowTo).

## Launch Cloud Shell

[Azure Cloud Shell](https://azure.microsoft.com/en-us/features/cloud-shell/) is a browser-based CLI tool integrated directly into the Azure portal. Cloud shell provides all of the tools you need to manage your Azure resources in a pre-configured, on-demand virtual machine.

Navigate to the [Azure Cloud Shell https://shell.azure.com](https://shell.azure.com) and login if needed. You may also click the Cloud Shell Icon from the Azure Portal:

![Launch Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/media/overview/overview-bash-pic.png)

On first launch, Cloud Shell prompts to create a resource group, storage account, and Azure Files share on your behalf. This is a one-time step and will be automatically attached for all sessions. A single file share can be mapped and will be used by both Bash and PowerShell in Cloud Shell (Preview).

## Checkout the lab material

Next, grab a copy of the materials for the rest of this lab by cloning the lab repository from GitHub to your Cloud Shell instance:

```console
git clone https://github.com/slack/ignite18.git
cd ignite18
```

## Register Providers

Provider registration ensures that your subscription has the necessary Azure services activated on your account. Type the following commands into the Cloud Shell:

```console
watch az provider register -n Microsoft.ContainerService -o table
```

`watch` will run your command every second and display the output. When the RegistrationState is Registered, CTRL+C to continue with the lab.

**Output:**
```
Registering is still on-going. You can monitor using 'az provider show -n Microsoft.ContainerService'
```

Verify that the provider has been registered by showing your provider:

```console
az provider show -n Microsoft.ContainerService -o table
```

**Output:**
```
Namespace                   RegistrationState
--------------------------  -------------------
Microsoft.ContainerService  Registered
```

Great! You are ready to [create your first AKS cluster](lab/01-create-cluster.md).
