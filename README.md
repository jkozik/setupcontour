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
Let's create a dummy application and verify that basic Ingress is working.  Borrowing from the Contour documentation, let's setup Kuard.

