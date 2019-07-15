---
title: Visualizing Your Mesh
description: This task shows you how to visualize your services within an Istio mesh.
weight: 49
keywords: [telemetry,visualization]
---

This task shows you how to visualize different aspects of your Istio mesh.

As part of this task, you install the [Kiali](https://www.kiali.io) add-on
and use the web-based graphical user interface to view service graphs of
the mesh and your Istio configuration objects. Lastly, you use the Kiali
Public API to generate graph data in the form of consumable JSON.

This task uses the [Bookinfo](/docs/examples/bookinfo/) sample application as the example throughout.

## Before you begin

The following instructions assume you have installed Helm and use it to install Kiali.
To install Kiali without using Helm, follow the [Kiali installation instructions](https://www.kiali.io/gettingstarted/)

### Create a secret

If you plan on installing Kiali using the `istio-demo.yaml` or `istio-demo-auth.yaml` file as described in the [Istio Quick Start Installation Steps](/docs/setup/kubernetes/install/kubernetes/#installation-steps) then a default secret will be created for you with a username of `admin` and passphrase of `admin`. You can therefore skip this section.

Create a secret in your Istio namespace with the credentials that you use to
authenticate to Kiali.

```bash
$KIALI_USERNAME=[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("kiali"))
$KIALI_PASSPHRASE=[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("REPLACE_WITH_YOUR_SECURE_PASSWORD"))

"apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: istio-system
  labels:
    app: kiali
type: Opaque
data:
  username: $KIALI_USERNAME
  passphrase: $KIALI_PASSPHRASE" | kubectl apply -f -
```

### Install Via Helm

Once you create the Kiali secret, follow
[the Helm install instructions](/docs/setup/kubernetes/install/helm/) to install Kiali via Helm.
You must use the `--set kiali.enabled=true` option when you run the `helm` command, for example:

```bash
$ helm template --set kiali.enabled=true install/kubernetes/helm/istio --name istio --namespace istio-system > $HOME/istio.yaml
$ kubectl apply -f $HOME/istio.yaml
```

This task does not discuss Jaeger and Grafana. If
you already installed them in your cluster and you want to see how Kiali
integrates with them, you must pass additional arguments to the
`helm` command, for example:

```bash
$ helm template \
    --set kiali.enabled=true \
    --set "kiali.dashboard.jaegerURL=http://jaeger-query:16686" \
    --set "kiali.dashboard.grafanaURL=http://grafana:3000" \
    install/kubernetes/helm/istio \
    --name istio --namespace istio-system > $HOME/istio.yaml
$ kubectl apply -f $HOME/istio.yaml
```

Once you install Istio and Kiali, deploy the [Bookinfo](/docs/examples/bookinfo/) sample application.

### Running on OpenShift

When Kiali runs on OpenShift it needs access to some OpenShift specific resources in order to function properly,
which can be done using the following commands after Kiali has been installed:

```bash
$ oc patch clusterrole kiali -p '[{"op":"add", "path":"/rules/-", "value":{"apiGroups":["apps.openshift.io"], "resources":["deploymentconfigs"],"verbs": ["get", "list", "watch"]}}]' --type json
$ oc patch clusterrole kiali -p '[{"op":"add", "path":"/rules/-", "value":{"apiGroups":["project.openshift.io"], "resources":["projects"],"verbs": ["get"]}}]' --type json
$ oc patch clusterrole kiali -p '[{"op":"add", "path":"/rules/-", "value":{"apiGroups":["route.openshift.io"], "resources":["routes"],"verbs": ["get"]}}]' --type json
```

## Generating a service graph

1.  To verify the service is running in your cluster, run the following command:

    ```bash
    $ kubectl -n istio-system get svc kiali
    ```

2.  To determine the Bookinfo URL, follow the instructions to determine the [Bookinfo ingress `GATEWAY_URL`](/docs/examples/bookinfo/#determining-the-ingress-ip-and-port).

3.  To send traffic to the mesh, you have three options

    *   Visit `http://$GATEWAY_URL/productpage` in your web browser

    *   Use the following command multiple times:

        ```bash
        $ curl http://$GATEWAY_URL/productpage
        ```

    *   If you installed the `watch` command in your system, send requests continually with:

        ```bash
        $ watch -n 1 curl -o /dev/null -s -w %{http_code} $GATEWAY_URL/productpage
        ```

4.  To open the Kiali UI, execute the following command in your Kubernetes environment:

    ```bash
    $ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001
    ```

5.  Visit <http://localhost:20001/kiali/console> in your web browser.

6.  To log into the Kiali UI, go to the Kiali login screen and enter the username and passphrase stored in the Kiali secret.

7.  View the overview of your mesh in the **Overview** page that appears immediately after you log in.
    The **Overview** page displays all the namespaces that have services in your mesh.
    The following screenshot shows a similar page:

    ![](https://istio.io/docs/tasks/telemetry/kiali/kiali-overview.png)

8.  To view a namespace graph, click on the `bookinfo` graph icon in the Bookinfo namespace card. The graph icon is in the lower left of
    the namespace card and looks like a connected group of circles.
    The page looks similar to:

    ![](https://istio.io/docs/tasks/telemetry/kiali/kiali-graph.png)

9.  To view a summary of metrics, select any node or edge in the graph to display
    its metric details in the summary details panel on the right.

10. To view your service mesh using different graph types, select a graph type
    from the **Graph Type** drop down menu. There are several graph types
    to choose from: **App**, **Versioned App**, **Workload**, **Service**

    - The **App** graph type aggregates all versions of an app into a single graph node.
      The following example shows a single **reviews** node representing the three versions
      of the reviews app.

        ![](https://istio.io/docs/tasks/telemetry/kiali/kiali-app.png)

    - The **Versioned App** graph type shows a node for each version of an app,
      but all versions of a particular app are grouped together. The following example
      shows the **reviews** group box that contains the three nodes that represents the
      three versions of the reviews app.

        ![](https://istio.io/docs/tasks/telemetry/kiali/kiali-versionedapp.png)
    - The Service graph type shows a node for each service in your mesh but excludes all apps and workloads from the graph.
    
        ![](https://istio.io/docs/tasks/telemetry/kiali/kiali-service-graph.png)

                        Example Service Graph
11. To examine the details about the Istio configuration, click on the Applications, Workloads, and Services menu icons on the left menu bar. The following screenshot shows the Bookinfo applications information:


       ![](https://istio.io/docs/tasks/telemetry/kiali/kiali-services.png)
        
       
                              Example Details





## About the Kiali Public API

To generate JSON files representing the graphs and other metrics, health, and configuration information, you can access the Kiali Public API. For example, point your browser to $KIALI_URL/api/namespaces/graph?namespaces=bookinfo&graphType=app to get the JSON representation of your graph using the app graph type.
The Kiali Public API is built on top of Prometheus queries and depends on the standard Istio metric configuration. It also makes Kubernetes API calls to obtain additional details about your services. For the best experience using Kiali, use the metadata labels app and version on your application components. As a template, the Bookinfo sample application follows this convention.

## Cleanup

If you are not planning any follow-up tasks, remove the Bookinfo sample application and Kiali from your cluster.
1. To remove the Bookinfo application, refer to the Bookinfo cleanup instructions.
2. To remove Kiali from a Kubernetes environment, remove all components with the app=kiali label:
```
kubectl delete all,secrets,sa,configmaps,deployments,ingresses,clusterroles,clusterrolebindings,virtualservices,destinationrules,customresourcedefinitions --selector=app=kiali -n istio-system
```
        
