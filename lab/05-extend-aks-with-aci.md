# Azure Kubernetes Service and Azure Container Instances

Azure Container Instances (ACI) provide a hosted environment for running containers in Azure. When using ACI, there is no need to manage the underlying compute infrastructure, Azure handles this management for you. When running containers in ACI, you are charged by the second for each running container.

When using the Virtual Kubelet provider for Azure Container Instances, both Linux and Windows containers can be scheduled on a container instance as if it is a standard Kubernetes node. This configuration allows you to take advantage of both the capabilities of Kubernetes and the management value and cost benefit of container instances.

Install the ACI to Kubernetes connector using the az CLI:
```
az aks install-connector -n ignite18 -g ignite18
```

**Output:**
```
Merged "ignite18" as current context in /tmp/tmpbgwvxtlp
Deploying the ACI connector for 'Linux' using Helm
NAME:   aci-connector-linux-eastus
LAST DEPLOYED: Thu Sep 20 17:59:05 2018NAMESPACE: defaultSTATUS: DEPLOYED

RESOURCES:
==> v1/Secret
NAME                                                TYPE    DATA  AGE
aci-connector-linux-eastus-virtual-kubelet-for-aks  Opaque  3     1s
==> v1/ServiceAccount
NAME                                                SECRETS  AGE
aci-connector-linux-eastus-virtual-kubelet-for-aks  1        1s

==> v1beta1/ClusterRoleBinding
NAME                                                AGE
aci-connector-linux-eastus-virtual-kubelet-for-aks  1s

==> v1beta1/Deployment
NAME                                                DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
aci-connector-linux-eastus-virtual-kubelet-for-aks  1        1        1           0          1s

==> v1/Pod(related)
NAME                                                             READY  STATUS             RESTARTS  AGE
aci-connector-linux-eastus-virtual-kubelet-for-aks-6c86769mzrt2  0/1    ContainerCreating  0         1s


NOTES:
The virtual kubelet is getting deployed on your cluster.

To verify that virtual kubelet has started, run:

  kubectl --namespace=default get pods -l "app=aci-connector-linux-eastus-virtual-kubelet-for-aks"

Note:
TLS key pair not provided for VK HTTP listener. A key pair was generated for you. This generated key pair is not suitable for production use.
```

The ACI connector automatically registers with the Kubernetes cluster as a virtual node. This virtual node can run Pods, just like the VM-based examples we've previously seen.

```
kubectl get node
```

**Output:**
```
NAME                                         STATUS    ROLES     AGE       VERSION
aks-nodepool1-25718494-0                     Ready     agent     1d        v1.11.2
aks-nodepool1-25718494-1                     Ready     agent     1d        v1.11.2
virtual-kubelet                              Ready     agent     1m        v1.11.2
```

To schedule work to our virtual node, we will use a few Kubernetes features, node selection and tolerations:

```
      nodeSelector:
        kubernetes.io/hostname: virtual-kubelet
      tolerations:
      - key: virtual-kubelet.io/provider
        operator: Equal
        value: azure
        effect: NoSchedule
```

These clauses in the `aci-helloworld` manifest inform the Kubernetes scheduler of some additional constraints. First, the scheduler should select a node whose hostname is `virtual-kubelet` and second, the workload can "tolerate" running on a node that is powered by a virtual kubelet.

Let's deploy an application that schedules to our virtual node:

```
kubectl apply -f manifests/aci-helloworld.yaml
```

Next, let's get a list of pods, but instead of selecting all pods we will use a label selector. This selector `app=aci-helloworld` will filter the results, only returning resources that have that label. Labels are key-value pairs and are part of the resource manifest.

```
kubectl get po -l app=aci-helloworld
```

**Output:**
```
NAME                              READY     STATUS    RESTARTS   AGE
aci-helloworld-57bf88c4c6-rffpb   0/1       Pending   0          30s
```

This pod is launching an Azure Container Instance, on-demand, without having to manage underlying compute. Once the Pod transitions to a `Running` state we can connect directly to that Pod via its public IP address:

```
kubectl get deploy,po -o wide -l app=aci-helloworld
```

**Output:**
```
NAME                                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS       IMAGES                     SELECTOR
deployment.extensions/aci-helloworld   1         1         1            1           31m       aci-helloworld   microsoft/aci-helloworld   app=aci-helloworld

NAME                                  READY     STATUS    RESTARTS   AGE       IP            NODE
pod/aci-helloworld-57bf88c4c6-rffpb   1/1       Running   0          31m       40.87.46.29   virtual-kubelet
```

Just like our other applications like brady and phippy, `aci-helloworld` is easily scaled with `kubectl`:

```
kubectl scale deploy/aci-helloworld --replicas=3
```

Azure Container Instances are automatically launched to accommodate the desired replicas.

**Output:**
```
kubectl get deploy,po -o wide -l app=aci-helloworld
NAME                                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS       IMAGES                     SELECTOR
deployment.extensions/aci-helloworld   3         3         3            3           30m       aci-helloworld   microsoft/aci-helloworld   app=aci-helloworld

NAME                                  READY     STATUS    RESTARTS   AGE       IP              NODE
pod/aci-helloworld-57bf88c4c6-dc226   1/1       Running   0          10m       23.100.24.76    virtual-kubelet
pod/aci-helloworld-57bf88c4c6-mxr7g   1/1       Running   0          10m       40.114.64.120   virtual-kubelet
pod/aci-helloworld-57bf88c4c6-rffpb   1/1       Running   0          30m       40.87.46.29     virtual-kubelet
```

## Virtual nodes in AKS [Optional]

AKS is opening a private preview with support for new virtual nodes, which includes private networking. If you are interested in joining, please fill out the sign-up form at [https:://aka.ms/aks-virtual-node-preview-sign-up](aka.ms/aks-virtual-node-preview-sign-up)

Next, [clean up your cluster and check out some additional resources](./06-cleanup-next-steps.md).
