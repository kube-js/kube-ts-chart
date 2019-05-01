# kube-ts-chart

### This chart is based on this: [tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes)

### Prerequisites

- A Kubernetes 1.10+ cluster with role-based access control (RBAC) enabled
- The kubectl command-line tool installed on your local machine and configured to connect to your cluster.
- A domain name and DNS A records which you can point to the DigitalOcean Load Balancer used by the Ingress.
- The Helm package manager installed on your local machine and Tiller installed on your cluster, as detailed in [How To Install Software on Kubernetes Clusters](https://www.digitalocean.com/community/tutorials/how-to-install-software-on-kubernetes-clusters-with-the-helm-package-manager) with the Helm Package Manager. Ensure that you are using Helm v2.12.1 or later or you may run into issues installing the cert-manager Helm chart.

### Installing tiller on your cluster

#### 1. Create the tiller serviceaccount:
```
kubectl -n kube-system create serviceaccount tiller
```

#### 2. Next, bind the tiller serviceaccount to the cluster-admin role:

```
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
```

#### 3. Now we can run helm init, which installs Tiller on our cluster, along with some local housekeeping tasks such as downloading the stable repo details:
```
helm init --service-account tiller
```

#### 4. Verify that tiller is running: 

```
kubectl get pods --namespace kube-system
```

### Setting Up the Kubernetes Nginx Ingress Controller

#### 1. Create mandatory resources:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
```

#### 2. Create ingress-nginx service using on of the following:

- free option using NodePort:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml
```

- paid option on digital ocean using a LoadBalancer (\$10/months):

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/cloud-generic.yaml
```

#### 3. Verify that either NodePort or LoadBalancer service has been successfuly create:

```
kubectl get svc --namespace=ingress-nginx
```

#### 4. You should see an external IP address, corresponding to the IP address of the NodePort / LoadBalancer service. Make a note of it and use in the next step.

```
Output
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)                      AGE
ingress-nginx   LoadBalancer   10.245.247.67   203.0.113.0   80:32486/TCP,443:32096/TCP   20h
```

### Installing and Configuring Cert-Manager

#### 1. Create the cert-manager Custom Resource Definitions (CRDs): 
```
kubectl apply  -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.6/deploy/manifests/00-crds.yaml
```

#### 2. We’ll add a label to the kube-system namespace, where we’ll install cert-manager, to enable advanced resource validation using a webhook:

```
kubectl label namespace kube-system certmanager.k8s.io/disable-validation="true"
```

#### 3. Install cert-manager helm chart into kube-system namespace:

```
helm install --name cert-manager --namespace kube-system stable/cert-manager --version v0.6.6
```


### Install kube-ts-chart

#### 1. Install [helm-github](https://github.com/sagansystems/helm-github) to allow install this chart (kube-ts-chart) directly from github:

```
helm plugin install --version master https://github.com/sagansystems/helm-github.git
```

#### 2. Install the chart
```
helm install --name my-release .
```

#### 2. Uninstall the chart
```
helm delete --purge my-release
```


```
helm github install --repo git@github.com:kube-js/kube-ts-chart.git
```
