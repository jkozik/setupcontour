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

# References
- [Using Gateway API with Contour](https://projectcontour.io/docs/main/guides/gateway-api/)
