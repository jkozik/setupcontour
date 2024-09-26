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
The HttpRoute for an application 

# References
- [Using Gateway API with Contour](https://projectcontour.io/docs/main/guides/gateway-api/)
