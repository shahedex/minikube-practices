# HTTPS host with NGINX-Ingress-Controller with Minikube

### Start your minikube 
```console
$ minikube start
```

### Checkout the status if the Minikube is running
```console
$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

### Checkout the minikube addons if Ingress is enabled (Ingress is disabled by default)
```console
$ minikube addons list
- addon-manager: enabled
- dashboard: enabled
- default-storageclass: enabled
- __ingress: disabled__
- ingress-dns: disabled
- logviewer: disabled
- metrics-server: disabled
...

```

