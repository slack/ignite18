## Resources

You have reached the end of this lab. Thanks for following along.

We've just scratched the surface of Azure Kubernetes Service, here are some additional resources to help you on your journey:

### Kubernetes on Azure:

* [AKS quickstart](https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-prepare-app)
* [Monitoring AKS container health](https://docs.microsoft.com/en-us/azure/monitoring/monitoring-container-health)
* [Integrate your AKS cluster with an existing VNet](https://docs.microsoft.com/en-us/azure/aks/networking-overview)
* [Deploy applications to AKS with Azure DevOps Project](https://docs.microsoft.com/en-us/azure/devops-project/azure-devops-project-aks)
* [CI/CD for containers](https://azure.microsoft.com/en-us/solutions/architecture/cicd-for-containers/)

### Kubernetes resources and tools:

* [The Illustrated Children's Guide to Kubernetes](https://www.youtube.com/watch?v=4ht22ReBjno).
* [Kubernetes Concepts](https://kubernetes.io/docs/concepts/)
* [Helm, the Kubernetes Package Manager](https://helm.sh)
* [Repository of community-maintained Helm charts](https://github.com/helm/charts)


## Clean up your cluster

Deleting your AKS cluster is as easy as creating it.

```
az aks delete -g ignite18 -n ignite18
```

Thanks!