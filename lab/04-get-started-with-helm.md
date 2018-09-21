# Helm, the Kubernetes package manager

To this point, we have been using `kubectl` and static YAML manifests to create and delete resources from our Kubernetes cluster. This works well when we have small simple applications. As an application grows in complexity and size you often have 10s of manifests to describe an application. This could include:

* Deployments, Services, configuration ConfigMaps, Secrets

[Helm](https://helm.sh) is a tool that can be installed on any Kubernetes cluster. Helm provides a new concept called a Chart, which groups an arbitrary collection of resources into a logical unit. Let's install `helm` in our cluster:

## Installing Helm

As we did with the `captainkube` application earlier in the lab, let's configure a Service Account and RBAC bindings so Helm has permission to create, modify, and delete resources in our cluster:

```
kubectl -n kube-system create sa tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller
```

**Output**:
```
serviceaccount/tiller created
clusterrolebinding.rbac.authorization.k8s.io/tiller created

Creating /home/jason/.helm
Creating /home/jason/.helm/repository
Creating /home/jason/.helm/repository/cache
Creating /home/jason/.helm/repository/local
Creating /home/jason/.helm/plugins
Creating /home/jason/.helm/starters
Creating /home/jason/.helm/cache/archive
Creating /home/jason/.helm/repository/repositories.yamlAdding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /home/jason/.helm.
Happy Helming!
```

This launches an in-cluster component called `tiller` which interacts with the Kubernetes APIs. We are ready to install some charts.

## Use a chart

A chart is a structured collection of yaml files that contain the same information that a manifest might, but are easier to understand, can be templated to remove large amounts of duplicated information, enable dynamic configuration, and can be versioned to provide the Helm user with a wide array of sophisticated deployment control.

Anyone can create a chart, but to get started, let's use a chart that has been authored by the Helm community.

Let's install WordPress, a popular, open source blog. Let's update the list of available charts and look for WordPress:

```
helm repo update
```

**Output:**
```
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

Search for WordPress:
```
helm search wordpress
```

**Output**:
```
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
stable/wordpress        3.0.0           4.9.8           Web publishing platform for building blogs and websites.
```

Great! Looks like WordPress 4.9.8 is available. Let's install this chart on our cluster:

```
helm install stable/wordpress
```

As you can see, WordPress is more involved than our simple set of Phippy microservices. A few things to note:

* Helm has generated a `release` for this application, named `exciting-dragonfly`. A release is a Helm construct that tracks all of the resources for this chart under one entity. This lets us run multiple, independent instances of WordPress on the same cluster. While we have taken auto-generated names, you can pass a release name. For example, you could create a release for each of your blogs: `recipes` and `books`.
* Helm returns a list of all of the resources that were created for this chart. We see the usual suspects, Deployments and Services, but we also see some new resources:
    * Secrets, ConfigMap, and Volumes!

**Output**:
```
NAME:   exciting-dragonfly
LAST DEPLOYED: Thu Sep 20 17:39:57 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1beta1/Deployment
NAME                          DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
exciting-dragonfly-wordpress  1        1        1           0          1s
==> v1beta1/StatefulSet
NAME                        DESIRED  CURRENT  AGE
exciting-dragonfly-mariadb  1        1        1s

==> v1/Pod(related)
NAME                                          READY  STATUS   RESTARTS  AGE
exciting-dragonfly-wordpress-67d854fbb-8jzxm  0/1    Pending  0         1s
exciting-dragonfly-mariadb-0                  0/1    Pending  0         1s

==> v1/Secret
NAME                          TYPE    DATA  AGE
exciting-dragonfly-mariadb    Opaque  2     1s
exciting-dragonfly-wordpress  Opaque  2     1s

==> v1/ConfigMap
NAME                                     DATA  AGE
exciting-dragonfly-mariadb-init-scripts  1     1s
exciting-dragonfly-mariadb               1     1s
exciting-dragonfly-mariadb-tests         1     1s

==> v1/PersistentVolumeClaim
NAME                          STATUS   VOLUME   CAPACITY  ACCESS MODES  STORAGECLASS  AGE
exciting-dragonfly-wordpress  Pending  default  1s

==> v1/Service
NAME                          TYPE          CLUSTER-IP    EXTERNAL-IP  PORT(S)                     AGE
exciting-dragonfly-mariadb    ClusterIP     10.0.255.3    <none>       3306/TCP                    1s
exciting-dragonfly-wordpress  LoadBalancer  10.0.137.156  <pending>    80:31268/TCP,443:31563/TCP  1s


NOTES:
1. Get the WordPress URL:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w exciting-dragonfly-wordpress'

  export SERVICE_IP=$(kubectl get svc --namespace default exciting-dragonfly-wordpress -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
  echo "WordPress URL: http://$SERVICE_IP/"
  echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Login with the following credentials to see your blog

  echo Username: user
  echo Password: $(kubectl get secret --namespace default exciting-dragonfly-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
  ```

  The chart author has helpfully included useful notes. Follow along and connect to your new instance of WordPress.

  You might also want to explore some of these new resource types. A few commands to try:

  ```
  # List pods
  kubectl get pods
  # List application secrets
  kubectl get secrets
  # List persistent volumes
  kubectl get pv,pvc
  ```

  Check resource usage:
  ```
  kubectl top pod
  kubectl top node
  ```

Check out [https://hub.kubeapps.com/](https://hub.kubeapps.com/) for a set of community-maintained charts.

Next, let's scale our cluster capacity by [connecting AKS to Azure Container Instances](./05-extend-aks-with-aci.md).