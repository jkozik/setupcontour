# Setup Contour Gateway API Dynamic
I am in the process of retiring my Ingress infrastucture and replacing it with Gateway API.  I picked [Contour](https://projectcontour.io/).

Following the instructions in the [Using Gateway API with Contour](https://projectcontour.io/docs/main/guides/gateway-api/) guide, I installed Contour Gateway API, create a gateway and test it with a few sample deployments.

## Install Contour, Dynamically provisioned
Following the [Option #2: Dynamically provisioned](https://projectcontour.io/docs/main/guides/gateway-api/#:~:text=Option%20%232%3A%20Dynamically%20provisioned) steps as follows:

```
jkozik@knode202:~/contour$ kubectl apply -f https://projectcontour.io/quickstart/contour-gateway-provisioner.yaml
customresourcedefinition.apiextensions.k8s.io/contourconfigurations.projectcontour.io configured
customresourcedefinition.apiextensions.k8s.io/contourdeployments.projectcontour.io configured
customresourcedefinition.apiextensions.k8s.io/extensionservices.projectcontour.io configured
customresourcedefinition.apiextensions.k8s.io/httpproxies.projectcontour.io configured
customresourcedefinition.apiextensions.k8s.io/tlscertificatedelegations.projectcontour.io configured
customresourcedefinition.apiextensions.k8s.io/backendlbpolicies.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/backendtlspolicies.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/grpcroutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/tcproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/tlsroutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/udproutes.gateway.networking.k8s.io created
namespace/projectcontour unchanged
serviceaccount/contour-gateway-provisioner created
clusterrole.rbac.authorization.k8s.io/contour-gateway-provisioner created
role.rbac.authorization.k8s.io/contour-gateway-provisioner created
rolebinding.rbac.authorization.k8s.io/contour-gateway-provisioner-leader-election created
clusterrolebinding.rbac.authorization.k8s.io/contour-gateway-provisioner created
deployment.apps/contour-gateway-provisioner created
jkozik@knode202:~/contour$
```

### Create a GatewayClass
```
jkozik@knode202:~/contour$ kubectl apply -f - <<EOF
kind: GatewayClass
apiVersion: gateway.networking.k8s.io/v1
metadata:
  name: contour
spec:
  controllerName: projectcontour.io/gateway-controller
EOF
gatewayclass.gateway.networking.k8s.io/contour created
jkozik@knode202:~/contour$
```

### Create a Gateway
```
jkozik@knode202:~/contour$ kubectl apply -f - <<EOF
kind: Gateway
apiVersion: gateway.networking.k8s.io/v1
metadata:
  name: contour
  namespace: projectcontour
spec:
  gatewayClassName: contour
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: All
EOF
gateway.gateway.networking.k8s.io/contour created

jkozik@knode202:~/contour$ kubectl get gateway,gatewayclass -A
NAMESPACE        NAME                                        CLASS     ADDRESS   PROGRAMMED   AGE
projectcontour   gateway.gateway.networking.k8s.io/contour   contour             True         110s

NAMESPACE   NAME                                             CONTROLLER                             ACCEPTED   AGE
            gatewayclass.gateway.networking.k8s.io/contour   projectcontour.io/gateway-controller   True       4m8s
jkozik@knode202:~/contour$
```
### NodePort
By default, Contour sets up Envoy to connect to a LoadBalancer.  I don't have one of those.  I have an external reverse proxy that wants to connect to my cluster through a NodePort.  Based on the [NodePort Service](https://projectcontour.io/docs/1.21/deploy-options/#:~:text=to%20run%20Contour.-,NodePort%20Service,-If%20your%20cluster) note, NodePort can easily be considered by editting the yaml for the envoy service. 
```
jkozik@knode202:~/contour$ kubectl get svc -n projectcontour
NAME              TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
contour-contour   ClusterIP      10.105.255.210   <none>        8001/TCP       17h
envoy-contour     LoadBalancer   10.109.188.45    <pending>     80:31182/TCP   17h
jkozik@knode202:~/contour$
```
Change LoadBalancer > NodePort
```
jkozik@knode202:~/contour$ kubectl edit svc envoy-contour -nprojectcontour
service/envoy-contour edited
jkozik@knode202:~/contour$ kubectl get svc -n projectcontour
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
contour-contour   ClusterIP   10.105.255.210   <none>        8001/TCP       17h
envoy-contour     NodePort    10.109.188.45    <none>        80:31182/TCP   17h
jkozik@knode202:~/contour$
```
## Verify, Install kuard service
Using the [Demo app for Kubernetes Up and Running book](https://github.com/kubernetes-up-and-running/kuard) as a test app, let's verify that the basic Gateway API HttpRoute enabled by Contour
### Deploy kuard
```
jkozik@knode202:~/contour/deployment$ kubectl apply -f kuard.yaml
deployment.apps/kuard created
service/kuard created
```
### Deploy httproute 
The HttpRoute for an application based on host name is pretty straight forward.  Not that different from an Ingress. Below, I display the yaml, apply it and run a curl test to verify that it works. 
```
jkozik@knode202:~/contour$ cat 01-kuardkoziknet-httproute.yaml
kind: HTTPRoute
apiVersion: gateway.networking.k8s.io/v1
metadata:
  name: kuard
  namespace: default
  labels:
    app: kuard
spec:
  parentRefs:
  - group: gateway.networking.k8s.io
    kind: Gateway
    name: contour
    namespace: projectcontour
  hostnames:
  - "kuard.kozik.net"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - kind: Service
      name: kuard
      port: 80

jkozik@knode202:~/contour$ kubectl apply -f 01-kuardkoziknet-httproute.yaml
httproute.gateway.networking.k8s.io/kuard created

jkozik@knode202:~/contour$ curl --verbose -H "Host: kuard.kozik.net" http://192.168.100.200:31182
*   Trying 192.168.100.200:31182...
* Connected to 192.168.100.200 (192.168.100.200) port 31182 (#0)
> GET / HTTP/1.1
> Host: kuard.kozik.net
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< content-length: 1678
< content-type: text/html
< date: Thu, 26 Sep 2024 21:50:42 GMT
< x-envoy-upstream-service-time: 4
< vary: Accept-Encoding
< server: envoy
<
<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8">

  <title>KUAR Demo</title>

  <link rel="stylesheet" href="/static/css/bootstrap.min.css">
  <link rel="stylesheet" href="/static/css/styles.css">

  <script>
var pageContext = {"hostname":"kuard-7cd5b94b5c-tnftl","addrs":["10.10.75.196"],"version":"v0.8.1-1","versionColor":"hsl(18,100%,50%)","requestDump":"GET / HTTP/1.1\r\nHost: kuard.kozik.net\r\nAccept: */*\r\nUser-Agent: curl/7.81.0\r\nX-Envoy-Expected-Rq-Timeout-Ms: 15000\r\nX-Envoy-Internal: true\r\nX-Forwarded-For: 192.168.100.202\r\nX-Forwarded-Proto: http\r\nX-Request-Id: b574dc4a-b56a-4d51-8f32-96814f448c8d\r\nX-Request-Start: t=1727387442.162","requestProto":"HTTP/1.1","requestAddr":"10.10.12.122:58270"}
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
* Connection #0 to host 192.168.100.200 left intact
jkozik@knode202:~/contour$
```
### Access https://kuard.kozik.net from browser
I setup a domain name on cloudflare and setup my reverse proxy (external to my cluster).  I have it pointing to http://192.168.100.200:31182.  

From a browser, I verify it.
![image](https://github.com/user-attachments/assets/c13c9054-5004-48de-bbde-ba8a44a05c56)

## Other HttpRoutes using sub-domain host, path, filter, URLRewrite, replacePrefixMatch
To get Ingress to work, I needed to use Annotations.  The Nginx Ingress came with a vast selection.  I wanted to verify that the HttpRoute object lets me recreate everything I used from Ingress.
First, I need to be able to have separate routes for different subdomains.
### k8s.kozik.net
This deployment and httproute defines a Chuck Norris quote service at this URL.
```
jkozik@knode202:~/contour/deployment$ kubectl apply -f 02_deployment_chuck.yaml
deployment.apps/chuck-norris-quote-service created
service/chuck-norris-quote-service unchanged

jkozik@knode202:~/contour$ cat 02-k8koziknet-httproute-host-chuck.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: chuck-host
spec:
  parentRefs:
  - group: gateway.networking.k8s.io
    kind: Gateway
    name: contour
    namespace: projectcontour
  hostnames:
    - "k8s.kozik.net"
  rules:
  - backendRefs:
    - name: chuck-norris-quote-service
      port: 8080

jkozik@knode202:~/contour$ kubectl apply -f 02-k8koziknet-httproute-host-chuck.yaml
httproute.gateway.networking.k8s.io/chuck-host created

jkozik@knode202:~/contour$ curl --verbose -H "Host: k8s.kozik.net" http://192.168.100.200:31182
*   Trying 192.168.100.200:31182...
* Connected to 192.168.100.200 (192.168.100.200) port 31182 (#0)
> GET / HTTP/1.1
> Host: k8s.kozik.net
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< x-pod-name: chuck-norris-quote-service-645d6889f-jcc6b
< x-pod-namespace: default
< content-type: application/json
< content-length: 73
< x-envoy-upstream-service-time: 105
< vary: Accept-Encoding
< date: Thu, 26 Sep 2024 22:24:57 GMT
< server: envoy
<
* Connection #0 to host 192.168.100.200 left intact
{"message":"Chuck Norris rewrote the Google search engine from scratch."}
jkozik@knode202:~/contour$
```
Note: The HTTPRoute is very straight forward.  The hosts k8s.kozik.net and kuard.kozik.net are both supported in two separate HTTPRoutes.

### future.k8s.kozik.net with path filters
I also deployed a Back-To-The-Future quotes application.  I setup an HTTPRoute to it through a sub-sub-domain, future.k8s.kozik.net.  Then I defined a path to the Chuck-Norris quotes application future.k8s.kozik.net/chuck.
### deploy back-to-the-future app

```
jkozik@knode202:~/contour/deployment$ kubectl apply -f 02_deployment_back2future.yaml
deployment.apps/back2future-quote-service created
service/back2future-quote-service unchanged

jkozik@knode202:~/contour/deployment$ curl --verbose -H "Host: future.k8s.kozik.net" http://192.168.100.200:31182
*   Trying 192.168.100.200:31182...
* Connected to 192.168.100.200 (192.168.100.200) port 31182 (#0)
> GET / HTTP/1.1
> Host: future.k8s.kozik.net
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< x-pod-name: back2future-quote-service-56c5bbcd9b-jglvs
< x-pod-namespace: default
< content-type: application/json
< content-length: 171
< x-envoy-upstream-service-time: 170
< vary: Accept-Encoding
< date: Sat, 28 Sep 2024 02:07:44 GMT
< server: envoy
<
* Connection #0 to host 192.168.100.200 left intact
{"message":"Oh. One other thing. If you guys ever have kids, and one of them, when he's eight years old, accidentally sets fire to the living room rug... go easy on him."}
jkozik@knode202:~/contour/deployment$
```
### Verify future.k8s.kozik.net/chuck
The HttpRoute adds a match to a path /chuck.  Verify it, below
```
jkozik@knode202:~/contour$ curl --verbose -H "Host: future.k8s.kozik.net" http://192.168.100.200:31182/chuck
*   Trying 192.168.100.200:31182...
* Connected to 192.168.100.200 (192.168.100.200) port 31182 (#0)
> GET /chuck HTTP/1.1
> Host: future.k8s.kozik.net
> User-Agent: curl/7.81.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< x-pod-name: chuck-norris-quote-service-645d6889f-jcc6b
< x-pod-namespace: default
< content-type: application/json
< content-length: 83
< x-envoy-upstream-service-time: 4
< vary: Accept-Encoding
< date: Sat, 28 Sep 2024 02:25:48 GMT
< server: envoy
<
* Connection #0 to host 192.168.100.200 left intact
{"message":"Chuck Norris can spawn threads that complete before they are started."}
jkozik@knode202:~/contour$
```
Look at the file `05-httproute-host-future-path-chuck-hello.yaml` it has examples HttpRoutes using sub-domain host, path, filter, URLRewrite, replacePrefixMatch.

I found examples hard to find.  In the references, I kept points to example HttpRoutes.  I found the Contour and the official Gateway API documentation a little vague.  Sorry. 

# References
- [Using Gateway API with Contour](https://projectcontour.io/docs/main/guides/gateway-api/)
- [Configure an HttpRoute](https://projectcontour.io/docs/main/guides/gateway-api/#configure-an-httproute:~:text=Configure%20an%20HTTPRoute)
- [HTTPRoute API Reference](https://www.gateway-api-controller.eks.aws.dev/dev/api-types/http-route/)
- [Traefik & Kubernetes with Gateway API](https://doc.traefik.io/traefik/routing/providers/kubernetes-gateway/)
- [HTTPRoute resource fields](https://yandex.cloud/en/docs/application-load-balancer/k8s-ref/http-route?utm_referrer=https%3A%2F%2Fyndx.auth.yandex.cloud%2F)
- [Gateway API HttpRoute Spec](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io%2fv1.HTTPRoute)
- [Kubernetes Gateway API - Using HTTPRoute rules to rewrite URI paths](https://serverfault.com/questions/1145923/kubernetes-gateway-api-using-httproute-rules-to-rewrite-uri-paths)
