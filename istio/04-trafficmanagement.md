##
# Traffic Management

The Istio [Bookinfo](https://istio.io/docs/examples/bookinfo/) sample consists of four separate microservices, each with multiple versions. Three different versions of one of the microservices, reviews, have been deployed and are running concurrently. To illustrate the problem this causes, access the Bookinfo app&#39;s /productpage in a browser and refresh several times. You&#39;ll notice that sometimes the book review output contains star ratings and other times it does not. This is because without an explicit default service version to route to, Istio routes requests to all available versions in a round robin fashion.

In this task, you will use Istio to send 100% of the traffic to the v1 version of each of the Bookinfo services. You then set a rule to selectively send traffic to version v2 of the reviews service based on a user identity header added to the request by the productpage service. Also we will see how we can gradualy shift the traffic to one version of the application. 

##
# Route All Traffic to one version of App

The initial goal of this task is to apply rules that route all traffic to v1 (version 1) of the microservices. Later, you will apply a rule to route traffic based on the value of an HTTP request header.

**Apply default destination rules**

Before you can use Istio to control the Bookinfo version routing, you need to define the available versions, called _subsets_, in [destination rules](https://istio.io/docs/concepts/traffic-management/#destination-rules).

Run the following command to create default destination rules for the Bookinfo services:

```

$ kubectl apply -f samples/bookinfo/networking/destination-rule-all-mtls.yaml

```

Wait a few seconds for the destination rules to propagate.

You can display the destination rules with the following command:

```

$ kubectl get destinationrules -o yaml

```

**Apply a virtual service**

To route to one version only, you apply virtual services that set the default version for the microservices. In this case, the virtual services will route all traffic to v1 of each microservice.

Run the following command to apply the virtual services:

```

$ kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml

```

Wait a few seconds for the virtual services to take effect.

You have now configured Istio to route to the v1 version of the Bookinfo microservices, most importantly the reviews service version 1.

**Test the new routing configuration**

You can easily test the new configuration by once again refreshing the /productpage of the Bookinfo app.

Open the Bookinfo site in your browser. The URL is http://$GATEWAY\_URL/productpage, where $GATEWAY\_URL is the External IP address of the ingress, as explained in the [Bookinfo](https://istio.io/docs/examples/bookinfo/#determining-the-ingress-ip-and-port) doc.

Notice that the reviews part of the page displays with no rating stars, no matter how many times you refresh. This is because you configured Istio to route all traffic for the reviews service to the version reviews:v1 and this version of the service does not access the star ratings service.

You have successfully accomplished the first part of this task: route traffic to one version of a service.

##
# Route traffic based on user identity (A/B)

Next, you will change the route configuration so that all traffic from a specific user is routed to a specific service version. In this case, all traffic from a user named Jason will be routed to the service reviews:v2.

Note that Istio doesn't have any special, built-in understanding of user identity. This example is enabled by the fact that the productpage service adds a custom end-user header to all outbound HTTP requests to the reviews service.

Remember, reviews:v2 is the version that includes the star ratings feature.

Run the following command to enable user-based routing:

```

$ kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml

```

Confirm the rule is created:

```

kubectl get virtualservice reviews -o yaml

```

On the /productpage of the Bookinfo app, log in as user jason. Leave the password as blank.

Refresh the browser. What do you see? The star ratings appear next to each review.

Log in as another user (pick any name you wish).

Refresh the browser. Now the stars are gone. This is because traffic is routed to reviews:v1 for all users except Jason.

You have successfully configured Istio to route traffic based on user identity.


##
# Traffic Shifting (Canary)

In this task you will migrate traffic from an old to new version of the reviews service using Istio&#39;s weighted routing feature. Note that this is very different than doing version migration using the deployment features of container orchestration platforms, which use instance scaling to manage the traffic.

With Istio, you can allow the two versions of the reviews service to scale up and down independently, without affecting the traffic distribution between them.

The below steps show you how to gradually migrate traffic from one version of a microservice to another. For example, you might migrate traffic from an older version to a new version.

A common use case is to migrate traffic gradually from one version of a microservice to another. In Istio, you accomplish this goal by configuring a sequence of rules that route a percentage of traffic to one service or another.

**Task1:**

In this task, you will send 50% of traffic to reviews:v1 and 50% to reviews:v3. Then, you will complete the migration by sending 100% of traffic to reviews:v3.

To get started, run this command to route all traffic to the v1 version of each microservice.

```

$ kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml

```

Open the Bookinfo site in your browser. The URL is http://$GATEWAY\_URL/productpage, where $GATEWAY\_URL is the External IP address of the ingress, as explained in the [Bookinfo](https://istio.io/docs/examples/bookinfo/#determining-the-ingress-ip-and-port) doc.

Notice that the reviews part of the page displays with no rating stars, no matter how many times you refresh. This is because you configured Istio to route all traffic for the reviews service to the version reviews:v1 and this version of the service does not access the star ratings service.

Transfer 50% of the traffic from reviews:v1 to reviews:v3 with the following command:

```
$ kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
```

Wait a few seconds for the new rules to propagate.

Confirm the rule was replaced:
In your browser and you now see _red_ colored star ratings approximately 50% of the time. This is because the v3 version of reviews accesses the star ratings service, but the v1 version does not.

**Task2:**

Shifting the traffic ratio to 80:20%

In the shell open the file samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
Now change the ratio to 80:20%

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 20
    - destination:
        host: reviews
        subset: v3
      weight: 80
```

Save the file and reapply the rules:

```
$ kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml

```

In your browser and you now see _red_ colored star ratings approximately 80% of the time. This is because the v3 version of reviews accesses the star ratings service, but the v1 version does not.

**Task3:**

Assuming you decide that the reviews:v3 microservice is stable, you can route 100% of the traffic to reviews:v3 by applying this virtual service:

```

$ kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-v3.yaml

```

Now when you refresh the /productpage you will always see book reviews with _red_ colored star ratings for each review.
