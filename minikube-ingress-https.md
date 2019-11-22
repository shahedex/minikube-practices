# HTTPS TLS with NGINX-Ingress-Controller with Minikube

## Start your minikube

```console
$ minikube start
```

## Checkout the status if the Minikube is running

```console
$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

## Checkout the minikube addons if Ingress is enabled (Ingress is disabled by default)

```console
$ minikube addons list
- addon-manager: enabled
- dashboard: enabled
...
- ingress: disabled
...
- ingress-dns: disabled
- logviewer: disabled
```

## Enable Ingress (If Ingress is disabled)

```console
$ minikube addons enable ingress
ingress was successfully enabled
```

## Checkout the ingress-controller running status

```console
$ kubectl get pods -n kube-system
etcd-minikube                               1/1     Running   3          13d
kube-proxy-qhs7c                            1/1     Running   3          13d
kube-scheduler-minikube                     1/1     Running   3          13d
...
nginx-ingress-controller-6fc5bcc8c9-rwzsg   1/1     Running   0          156m
...
storage-provisioner                         1/1     Running   4          13d
```

## Run an example nginx deployment to checkout the example

```console
$ kubectl run nginx --image=nginx
deployment "nginx" created
```

## Checkout the deployment status

```console
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6db489d4b7-2w4fr   1/1     Running   0          154m
```

## Create a Service to expose the nginx deployment in the cluster

```console
$ kubectl expose deployment nginx --port 80
service "nginx" exposed
```

## Create a file named nginx-ingress.yaml to write the Ingress deployment

```console
$ vim nginx-ingress.yaml
```

## Paste the following YAML texts to the nginx-ingress.yaml file

```console
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx
spec:
  rules:
    - host: example.com
      http:
        paths:
          - backend:
              serviceName: nginx
              servicePort: 80
```

## Add example.com hostname at the end of your /etc/hosts file to map minikube ip to example.com

```console
$ echo "($minikube ip) example.com" | sudo tee -a /etc/hosts
```

## Apply the ingress file to Kubermentes

```console
$ kubectl apply -f nginx-ingress.yaml
ingress "nginx" configured
```

## Check that if your host is working with nginx

```console
$ curl example.com
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Voila! It's working!!

## Generate a self signed TLS certificate with OPENSSL

```console
$ openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout tls.key -out tls.crt -subj "/CN=example.com" -days=365
Generating a RSA private key
.....++++
....................................................................................................................................................................................................................................++++
writing new private key to 'tls.key'
-----
```

## Check if the key files are generated

```console
$ ls
nginx-ingress.yaml tls.crt tls.key
```

Cool! Private key and the Certificates are created.

## Create a TLS Secret to bind these keys with Kubernetes

```console
$ kubectl create secret tls example-com-tls --cert=tls.crt --key=tls.key
secret "example-com-tls" created
```

## Check out the generated secret with the keys in it

```console
$ kubectl get secret -o yaml
apiVersion: v1
items:
- apiVersion: v1
  data:
    ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQVRBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwdGFXNXAKYTNWaVpVTkJNQjRYRFRFNU1URXdPREEzTVRjd09Wb1hEVEk1TVRFd05qQTNNVGN3T1Zvd0ZURVRNQkVHQTFVRQpBeE1LYldsdWFXdDFZbVZEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTjFBCnJtSUJzWTZVK0xocFhSTm5kblNaMlRiczdZRC9GY25Y...
    namespace: ZGVmYXVsdA==
    token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklsY3lURVo2TlRsTldIaElhV3gxY1dVNGRVNTVSSE15VkZZMU5HZzNZazk1YVY4eVMwcHZhbVJOWVdjaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUprWldaaGRXeDBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1...
  kind: Secret
  metadata:
    annotations:
      kubernetes.io/service-account.name: default
      kubernetes.io/service-account.uid: 662d3e50-c8f2-4628-b492-e35a56f702c0
    creationTimestamp: "2019-11-09T07:17:49Z"
    name: default-token-4pntj
    namespace: default
    resourceVersion: "313"
    selfLink: /api/v1/namespaces/default/secrets/default-token-4pntj
    uid: 2f802621-7328-4057-900a-55c58ce1742a
  type: kubernetes.io/service-account-token
- apiVersion: v1
  data:
    tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUZEVENDQXZXZ0F3SUJBZ0lVSHE2MWYrMFJDSkEwanpzdmMrYWNnZjFnZUpFd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0ZqRVVNQklHQTFVRUF3d0xaWGhoYlhCc1pTNWpiMjB3SGhjTk1Ua3hNVEl5TURjek5qSTNXaGNOTWpBeApNVEl4TURjek5qSTNXakFXTVJRd0VnWURWUVFEREF0bGVHRnRjR3hsTG1OdmJUQ0NBaUl3RFFZSktvWklodmNOCkFRRUJCUUFEZ2dJUEFEQ0NBZ29DZ2dJQkFMcU1MVjVV...
    tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUpRZ0lCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQ1N3d2dna29BZ0VBQW9JQ0FRQzZqQzFlVkMyY1pVT0kKOGxlbGVxNFVPZGVtUVZSSHhzZStqTlc4YnlZNVl5cVg4Qkhzemo3cnFsK05aM0hjd21kMHFydzZnaHg4VHF3LwpoaXdOWHlzRlRXZ24wejRYUXBhKy9zUUNCUWlaNWJKdVdpZzJoRlhSTWxRdDN2QUtIR1IxamVTYmw1djdLREZ3ClZrVldRWHRicjErTDRobFR6S2Y3OUZWWHNKL3VEMW9kditQclBBOTM4UUxCSmV0R0dyQkE1T1BSb2NRU1RGcVoKQkQrNmlsV01oWSt6eHgyYTROSEU5ckM4SjJoR3Vhcm41cTBidnFWMWFhcXo0QkhDNGxmNHRzQWpnR1hrSzVhUwpYd3g3Tmd4clY4Vnc4SW9JQUc3aExvK2ZtQ0pNaX...
  kind: Secret
  metadata:
    creationTimestamp: "2019-11-22T07:38:50Z"
    name: example-com-tls
    namespace: default
    resourceVersion: "68324"
    selfLink: /api/v1/namespaces/default/secrets/example-com-tls
    uid: 7d7b50a8-a0eb-40d7-b7e8-ac0c5a78f568
  type: kubernetes.io/tls
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""

```

## Add the following YAML texts to the nginx-ingress.yaml file to enable TLS requests

```console
...
spec:
  tls:
    - secretName: example-com-tls
      hosts:
        - example.com
  rules:
    - host: example.com
   ...
```

## Apply the ingress file again to Kubernentes to update the existing rules

```console
$ kubectl apply -f nginx-ingress.yaml
ingress "nginx" configured
```

## Check if the TLS is working

```console
$ curl -k https://example.com
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Nice! Working fine!!
