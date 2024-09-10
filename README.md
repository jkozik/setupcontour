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

jkozik@knode202:~/contour$ kubectl get svc,pods,ing -n projectcontour
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/contour   ClusterIP      10.97.223.186   <none>        8001/TCP                     2m
service/envoy     LoadBalancer   10.100.173.66   <pending>     80:30825/TCP,443:31400/TCP   2m

NAME                                READY   STATUS      RESTARTS   AGE
pod/contour-66486bf57-ctfs2         1/1     Running     0          2m
pod/contour-66486bf57-d4pzx         1/1     Running     0          2m
pod/contour-certgen-v1-30-0-25jqp   0/1     Completed   0          2m
pod/envoy-62wg4                     2/2     Running     0          2m
pod/envoy-ggl82                     2/2     Running     0          2m
pod/envoy-k8rf8                     2/2     Running     0          2m
jkozik@knode202:~/contour$
```


