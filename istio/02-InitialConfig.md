
<h2>Jumpbox/Kubectl Configuration</h2>
 
**Step1:** Login to the jumpbox provide as part of your lab and run execute the following commands. Note that the lab jumpbox is configured with all necessary utilities and configurations to complete the lab. 
You can use PuTTY or PowerShell to access the jumbox

```
ssh <jumpbox_fqdn>
```
Once you are logged in, switch to the root user context by 

```
sudo su -
```

**Step2:** Access your Azure subscription using the username and password from the lab environment.

```
az login -u <USERNAME>
```

**Step3:** List the AKS clusters in the subscription

```
az aks list -o table
```

Sample output:
```
[root@jumpbox ~]# az aks list -o table
Name      Location    ResourceGroup    KubernetesVersion    ProvisioningState                                                                                                        
--------  ----------  ---------------  -------------------  -------------------                                                                                                      
acs73294  westus      labrg-73294      1.12.8               Succeeded                                                                                                               
```

**Step4:** Connect to the AKS cluster
```
az aks get-credentials -n <cluster_name> -g <resourcegroupname>
```
Eg: az aks get-credentials -n acs73293 -g labrg-73293
 
**Step5:** Verify the cluster access
```
kubectl cluster-info
kubectl get nodes
```
<h3>HELM Configuration</h3>
Configure the Cluster Role and RoleBinding to provide enough RBAC access in the cluster to perform the HELM tasks. 


**Step1:** Initiate HELM by running the flowing command
```
helm init --force-upgrade
```

Verify that HELM is running by the following command:
```
helm version
```

**NOTE:**Wait for a few seconds and retry if you get an error message: “helm could not find a ready tiller pod”.

Run the following command to list the HELM packages available. 

```
helm ls
```

**WARNING** In some cases it's necessary to add the directory manually to your $PATH variable.

```output
helm not found. Is /usr/local/bin in your $PATH?
Failed to install helm
```

To fix this issue you have to add /usr/local/bin to your $PATH variable

```bash
PATH=$PATH:/usr/local/bin
```

To make this permanently and not only for the current session you can add it to your .bash_profile which is executed everytime a bash is started.

```bash
echo 'export PATH=/usr/local/bin:$PATH' >>~/.bash_profile
```


You will see an error message hinting that you don’t have the enough access rights in the cluster/namespaces. 

**Step2:** To fix the permission issues, follow the below steps to create a service account for tiller and get the proper RBAC privileges applied for the service account.
```
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
```


**Step3:** Now update the tiller deployment to use this service account.
```
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

 
**Step4:** The tiller pods will be deployed in few seconds and verify the HELM functionality again.
```
helm version
helm ls   (output will be blank and it shouldn’t return any errors.)
```

 

<h3>Istio Download and install</h3>

**Step1:** Run the following commands to download the ISTIO and place its components in the /istio directory in your jumpbox.
```
ISTIO_VERSION=1.1.11
curl -sL "https://github.com/istio/istio/releases/download/$ISTIO_VERSION/istio-$ISTIO_VERSION-linux.tar.gz" | tar xz
cp -r istio-1.1.11/ /istio
cd /istio
chmod 755 ./bin/istioctl
sudo cp -p ./bin/istioctl /usr/local/bin/istioctl
```
 
**Step2:** Install Istio CRD

Istio uses Custom Resource Definitions to work inside the Kubernetes cluster. Create the CRDS with the following command:
```
helm install install/kubernetes/helm/istio-init --name istio-init --namespace istio-system
```
 
Verify and wait for all 53 CRDs to populate
 ```
kubectl get crds | grep 'istio.io' | wc -l
```


<h3>Grafana Installation</h3>

We'll be installing Grafana also as part of our Istio installation. Grafana provides analytics and monitoring dashboards, 
Before we can install the Istio components, we must create the secrets for Grafana. Create these secrets by running the following commands:

```
GRAFANA_USERNAME=$(echo -n "grafana" | base64)
GRAFANA_PASSPHRASE=$(echo -n "Graf@naP@ss01" | base64)

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: grafana
  namespace: istio-system
  labels:
    app: grafana
type: Opaque
data:
  username: $GRAFANA_USERNAME
  passphrase: $GRAFANA_PASSPHRASE
