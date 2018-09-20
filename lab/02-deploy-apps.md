# Deploy some applications

This lab walks you through deploying four applications all written in different languages. Once containerized, they all find a happy home on our Kubernetes cluster.

Should you wonder why we have owls, giraffes, parrots, and sloths, this cast of characters comes from [The Illustrated Children's Guide to Kubernetes](https://www.youtube.com/watch?v=4ht22ReBjno). 


## Deploy parrot

Parrot is a .NET Core application which serves a simple webpage which displays the contents of the Kubernetes cluster.

Applications in Kubernetes are composed of multiple types, including a Service, a Deployment, ConfigMaps, etc. If you examine the [parrot manifest](../manifests/parrot.yaml) you can see the Service and Deployment that comprise parrot.

Deploy parrot using `kubectl apply`:

```console
kubectl apply -f manifests/parrot.yaml
```

**Output:**
```console
service "parrot" created
deployment.apps "parrot" created
```

After the resources are submitted to Kubernetes, a set of control loops detect that the cluster state no longer matches the desired state. These control loops kick into action and create services and launch containers. You can see the results by asking kubectl to show Deployments, Services, and Pods:

```console
kubectl get svc,ep,deploy,po -l app=parrot
```

**Output:**
```
NAME             TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/parrot   LoadBalancer   10.0.213.246   <pending>     80:31801/TCP   45s

NAME               ENDPOINTS        AGE
endpoints/parrot   10.244.0.22:80   45s

NAME                           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/parrot   1         1         1            1           45s

NAME                          READY     STATUS    RESTARTS   AGE
pod/parrot-77b5f95cc6-jjwp8   1/1       Running   0          44s
```

Run this command a few times and you can watch as the cluster state converges. A few things to observe:

* By requesting a Service of type `LoadBalancer`, Kubernetes will reach out to Azure infrastructure and automatically provision a loadbalancer and public IP address. The loadbalancer is automatically configured to point to your application workload
* When the `EXTERNAL-IP` field for our `service/parrot` transitions from `<pending>` to a public IP, our service should be reachable

## Deploy captainkube

The second application in our example microservice is `captainkube`. This application is written in Go and is written to introspect the Kubernetes cluster and report cluster state to parrot for display.

Like parrot, submit the manifest for `captainkube`:

```console
kubectl apply -f manifests/captainkube.yaml
```

**Output:**
```
serviceaccount "captainkube" created
clusterrole.rbac.authorization.k8s.io "captainkube-role" created
clusterrolebinding.rbac.authorization.k8s.io "captainkube-binding" created
deployment.apps "captainkube" created
```

There were quite a few more resources created for `captainkube`, this is because `captainkube` uses a Kubernetes service account to talk to the cluster to fetch state. We have associated a set of permissions with the captainkube application which limits what captainkube can see from the underlying cluster.

Inspect the cluster, now that we have deployed a second application:

```console
kubectl get svc,deploy,pod
```

**Output:**
```console
NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
service/kubernetes   ClusterIP      10.0.0.1       <none>          443/TCP        1d
service/parrot       LoadBalancer   10.0.213.246   40.117.78.167   80:31801/TCP   10m

NAME                                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/captainkube   1         1         1            1           2m
deployment.extensions/parrot        1         1         1            1           10m

NAME                              READY     STATUS    RESTARTS   AGE
pod/captainkube-f55b77446-hf2pd   1/1       Running   0          2m
pod/parrot-77b5f95cc6-jjwp8       1/1       Running   0          10m
```

```console
kubectl get deploy,rs,po
```

**Output:**
```
NAME                                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/captainkube   1         1         1            1           9s

NAME                                           DESIRED   CURRENT   READY     AGE
replicaset.extensions/captainkube-7f4c784569   1         1         1         9s

NAME                               READY     STATUS    RESTARTS   AGE
pod/captainkube-7f4c784569-2vndh   1/1       Running   0          9s
```

## Deploy phippy and brady

Next let's add two more applications to our cluster. Phippy is a simple PHP application and Brady is written in Node.js. Both are happy to join our AKS cluster:

```console
kubectl apply -f manifests/phippy.yaml
kubectl apply -f manifests/brady.yaml
```

**Output:**
```
service "phippy" created
deployment.apps "phippy" created
service "nodebrady" created
deployment.apps "nodebrady" created
```

Great! You should see phippy and brady show up in the parrot web interface.

Next, let's [explore scaling our application](./03-scale-apps.md).