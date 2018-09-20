# Scaling applications

Now that all of our services are up and running, let's double-check our pods:

```
kubectl get deploy,po
```

**Output:**
```
NAME                                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/captainkube   1         1         1            1           49m
deployment.extensions/nodebrady     1         1         1            1           22m
deployment.extensions/parrot        1         1         1            1           57m
deployment.extensions/phippy        1         1         1            1           4m

NAME                              READY     STATUS    RESTARTS   AGE
pod/captainkube-f55b77446-hf2pd   1/1       Running   0          49m
pod/nodebrady-6d8c9f7bcf-p2g6w    1/1       Running   0          22m
pod/parrot-77b5f95cc6-jjwp8       1/1       Running   0          57m
pod/phippy-7cc47cc88-ftbtz        1/1       Running   0          4m
```

Remember that the deployment tracks the desired state for our application. If we want more copies of brady, we tell the deployment to increase its replica count. The deployment controller will notice that the desired cluster state doesn't match the current state and create or delete pods as needed.

```
kubectl scale deployment/nodebrady --replicas=5
```

**Output:**
```
deployment.extensions "nodebrady" scaled
```

Check the state of the cluster now. We should see 2 instances of `nodebrady`, reported both by the deployment controller and then the number of pods.
```
kubectl get deploy,po -o wide
```

**Output:**
```
NAME                                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS    IMAGES                       SELECTOR
deployment.extensions/captainkube   1         1         1            1           51m       captainkube   jhansen/captainkube:v0.0.1   app=captainkube
deployment.extensions/nodebrady     2         2         2            2           24m       nodebrady     jhansen/brady:v0.0.1         app=nodebrady
deployment.extensions/parrot        1         1         1            1           59m       parrot        jhansen/parrot:v0.0.1        app=parrot
deployment.extensions/phippy        1         1         1            1           6m        phippy        jhansen/phippy:v0.0.1        app=phippy

NAME                              READY     STATUS    RESTARTS   AGE       IP            NODE
pod/captainkube-f55b77446-hf2pd   1/1       Running   0          51m       10.244.0.23   aks-nodepool1-25718494-0
pod/nodebrady-6d8c9f7bcf-8pl7n    1/1       Running   0          10s       10.244.1.21   aks-nodepool1-25718494-1
pod/nodebrady-6d8c9f7bcf-b6hn2    1/1       Running   0          10s       10.244.0.29   aks-nodepool1-25718494-0
pod/nodebrady-6d8c9f7bcf-hhtv5    1/1       Running   0          6m        10.244.1.20   aks-nodepool1-25718494-1
pod/nodebrady-6d8c9f7bcf-p2g6w    1/1       Running   0          6m        10.244.0.24   aks-nodepool1-25718494-0
pod/nodebrady-6d8c9f7bcf-zdmjp    1/1       Running   0          10s       10.244.0.30   aks-nodepool1-25718494-0
pod/parrot-77b5f95cc6-jjwp8       1/1       Running   0          59m       10.244.0.22   aks-nodepool1-25718494-0
pod/phippy-7cc47cc88-ftbtz        1/1       Running   0          6m        10.244.0.25   aks-nodepool1-25718494-0
```

## Delete Pods, watch them come back

What happens if your application crashes, or a node becomes unavailable? Kubernetes control loops are always watching the current state of the cluster and driving toward desired state. Let's delete a node brady pod and see what happens:

```
kubectl delete pod/nodebrady-6d8c9f7bcf-p2g6w
```

**Output:**
```
pod "nodebrady-6d8c9f7bcf-p2g6w" deleted
```

Kubernetes control loops are fast. You might miss it, list the pods again and you'll see that a replacement instance of `nodebrady` is already underway:

```
kubectl get deploy,po -l app=nodebrady
```

**Output:**
```
NAME                              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nodebrady   5         5         5            5           17h

NAME                             READY     STATUS    RESTARTS   AGE
pod/nodebrady-6d8c9f7bcf-4jpm2   1/1       Running   0          1m
pod/nodebrady-6d8c9f7bcf-8pl7n   1/1       Running   0          4m
pod/nodebrady-6d8c9f7bcf-b6hn2   1/1       Running   0          4m
pod/nodebrady-6d8c9f7bcf-hhtv5   1/1       Running   0          17h
pod/nodebrady-6d8c9f7bcf-zdmjp   1/1       Running   0          4m
```

Pretty neat!

Let's clean up Phippy and friends, tell `kubectl` to delete all the resources that appear in the `manifests/` directory:

```
$ kubectl delete -f manifests/
```

**Output:**
```
service "nodebrady" deleted
deployment.apps "nodebrady" deleted
serviceaccount "captainkube" deleted
clusterrole.rbac.authorization.k8s.io "captainkube-role" deleted
clusterrolebinding.rbac.authorization.k8s.io "captainkube-binding" deleted
deployment.apps "captainkube" deleted
service "parrot" deleted
deployment.apps "parrot" deleted
service "phippy" deleted
deployment.extensions "phippy" deleted
```

Next, let's look at how we can make it easy to manage applications with [Helm, the Kubernetes package manager](./04-get-started-with-helm.md).