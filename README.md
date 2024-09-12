# Setup Contour Ingress Controller
I have cluster working with a Nginx ingress controller.  I want to add a second ingress controller.  The Contour/Envoy ingress controller was recommended.

In these notes, I record the steps to install Contour/Envoy, verify basic ingress functionality and try-out the new HttpProxy controller  built into Contour. 

# Install Contour and Envoy
Following the instructions on the [getting started page](https://projectcontour.io/getting-started/), install contour from the [project website](https://projectcontour.io/).
```
jkozik@knode202:~/contour$ kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
namespace/projectcontour created
serviceaccount/contour created
serviceaccount/envoy created
configmap/contour created
customresourcedefinition.apiextensions.k8s.io/contourconfigurations.projectcontour.io created
customresourcedefinition.apiextensions.k8s.io/contourdeployments.projectcontour.io created
customresourcedefinition.apiextensions.k8s.io/extensionservices.projectcontour.io created
customresourcedefinition.apiextensions.k8s.io/httpproxies.projectcontour.io created
customresourcedefinition.apiextensions.k8s.io/tlscertificatedelegations.projectcontour.io created
serviceaccount/contour-certgen created
rolebinding.rbac.authorization.k8s.io/contour created
role.rbac.authorization.k8s.io/contour-certgen created
job.batch/contour-certgen-v1-30-0 created
clusterrolebinding.rbac.authorization.k8s.io/contour created
rolebinding.rbac.authorization.k8s.io/contour-rolebinding created
clusterrole.rbac.authorization.k8s.io/contour created
role.rbac.authorization.k8s.io/contour created
service/contour created
service/envoy created
deployment.apps/contour created
daemonset.apps/envoy created

jkozik@knode202:~/contour$ kubectl get svc,pods,ing,daemonset -n projectcontour
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/contour   ClusterIP      10.97.223.186   <none>        8001/TCP                     92m
service/envoy     LoadBalancer   10.100.173.66   <pending>     80:30825/TCP,443:31400/TCP   92m

NAME                                READY   STATUS      RESTARTS   AGE
pod/contour-66486bf57-ctfs2         1/1     Running     0          92m
pod/contour-66486bf57-d4pzx         1/1     Running     0          92m
pod/contour-certgen-v1-30-0-25jqp   0/1     Completed   0          92m
pod/envoy-62wg4                     2/2     Running     0          92m
pod/envoy-ggl82                     2/2     Running     0          92m
pod/envoy-k8rf8                     2/2     Running     0          92m

NAME                   DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/envoy   3         3         3       3            3           <none>          92m
jkozik@knode202:~/contour$ kubectl get svc,pods,ing,daemonset,deployment,crd -n projectcontour
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/contour   ClusterIP      10.97.223.186   <none>        8001/TCP                     95m
service/envoy     LoadBalancer   10.100.173.66   <pending>     80:30825/TCP,443:31400/TCP   95m

NAME                                READY   STATUS      RESTARTS   AGE
pod/contour-66486bf57-ctfs2         1/1     Running     0          95m
pod/contour-66486bf57-d4pzx         1/1     Running     0          95m
pod/contour-certgen-v1-30-0-25jqp   0/1     Completed   0          95m
pod/envoy-62wg4                     2/2     Running     0          95m
pod/envoy-ggl82                     2/2     Running     0          95m
pod/envoy-k8rf8                     2/2     Running     0          95m

NAME                   DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/envoy   3         3         3       3            3           <none>          95m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/contour   2/2     2            2           95m

NAME                                                                                                CREATED AT
customresourcedefinition.apiextensions.k8s.io/apiservers.operator.tigera.io                         2024-05-18T00:15:59Z
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org               2024-05-18T00:15:58Z
customresourcedefinition.apiextensions.k8s.io/bgpfilters.crd.projectcalico.org                      2024-05-18T00:15:58Z
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org                        2024-05-18T00:15:58Z
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org                 2024-05-18T00:15:58Z
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org              2024-05-18T00:15:58Z
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org             2024-05-18T00:15:58Z
customresourcedefinition.apiextensions.k8s.io/contourconfigurations.projectcontour.io               2024-09-10T22:21:10Z
customresourcedefinition.apiextensions.k8s.io/contourdeployments.projectcontour.io                  2024-09-10T22:21:10Z
customresourcedefinition.apiextensions.k8s.io/extensionservices.projectcontour.io                   2024-09-10T22:21:10Z
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org             2024-05-18T00:15:58Z
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org           2024-05-18T00:15:58Z
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org               2024-05-18T00:15:58Z
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org                   2024-05-18T00:15:58Z
customresourcedefinition.apiextensions.k8s.io/httpproxies.projectcontour.io                         2024-09-10T22:21:10Z
customresourcedefinition.apiextensions.k8s.io/imagesets.operator.tigera.io                          2024-05-18T00:15:59Z
customresourcedefinition.apiextensions.k8s.io/installations.operator.tigera.io                      2024-05-18T00:15:59Z
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org                      2024-05-18T00:15:58Z
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org                     2024-05-18T00:15:58Z
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org                     2024-05-18T00:15:58Z
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org                         2024-05-18T00:15:59Z
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org                  2024-05-18T00:15:59Z
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org   2024-05-18T00:15:59Z
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org                 2024-05-18T00:15:59Z
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org                     2024-05-18T00:15:59Z
customresourcedefinition.apiextensions.k8s.io/tigerastatuses.operator.tigera.io                     2024-05-18T00:15:59Z
customresourcedefinition.apiextensions.k8s.io/tlscertificatedelegations.projectcontour.io           2024-09-10T22:21:11Z
jkozik@knode202:~/contour$
```
Notes about above:
- Envoy is installed as a daemonset across each of the nodes of my cluster.  This is the data plane.  It carries the traffice
- Contour is deployed redundantly as a deployment. This is the controller that parses the Ingress objects and configures Envoy.
- The envoy service is the LoadBalancer port, specifically `80:30825/TCP,443:31400/TCP`, that takes in (ingresses) traffice from outside of the cluster.

## Envoy Loadbalancer -> NodePort
In my cluster, I am self hosted and I don't support the LoadBalancer type.  If I was using Contour on AWS, I would use AWS's loadbalancer service, but I have an external reverse proxy that routes traffic on my Home LAN to my cluster.  This external reverse proxy also routes traffic to other docker contains and good old web servers.  

I need to reconfigure Envoy to expose a NodePort.  In the NodePort Service section of the [Deployment Options](https://projectcontour.io/docs/1.21/deploy-options/) documentation says `you can change the Envoy Service in the 02-service-envoy.yaml file and set type to NodePort.`
```
jkozik@knode202:~/contour.sav/contour/examples/contour$ kubectl get svc -n projectcontour envoy -ojson | jq '.spec.type'
"LoadBalancer"

jkozik@knode202:~/contour.sav/contour/examples/contour$  kubectl patch svc -n projectcontour envoy -p  '{"spec": {"type": "NodePort"}}'
service/envoy patched

jkozik@knode202:~/contour.sav/contour/examples/contour$ kubectl get svc -n projectcontour envoy -ojson | jq '.spec.type'
"NodePort"

jkozik@knode202:~/contour.sav/contour/examples/contour$ kubectl get svc -n projectcontour
NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
contour   ClusterIP   10.97.223.186   <none>        8001/TCP                     140m
envoy     NodePort    10.100.173.66   <none>        80:30825/TCP,443:31400/TCP   140m
```
With simple patch, Contour is ready for testing.  

## Test Basic Ingress
Let's create a dummy application and verify that basic Ingress is working.  Borrowing from the [Contour documentation[(https://projectcontour.io/docs/v1.9.0/deploy-options/), let's setup Kuard. Kuard is a [Demo app for Kubernetes Up and Running book](https://github.com/kubernetes-up-and-running/kuard) that I've seen in a few places.  It serves as a good hello-world application that includes several useful troubleshooting tools. 

### download kuard.yaml, edit LoadBalancer -> NodePort
I am using the kuard.yaml file from the [Contour repository](https://github.com/projectcontour/contour). I found it in the [site/content/examples] directory. 

In my case, I am using an external reverse proxy.  I need NodePort not LoadBalancer support, so I edit the service type.
```
jkozik@knode202:~/contour$ wget https://projectcontour.io/examples/kuard.yaml
--2024-09-11 03:37:07--  https://projectcontour.io/examples/kuard.yaml
Resolving projectcontour.io (projectcontour.io)... 3.18.31.67, 3.139.180.21
Connecting to projectcontour.io (projectcontour.io)|3.18.31.67|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 709 [application/x-yaml]
Saving to: ‘kuard.yaml’

kuard.yaml                              100%[============================================================================>]     709  --.-KB/s    in 0s

2024-09-11 03:37:08 (263 MB/s) - ‘kuard.yaml’ saved [709/709]

jkozik@knode202:~/contour$ sed -i'' "s/type: ClusterIP/type: NodePort/" kuard.yaml
```
It is useful to look and the Ingress manifest.  Note:  even though Contour is a total different Ingress, it still supports the standard [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) object.  I have been using the [Nginx Ingress Controller](https://docs.nginx.com/nginx-ingress-controller/installation/installing-nic/installation-with-manifests/) and this looks normal to me. 
```
jkozik@knode202:~/contour$ cat kuard.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: kuard
  name: kuard
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kuard
  template:
    metadata:
      labels:
        app: kuard
    spec:
      containers:
      - image: gcr.io/kuar-demo/kuard-amd64:1
        name: kuard
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: kuard
  name: kuard
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: kuard
  sessionAffinity: None
  type: NodePort
---
apiVersion: networking.k8s.io/v1        # <--------  Ingress
kind: Ingress
metadata:
  name: kuard
  labels:
    app: kuard
spec:
  defaultBackend:
    service:
      name: kuard
      port:
        number: 80
jkozik@knode202:~/contour$
```
### create the Ingress
```
jkozik@knode202:~/contour$ kubectl apply -f kuard.yaml
deployment.apps/kuard created
service/kuard created
ingress.networking.k8s.io/kuard created

jkozik@knode202:~/contour$ kubectl get svc,ing,deployment,pods | grep kuard
service/kuard             NodePort    10.109.111.89    <none>        80:31386/TCP   11h
ingress.networking.k8s.io/kuard            <none>   *                                           80      11h
deployment.apps/kuard                    3/3     3            3           11h
pod/kuard-7cd5b94b5c-2hsn8                    1/1     Running     0                11h
pod/kuard-7cd5b94b5c-4hpsh                    1/1     Running     0                11h
pod/kuard-7cd5b94b5c-k4gcn                    1/1     Running     0                11h

jkozik@knode202:~/contour$ kubectl get ing kuard
NAME    CLASS    HOSTS   ADDRESS   PORTS   AGE
kuard   <none>   *                 80      11h
jkozik@knode202:~/contour$
```
Note:  The Ingress was defined without an ingress class.  I already have one Ingress Controller installed with a class name "nginx".  I need to give my new one a class name also.
```
jkozik@knode202:~/contour$ sed -i'' '$a\  ingressClassName: contour
' kuard.yaml

jkozik@knode202:~/contour$ kubectl apply -f kuard.yaml
deployment.apps/kuard unchanged
service/kuard unchanged
ingress.networking.k8s.io/kuard configured

jkozik@knode202:~/contour$ kubectl get ing kuard
NAME    CLASS     HOSTS   ADDRESS   PORTS   AGE
kuard   contour   *                 80      11h
jkozik@knode202:~/contour$
```
### Verify
First I want to curl the kuard application through its NodePort service.  
#### curl kuard NodePort
```
jkozik@knode202:~/contour$ kubectl get svc,ing kuard
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kuard   NodePort   10.109.111.89   <none>        80:31386/TCP   11h

NAME                              CLASS     HOSTS   ADDRESS   PORTS   AGE
ingress.networking.k8s.io/kuard   contour   *                 80      11h

jkozik@knode202:~/contour$ kubectl get nodes -o wide
NAME       STATUS   ROLES           AGE    VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
knode200   Ready    worker          116d   v1.29.1   192.168.100.200   <none>        Ubuntu 22.04.4 LTS   5.15.0-107-generic   containerd://1.7.13
knode201   Ready    worker          116d   v1.29.1   192.168.100.201   <none>        Ubuntu 22.04.4 LTS   5.15.0-107-generic   containerd://1.7.13
knode202   Ready    control-plane   116d   v1.29.5   192.168.100.202   <none>        Ubuntu 22.04.4 LTS   5.15.0-112-generic   containerd://1.7.13
knode203   Ready    worker          102d   v1.29.1   192.168.100.203   <none>        Ubuntu 22.04.3 LTS   5.15.0-112-generic   containerd://1.7.13
jkozik@knode202:~/contour$

jkozik@knode202:~/contour$ curl http://192.168.100.200:31386
<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8">

  <title>KUAR Demo</title>

  <link rel="stylesheet" href="/static/css/bootstrap.min.css">
  <link rel="stylesheet" href="/static/css/styles.css">

  <script>
var pageContext = {"hostname":"kuard-7cd5b94b5c-4hpsh","addrs":["10.10.12.120"],"version":"v0.8.1-1","versionColor":"hsl(18,100%,50%)","requestDump":"GET / HTTP/1.1\r\nHost: 192.168.100.200:31386\r\nAccept: */*\r\nUser-Agent: curl/7.81.0","requestProto":"HTTP/1.1","requestAddr":"192.168.100.200:34166"}
  </script>
</head>


<svg style="position: absolute; width: 0; height: 0; overflow: hidden;" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
<defs>
<symbol id="icon-power" viewBox="0 0 32 32">
<title>power</title>
<path class="path1" d="M12 0l-12 16h12l-8 16 28-20h-16l12-12z"></path>
</symbol>
<symbol id="icon-notification" viewBox="0 0 32 32">
<title>notification</title>
<path class="path1" d="M16 3c-3.472 0-6.737 1.352-9.192 3.808s-3.808 5.72-3.808 9.192c0 3.472 1.352 6.737 3.808 9.192s5.72 3.808 9.192 3.808c3.472 0 6.737-1.352 9.192-3.808s3.808-5.72 3.808-9.192c0-3.472-1.352-6.737-3.808-9.192s-5.72-3.808-9.192-3.808zM16 0v0c8.837 0 16 7.163 16 16s-7.163 16-16 16c-8.837 0-16-7.163-16-16s7.163-16 16-16zM14 22h4v4h-4zM14 6h4v12h-4z"></path>
</symbol>
</defs>
</svg>

<body>
  <div id="root"></div>
  <script src="/built/bundle.js" type="text/javascript"></script>
</body>
</html>
jkozik@knode202:~/contour$
```
Note: the curl command `curl http://192.168.100.200:31386` used the LAN IP address of my cluster.  It is the address of one of my worker nodes.  I believe I could have picked any address.  The port number, 31386, is the NodePort of the kuard service.  I is good to verify that one can curl it.  But it did not go throught the Contour/Envoy layer.  

#### curl kuard through the Contour/Envoy NodePort
Let verify kuard through the envoy proxy.
```
jkozik@knode202:~/contour$ kubectl get svc -n projectcontour
NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
contour   ClusterIP   10.97.223.186   <none>        8001/TCP                     17h
envoy     NodePort    10.100.173.66   <none>        80:30825/TCP,443:31400/TCP   17h


jkozik@knode202:~/contour$ curl 192.168.100.200:30825 | head
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1685  100  1685    0     0   124k      0 --:--:-- --:--:-- --:--:--  137k
<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8">

  <title>KUAR Demo</title>

  <link rel="stylesheet" href="/static/css/bootstrap.min.css">
  <link rel="stylesheet" href="/static/css/styles.css">
jkozik@knode202:~/contour$
```
Same output.  It took me a while to figure this out. One can go directly to a service's NodePort, with no need for a proxy.  In my case, I have an external reverse proxy.  It really only wants to know one address/port for my whole cluster.  Every time I add or change a service, I do not want to reprogram my reverse proxy.  

But note also, the Ingress object specified in the kuard.yaml file did not have a path or host. That means all traffic to my Contour/Envoy ingress goes to the kuard service.  This needs to be fixed. 

### setup external domain kuard.kozik.net
For me, a full end-to-end test of a ingress goes all the way to the Internet.  As is my usual practice, I defined a subdomain name, mapped it to my home LAN's external IP address and configured my reverse proxy to map the incoming traffic to my kubernetes cluster's Contour ingress NodePort.

#### Setup a sub-domain in cloudflare
I use Cloudflare as my DNS provider; the configuration looks like this.
![image](https://github.com/user-attachments/assets/52f84721-6f73-43f8-8869-e8c2cd2bf22a)
I setup kuard as a subdomain.  I map it to my home IP address.
```
jkozik@knode202:~/contour$ dig +noall +answer kuard.kozik.net
kuard.kozik.net.        251     IN      A       69.243.XXX.XXX
jkozik@knode202:~/contour$
```
All my incoming port 80 and 443 go to an Apache httpd reverse proxy.  I have setup a virtual host for kuard.kozik.net.  It works for both port 80 and 443.  I just show an excerpt here:
##### kuard.kozik.net reverse proxy config  (partial)
```
[root@dell1 sites-enabled]# cat proxy.kuard.kozik.net-le-ssl.conf
<IfModule mod_ssl.c>
<VirtualHost *:443>
 ServerName kuard.kozik.net
 ServerAlias www.kuard.kozik.net

 RewriteEngine on
 #RewriteCond %{SERVER_NAME} =www.kuard.kozik.net
 #ReWriteRule ^ https://kuard.kozik.net%{REQUEST_URI} [END,QSA,R=permanent]
 RewriteCond %{HTTP:Upgrade} =websocket [NC]
 RewriteRule /(.*)           ws://192.168.100.200:30825//$1 [P,L]
 RewriteCond %{HTTP:Upgrade} !=websocket [NC]
 RewriteRule /(.*)           http://192.168.100.200:30825/$1 [P,L]

 Header always set X-Frame-Options SAMEORIGIN

 ProxyPreserveHost On
 ProxyPass /.well-known !
 ProxyPass /  http://192.168.100.200:30825/
 ProxyPassReverse /  http://192.168.100.200:30825/

   ErrorLog logs/kuard.kozik.net-error_log
   CustomLog logs/kuard.kozik.net-access_log combined

SSLCertificateFile /etc/letsencrypt/live/kozik.net/cert.pem
SSLCertificateKeyFile /etc/letsencrypt/live/kozik.net/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf
SSLCertificateChainFile /etc/letsencrypt/live/kozik.net/chain.pem
</VirtualHost>
</IfModule>
```
Note:  Above the incoming https/443 traffic is terminated and redirected over port 80 to the envoy proxy on port 30825 of the kubernetes cluster.  I have decided to terminate all https traffic at the reverse proxy because I sometime have multiple destinations in my Home LAN that are not all on the kubernetes cluster. For example, I have some docker containers and some web pages on other segments of my home LAN.  Thus it is easier to terminate my SSL in one place.  I only have one IP address for the whole home LAN.

Once this is setup, verify http://kuard.kozik.net...

From the command line:
```
jkozik@knode202:~/contour$ curl https://kuard.kozik.net | head
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1790  100  1790    0     0  10272      0 --:--:-- --:--:-- --:--:-- 10346
<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8">

  <title>KUAR Demo</title>

  <link rel="stylesheet" href="/static/css/bootstrap.min.css">
  <link rel="stylesheet" href="/static/css/styles.css">
jkozik@knode202:~/contour$
```
and from the browser:
![image](https://github.com/user-attachments/assets/7ffaf441-3f22-44ad-a5ce-67a640851ffd)

Note: I have my reverse proxy to handle both http and https traffic.  http://kuard.kozik.net redirects to https://kuard.kozik.net.  That then redirects to http://192.168.100.200:30825 (envoy NodePort).

## Add a path to hello-world  https://kuard.kozik.net/hello-world
I borrowed a `hello-world` applicatinon from [bhargavshah86/kube-test(https://github.com/skynet86/kube-test-dockerfile).  I want to spin it up and add a path to it in my new Contour Ingress
### Spin up Hello World
```
jkozik@knode202:~/contour$ kubectl apply -f hello-world.yaml
deployment.apps/hello-world-deployment created
service/hello-world created

jkozik@knode202:~/contour$ kubectl get svc,deployment,pod | grep hello
service/hello-world       NodePort    10.101.234.115   <none>        80:30081/TCP   53s
deployment.apps/hello-world-deployment   2/2     2            2           53s
pod/hello-world-deployment-557bcf8cbf-6q9n5   1/1     Running     0                53s
pod/hello-world-deployment-557bcf8cbf-vgj9p   1/1     Running     0                53s
jkozik@knode202:~/contour$
```
Let's create a new Ingress manifest for hello-world


# References
- [Deployment Options](https://projectcontour.io/docs/1.21/deploy-options/)
- [Getting Started with Contour](https://projectcontour.io/getting-started/)
- [Github repository: Contour is a Kubernetes ingress controller using Envoy proxy.](https://github.com/projectcontour/contour)
- [Demo app for Kubernetes Up and Running book](https://github.com/kubernetes-up-and-running/kuard)
