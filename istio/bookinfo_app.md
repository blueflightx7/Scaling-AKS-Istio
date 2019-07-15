**INSTALL BOOKINFO SAMPLE APP**



Change directory to the root of the Istio installation ( /istio ).

The default Istio installation uses automatic sidecar injection. Label the namespace that will host the application with istio-injection=enabled:

**Step1:**  enable istio injection on default namespace

```
kubectl label namespace default istio-injection=enabled
```

**Step2:** Deploy booking app

```
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

The command above launches all four services shown in the bookinfo application architecture including all 3 versions of the reviews service, v1, v2, and v3.

Confirm all services and pods are correctly defined and running:

```
kubectl get pods
```
```
kubectl get svc
```

To confirm that the Bookinfo application is running, send a request to it by a curl command from some pod, for example from ratings:

**Step3:** Make sure app is working fine internally

```
kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath=&#39;{.items[0].metadata.name}&#39;) -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>" 
```
Now that the Bookinfo services are up and running, you need to make the application accessible from outside of your Kubernetes cluster, e.g., from a browser. An Istio Gateway is used for this purpose.

**Step4:** Provision Ingress Gateway for external access

```
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```
Confirm the gateway has been created:
```
kubectl get gateway
```
**Step5:** Run the following commands to obtain the GATEWAY URL/IP address
```
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

echo $ GATEWAY_URL
```
**Step6:** To confirm that the Bookinfo application is accessible from outside the cluster, run the following curl command:

```
curl -s http://${GATEWAY_URL}/productpage | grep -o "<title>.*</title>"
```
You can also point your browser to http://$GATEWAY\_URL/productpage to view the Bookinfo web page. If you refresh the page several times, you should see different versions of reviews shown in productpage, presented in a round robin style (red stars, black stars, no stars), since we haven&#39;t yet used Istio to control the version routing.

**Step7:** In web browser, go to ```http://${GATEWAY\_URL}/productpage```
