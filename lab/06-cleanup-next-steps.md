## Resources

You have reached the end of this lab. Thanks for following along.

We've just scratched the surface of Azure Kubernetes Service, here are some additional resources to help you on your journey:

* [AKS quickstart, including building containers](https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-prepare-app)
* [Monitoring AKS container health](https://docs.microsoft.com/en-us/azure/monitoring/monitoring-container-health)
* [Integrate your AKS cluster with an existing VNet](https://docs.microsoft.com/en-us/azure/aks/networking-overview)
* [Deploy applications to AKS with Azure DevOps Project](https://docs.microsoft.com/en-us/azure/devops-project/azure-devops-project-aks)


## Clean up your cluster

Deleting your AKS cluster is as easy as creating it.

```
az aks delete -g ignite18 -n ignite18
```

Thanks!