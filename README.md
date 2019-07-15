# Scaling Kubernetes with Istio

_Scale Kubernetes with a Service Mesh​_

## Overview

Welcome to our Microsoft Ready Technical Lab, this workshop will guide you through understanding core tenants of a service mesh with a specific focus given to Istio.

The labs are based upon two different applications with one being a node.js application that allows for voting on the Justice League Superheroes (with more options coming soon). The other being the sample application provided by Istio which can be [Found Here](https://istio.io/docs/examples/bookinfo/). For the first application data is stored in MongoDB.

> Note: These labs are designed to run on a Linux CentOS VM running in Azure (jumpbox) along with Azure Cloud Shell. They can potentially be run locally on a Mac or Windows, but the instructions are not written towards that experience. ie - "You're on your own."s

> Note: Since we are working on a jumpbox, note that Copy and Paste are a bit different when working in the terminal. You can use Shift+Ctrl+C for Copy and Shift+Ctrl+V for Paste when working in the terminal. Outside of the terminal Copy and Paste behaves as expected using Ctrl+C and Ctrl+V. 

## Links

### Lab Pre-Work and Additional Labs
Topics for pre-work include foundational Kubernetes concepts along with foundational OSS tools and syntax references. Labs within pre-work will provide a foundation for Kubernetes with Advanced labs available for further skill-building activities

  1. [Service Mesh Lab Pre-Work](prework/PreWork.md)
   

### Service Mesh Labs

  1. [Lab Access URL (temporary)](istio/01-Lab-Intro.md)
  2. [Initial Configuration](istio/02-InitialConfig.md)
  3. [Sample Application Setup](istio/03-bookinfo_app.md)
  4. [Traffic Routing](istio/04-trafficmanagement.md)
  5. [Authorization with HTTP](istio/05-AuthzHTTP.md)
  6. [Visualize Metrics with Graphana](istio/06-VisualizeMesh.md)
  7. [Visualize Mesh with Kiali](istio/Kiali.md)

  
## Contributing

This project is a fork of a fork of different projects that was then customized to match this technical lab. We welcome contributions and suggestions. Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us the rights to use your contribution. For details, visit https://cla.microsoft.com.

## License

This software is covered under the MIT license. You can read the license [here][license].

This software contains code from Heroku Buildpacks, which are also covered by the MIT license.

This software contains code from [Helm](http://helm.sh), which is covered by the Apache v2.0 license.

You can read third-party software licenses [here][Third-Party Licenses].

