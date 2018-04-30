# Shopping Cart Application with Microservices Approach

Building microservices application (Shopping Cart Application - Polyglot for services) using Kubernetes + Istio with its ecosystem parts.

> ### Disclamation
>
> * Should have `MINIKUBE_HOME` environment variable in your machine, and value should point to `C:\users\<your name>\`
> * Should run powershell script to create `minikube` machine in `C:` drive.
> * If it threw the exception that it couldn't find out `minikube` machine in Hyper-V so just simply delete everything in `<user>/.minikube` folder, but we could keep `cache` folder to avoid download everything from scratch, then runs it subsequently.

## Technical Stack

* [Hyper-V](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/hyper-v-technology-overview) or [VirtualBox](https://www.virtualbox.org)
* [Docker](https://www.docker.com)
* [Kubernetes](https://kubernetes.io) ([minikube](https://github.com/kubernetes/minikube) v0.25.2 for windows)
* [Istio](https://istio.io)
* [Ambassador](https://www.getambassador.io)
* [Weave Scope](https://www.weave.works) on Kubernetes
* [.NET Core SDK](https://www.microsoft.com/net/download/windows)
* [NodeJS](https://nodejs.org)
* [xip](http://xip.io) or [nip](http://nip.io) for access virtual
  hosts on your development web server from devices on your
  local network, like iPads, iPhones, and other computers.

## Setup Local Kubernetes

* Using `minikube` for `Windows` in this project, but you can use `Mac` or `Linux` version as well

* Download the appropriate package of your minikube at https://github.com/kubernetes/minikube/releases (Used `v0.25.2` for this project)

* Install it into your machine (Windows 10 in this case)

* After installed `minikube`, then run

`Hyper-V`

```
> minikube start --kubernetes-version="v1.9.0" --vm-driver=hyperv --hyperv-virtual-switch="minikube_switch" --cpus=4 --memory=4096 --v=999 --alsologtostderr
```

or starting with full option

```
> minikube start --vm-driver hyperv --kubernetes-version="v1.9.0" --hyperv-virtual-switch="minikube_switch" --memory 4096 --cpus=4 --extra-config=apiserver.authorization-mode=RBAC --extra-config=apiserver.Features.EnableSwaggerUI=true -extra-config=apiserver.Admission.PluginNames=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota --v=999 --alsologtostderr
```

`VirtualBox v5.2.8`

```
> minikube start --vm-driver="virtualbox" --kubernetes-version="v1.10.0" --cpus=4 --memory 4096 --extra-config=apiserver.authorization-mode=RBAC,apiserver.Features.EnableSwaggerUI=true,apiserver.Admission.PluginNames=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota --v=7 --alsologtostderr
```

## Setup Istio

* Download the appropriate package of Istio at https://github.com/istio/istio/releases

* Upzip it into your disk, let say `D:\istio\`

* `cd` into `D:\istio\`, then run

```
> kubectl create -f install/kubernetes/istio.yaml

or

> kubectl create -f install/kubernetes/istio-auth.yaml
```

![](https://github.com/thangchung/shopping-cart-k8s/blob/master/assets/default-istio-images.PNG)

**_Notes: set `istio\bin\istioctl.exe` to the `PATH` of the windows._**

## Setup Ambassador

* If you're running in a cluster with RBAC enabled:

```
> kubectl apply -f https://getambassador.io/yaml/ambassador/ambassador-rbac.yaml
```

* Without RBAC, you can use:

```
> kubectl apply -f https://getambassador.io/yaml/ambassador/ambassador-no-rbac.yaml
```

**_Notes: for some reason, I couldn't run the no-rbac mode on my local development._**

## Dashboard

```
> minikube dashboard
```

![](https://github.com/thangchung/shopping-cart-k8s/blob/master/assets/minikube-ui.PNG)

```
> kubectl get svc -n istio-system
```

```
> export GATEWAY_URL=$(kubectl get po -l istio-ingress -n istio-system -o jsonpath='{.items[0].status.hostIP}'):$(kubectl get svc istio-ingress -n istio-system -o jsonpath='{.spec.ports[0].nodePort}')
```

```
> curl $GETWAY_URL
```

## Build Our Own microservices

* Run

```
> minikube docker-env
```

* Copy and Run

```
> @FOR /f "tokens=*" %i IN ('minikube docker-env') DO @%i
```

From now on, we can type `docker images` to list out all images in Kubernetes local node.

* Build our microservices by running

```
> powershell -f build-all.ps1
```

* Then if you want to just test it then run following commands

```
> cd k8s
> kubectl apply -f shopping-cart.yaml
```

* In reality, we need to inject the **sidecards** into microservices as following

```
> cd k8s
> istioctl kube-inject -f shopping-cart.yaml | kubectl apply -f -
```

![](https://github.com/thangchung/shopping-cart-k8s/blob/master/assets/shopping-cart-images.PNG)

## Available Microservices

* Get host IP

```
> minikube ip
```

* Get Ambassador port

```
> kubectl get svc ambassador -o jsonpath='{.spec.ports[0].nodePort}'
```

* Finally, open browser with `<IP>:<PORT>`

* Microservices
  * Catalog service: `www.<IP>.xip.io:<PORT>/c/swagger/`. For example, http://www.192.168.1.6.xip.io:32097/c/swagger/
  * Supplier service: `www.<IP>.xip.io:<PORT>/s/`
  * Security service: `www.<IP>.xip.io:<PORT>/id/account/login` or `www.<IP>.xip.io:<PORT>/id/.well-known/openid-configuration`
  * Email service: `www.<IP>.xip.io:<PORT>/e/`

## Develop A New Service

* Build the whole application using

```
> powershell -f build-all.ps1
```

* Then run

```
> kubectl delete -f shopping-cart.yaml
```

* And

```
> kubectl apply -f shopping-cart.yaml
```

* Waiting a sec for Kubernetes to refresh.

## Metrics Collection, Distributed Tracing, and Visualization

### Setup Prometheus

TODO

### Setup Grafana

TODO

### Setup Service Graph

TODO

### Setup Zipkin or Jaeger

TODO

### Setup Weave Scope

* Install and run it on the local

```
> kubectl apply -f "https://cloud.weave.works/k8s/scope.yaml?k8s-version=v1.9.0"
```

* Then `port-forward` it out as following

```
> kubectl get -n weave pod --selector=weave-scope-component=app -o jsonpath='{.items..metadata.name}'
> kubectl port-forward -n <weave scope name> 4040
```

* Go to `http://localhost:4040`

### Tips and Tricks

* Print out environment variables in one container

```
> kubectl get pods
```

```
> kubectl exec <pod name> env
```
