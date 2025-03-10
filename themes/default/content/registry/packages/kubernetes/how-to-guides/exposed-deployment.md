---
title: Deploy NGINX to a Kubernetes Cluster
meta_desc: How to deploy NGINX to a Kubernetes cluster with a publicly exposed IP using
           a Kubernetes Service.
aliases: ["/docs/reference/tutorials/kubernetes/tutorial-exposed-deployment/"]
layout: how-to-guide
---

In this tutorial, we'll deploy `nginx` to a Kubernetes cluster, and publicly expose it to the
Internet with an IP address, using a Kubernetes `Service`. Along the way, we'll get a flavor for the
Pulumi CLI utility, and a general sense of the workflow for writing and managing Kubernetes
applications with Pulumi.

In the gif below we see the experience of deploying this example with `pulumi up`. Notice that
Pulumi has an inherent notion of "done-ness" -- Pulumi waits for the IP address to be allocated to
the `Service`. Because this example uses the Pulumi concept of _stack exports_ to report this IP
address, in this example we are also able to use `curl` to reach the `nginx` server.

![Allocating a public IP to a Deployment](/images/docs/quickstart/kubernetes/exposed-deploy.gif "Allocating a public IP to a Deployment")

## Running the App

Start by downloading the [example code](https://github.com/pulumi/examples/blob/master/kubernetes-ts-exposed-deployment/README.md).

If you haven't already, follow the steps in [Pulumi Installation and
Setup](/docs/install/) and [Configuring Pulumi
Kubernetes](/registry/packages/kubernetes/installation-configuration) to get setup with
Pulumi and Kubernetes.

Now, install dependencies:

```sh
$ npm install
```

Create a new stack:

```sh
$ pulumi stack init
Enter a stack name: exposed-deployment-dev
```

This example will attempt to expose the `nginx` deployment Internet with a `Service` of type
`LoadBalancer`. Since minikube does not support `LoadBalancer`, the application already knows to use
type `ClusterIP` instead; all you need to do is to tell it whether you're deploying to minikube:

```sh
$ pulumi config set isMinikube <value>
```

Perform the deployment:

```sh
$ pulumi up
Updating stack 'exposed-deployment-dev'
Performing changes:

     Type                           Name                                       Status      Info
 +   pulumi:pulumi:Stack            exposed-deployment-exposed-deployment-dev  created     1 warning
 +   ├─ kubernetes:apps:Deployment  nginx                                      created
 +   └─ kubernetes:core:Service     nginx                                      created     2 info messages

Diagnostics:
  kubernetes:core:Service: nginx
    info: ✅ Service 'nginx-rn6uipeg' successfully created endpoint objects

    info: ✅ Service has been allocated an IP

---outputs:---
frontendIp: "35.226.79.225"

info: 3 changes performed:
    + 3 resources created
Update duration: 46.555593397s

Permalink: https://app.pulumi.com/hausdorff/exposed-deployment-dev/updates/1
```

We can see here in the `---outputs:---` section that nginx was allocated a public IP, in this
case `35.226.79.225`. It is exported with a stack output variable, `frontendIp`. We can use `curl`
and `grep` to retrieve the `<title>` of the site the proxy points at.

> _Note_: minikube does not support type `LoadBalancer`; if you are deploying to minikube, make sure
> to run `kubectl port-forward svc/frontend 8080:80` to forward the cluster port to the local
> machine and access the service via `localhost:8080`.

```sh
$ curl -sL $(pulumi stack output frontendIp) | grep "<title>"
<title>Welcome to nginx!</title>
```

## Next steps

Now that `nginx` is deployed and exposed to the internet with an IP, try playing around with the
example!

If we change the `nginx` image to `nginx:1.16-alpine`, we can run `pulumi preview --diff` and see
this change reported to us:

![Diff](/images/docs/quickstart/kubernetes/diff.gif "Reporting a diff after we change the app")

Notice also that if you provide an image that does not exist, Pulumi will report errors as it sees
them. You should see something similar in principle to this:

![Diff](/images/docs/quickstart/kubernetes/error.gif "Error reporting")