EOF
```
 
<h3>Istio Installation</h3>

**Step1:** Ensure that all the 53 CRDs got created for Istio and run the following commands to install Istio Service Mesh.
```
helm install install/kubernetes/helm/istio --name istio --namespace istio-system \
  --set global.controlPlaneSecurityEnabled=true \
  --set mixer.adapters.useAdapterCRDs=false \
  --set grafana.enabled=true --set grafana.security.enabled=true \
  --set tracing.enabled=true
```

 
**Step2:** Validate Istio and wait for all objects to be provisioned.
```
kubectl get svc --namespace istio-system --output wide
```


**Sample Output:**
```
[root@jumpbox istio]# kubectl get svc --namespace istio-system --output wide

```
```
NAME                     TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                                                                                                                                      AGE   SELECTOR
grafana                  ClusterIP      10.0.3.40      <none>           3000/TCP                                                                                                                                     13m   app=grafana
istio-citadel            ClusterIP      10.0.15.126    <none>           8060/TCP,15014/TCP                                                                                                                           13m   istio=citadel
istio-galley             ClusterIP      10.0.122.142   <none>           443/TCP,15014/TCP,9901/TCP                                                                                                                   13m   istio=galley
istio-ingressgateway     LoadBalancer   10.0.76.80     40.118.131.149   15020:30643/TCP,80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:32603/TCP,15030:30023/TCP,15031:32366/TCP,15032:30231/TCP,15443:30332/TCP   13m   app=istio-ingressgateway,istio=ingressgateway,release=istio
istio-pilot              ClusterIP      10.0.137.49    <none>           15010/TCP,15011/TCP,8080/TCP,15014/TCP                                                                                                       13m   istio=pilot
istio-policy             ClusterIP      10.0.63.1      <none>           9091/TCP,15004/TCP,15014/TCP                                                                                                                 13m   istio-mixer-type=policy,istio=mixer
istio-sidecar-injector   ClusterIP      10.0.49.198    <none>           443/TCP                                                                                                                                      13m   istio=sidecar-injector
istio-telemetry          ClusterIP      10.0.47.232    <none>           9091/TCP,15004/TCP,15014/TCP,42422/TCP                                                                                                       13m   istio-mixer-type=telemetry,istio=mixer
jaeger-agent             ClusterIP      None           <none>           5775/UDP,6831/UDP,6832/UDP                                                                                                                   13m   app=jaeger
jaeger-collector         ClusterIP      10.0.33.125    <none>           14267/TCP,14268/TCP                                                                                                                          13m   app=jaeger
jaeger-query             ClusterIP      10.0.255.218   <none>           16686/TCP                                                                                                                                    13m   app=jaeger
prometheus               ClusterIP      10.0.126.124   <none>           9090/TCP                                                                                                                                     13m   app=prometheus
tracing                  ClusterIP      10.0.151.57    <none>           80/TCP                                                                                                                                       13m   app=jaeger
zipkin                   ClusterIP      10.0.129.239   <none>           9411/TCP                                                                                                                                     13m   app=jaeger
```
```
kubectl get pods --namespace istio-system
```

**Sample output:**
```
[root@jumpbox istio]# kubectl get pods --namespace istio-system
NAME                                      READY   STATUS      RESTARTS   AGE
grafana-c9779997c-t2ntb                   1/1     Running     0          3m4s
istio-citadel-6665bfcc5-gtnqr             1/1     Running     0          3m4s
istio-galley-96655cb6c-vk787              1/1     Running     0          3m4s
istio-ingressgateway-5f8c7d456b-2xph7     1/1     Running     0          3m4s
istio-init-crd-10-jq4vt                   0/1     Completed   0          3m50s
istio-init-crd-11-w2nkd                   0/1     Completed   0          3m50s
istio-pilot-68f5b484f4-z7g54              2/2     Running     0          3m4s
istio-policy-57dd4b8b7f-nxm22             2/2     Running     1          3m4s
istio-sidecar-injector-5f58db6564-5sq4s   1/1     Running     0          3m4s
istio-telemetry-66c8f78cd9-p6j7k          2/2     Running     2          3m4s
istio-tracing-79db5954f-qm49f             1/1     Running     0          3m3s
prometheus-5977597c75-mfrqw               1/1     Running     0          3m4s
```
**NOTE: It will take a couple of minutes for all the PODs/Services to be in running state.**

**NOTE:** Ensure that you are always at the /istio directory in your jumpbox shell to execute the commands in the following exercises.
