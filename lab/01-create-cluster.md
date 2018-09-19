# Create an AKS Cluster

Create a resource group to hold the cluster. This resource group should be provisioned in East US.

```console
az group create --name ignite18 --location eastus
```

**Output:**
```
{
  "id": "/subscriptions/57ac26cf-a9f0-4908-b300-9a4e9a0fb205/resourceGroups/foo",
  "location": "westus2",
  "managedBy": null,
  "name": "foo",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

Create a two node AKS cluster running Kubernetes 1.11:

```console
az aks create -g ignite18 --name ignite18 \
    --kubernetes-version 1.11.2 \
    --node-count 2 \
    --enable-addons http_application_routing --no-ssh-key
```

This cluster create will provision the control plane and attach VMs to your cluster. Initial cluster creation usually takes about 15 minutes.

If you close your Cloud Shell instance, you can watch for cluster creation to complete:

```console
az aks list -g ignite18 -o table
```

**Output:**
```
Name      Location    ResourceGroup    KubernetesVersion    ProvisioningState    Fqdn
--------  ----------  ---------------  -------------------  -------------------  ------------------------------------------------------
ignite18  eastus      ignite18         1.11.2               Succeeded            ignite18-ignite18-57ac26-22b0f7a8.hcp.eastus.azmk8s.io
```

## Fetch cluster credentials

Next, we need to fetch credentials to authenticate to the Kubernetes API. We will ask AKS for the cluster admin credentials so that we can configure Resource Based Access Control (RBAC) in a later step.

```console
az aks get-credentials -g ignite18 -n ignite18 --admin
```

**Output:**
```
Merged "ignite18-admin" as current context in /Users/jhansen/.kube/config
```

## List cluster nodes

Next, let's talk to the new cluster and list the cluster nodes:

```console
kubectl get nodes
```

Next [Let's deploy our services...](02-deploy-apps.md)