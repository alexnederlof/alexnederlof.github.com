---
layout: single
title: "Local development with Kubernetes on Mac OS"
date: 2018-03-11 13:24:31 +0200
comments: true
categories: 
twitterDescription: "How run develop in your IDE, while communicating with services in a local Kubernetes cluster"
---

Recently I started a new project where we decided to use a microservice achitecture
running on Kubernetes. Lucky for us, Docker for Mac just shipped with Kubernetes
on board, so setting it up was easy. However, figuring out how to route 
traffic between services inside the cluster, and the Java code we're working
on in our IDE wasn't easy. In this blog, I'd like to show you how we set it up.

<!--more-->

On a Mac, docker runs in a Virtual Machine, and in that machine Kubernetes
runs its nodes, which in turn run the pods. Let's say we have two services
`foo` and `bar` which need to talk to each other over HTTP. If we want to be able 
to work on the services from an IDE we have to cover three scenario's:

1. `foo` and `bar` run in the cluster
1. One of them runs in the IDE, the other in the cluster
1. Both run in the IDE.

Let's look at the Kubernetes descriptor for scenario 1. We use a simple 
deployment on a port, and a service of type `LoadBalancer` to make sure 
the port is available from outside the cluster in the browser on our host machine.

{% gist 8ef8477cce49a4b532c79d39eaae77f5 foo.yaml %}

The service `bar` has the [exact same configuration](https://gist.github.com/alexnederlof/8ef8477cce49a4b532c79d39eaae77f5#file-bar-yaml), 
but then on port `8081`. 
We use the an echo service which is a container that just echo's the request
headers back as a JSON object.

I'm assuming you have a Kubernetes running. Let's deploy the two services

```sh
kubectl apply -f foo.yaml
kubectl apply -f bar.yaml
```

It that went well, `kubectl get pods` will show something like:

```
NAME                   READY     STATUS    RESTARTS   AGE
bar-7b9fd9f6b8-tjdtd   1/1       Running   0          2m
foo-6d5457d5b4-njvrq   1/1       Running   0          2m
```

Cool! Now let's see if both work visit [http://localhost:8080](http://localhost:8080) 
and [http://localhost:8081](http://localhost:8081) to see the services running. Since
you opened it in your browser, it means you can also reach it from your IDE.

Next, let's verify the services can reach each other inside kubernetes. We 
can exec into the machine using 

    kubectl exec -t -i bar-7b9fd9f6b8-tjdtd sh
    
We want to use curl, which is not installed yet, so we install it using

    apk update && apk add curl
    
After installing those you should be able to curl the services using 
Kubernete's DNS using 

    curl foo.default.svc.cluster.local:8080
    curl bar.default.svc.cluster.local:8081
    
Keep this window open, as we'll use it in the next step.

That completes scenario one. Now for the harder part. Let's say we
want to work on the `foo` service in our IDE, outside kubernetes. 
We still need `foo` to talk to `bar` which runs inside the cluster,
 and vice versa. But how do we set that up?

It turns out that besides deployments, you can also define manual 
[endpoints](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/). 
The way it works is that Kubernetes creates a service, but sends the
traffic to a custom IP you specify. If we use the LAN IP of our host
machine, services from inside the cluster can route out of the Kubernetes
and reach it. Vice versa, we can still reach the other services via
the LoadBalance end points. Let's see how that works:

First, in a new window delete the foo deployment and service:

```sh
kubectl delete service foo
kubectl delete deployment foo
```

Then, mimic working on Foo in my IDE by simply rinning it with docker
(normally you'd start this application in your IDE or whatever development
environment you use).

    docker run -p 8080:8000 paddycarey/go-echo

Now verify you can reach that service on [localhost:8080](http://localhost:8080).
Now go back to the window that is logged into the `bar` pod, and notice that 
you cannot reach `foo` anymore using curl. That's because we have no service
in the cluster anymore that serves `foo`.

Next, we have to set up the new service and endpoint. For that we need the LAN ip of
our host machine, which on mac you can find using `ifconfig` or if you're
on WiFi you can click the WiFi icon in the menu bar while holding the option key.
Let's say your IP is `192.168.0.2`. We are goign setup a new service, but 
this time, the service will not have the `LoadBalance` type. If we would use that, it 
would conflict with the host machine that is running the docker image. Besides
that, it's completely the same. Instead of adding a deployment, we add an endpoint
that tells the service on which IP and port it can find the service:

{% gist 8ef8477cce49a4b532c79d39eaae77f5 ide-service.yaml %}

After applying these configs, we can see the service is up and running
using `kubectl get services`:

```
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
bar          LoadBalancer   10.105.255.191   localhost     8081:31545/TCP   1h
foo          ClusterIP      10.111.34.149    <none>        8080/TCP         11s
```

Now, switch back to the window that has the `bar` service open. If you retry
`curl foo.default.svc.cluster.local:8080` you'll notice that it is working again! 
Congratulations, you can now send traffic back and forth between servies in
the Kuberenetes cluster, and services outside of it.

## Automating
Naturally, you want to automate this, because your LAN IP changes often, and
you don't want to keep double copies of all your descriptors. In the gist, 
[you'll find a small bash script](https://gist.github.com/alexnederlof/8ef8477cce49a4b532c79d39eaae77f5#file-create_ide_service-sh)
 that grabs your IP address, and create a services
from [a template](https://gist.github.com/alexnederlof/8ef8477cce49a4b532c79d39eaae77f5#file-ide-template-yaml). This makes it easy while developing to switch to running
a service from your IDE, and switch back by applying your original configuration.
You can simply run

    bash create_ide_service.sh foo 8080

Which will do the steps we did by hand before. If I want to switch back
to the in-cluster version, I apply the original `foo.yaml`, and everything
is restored. By adding `*.tmp.ide.yml` to your `.gitignore` these temporary
templates will not be added to git.

 This blog was written using Docker Version 18.03.0-ce-rc1-mac54 (23022),
 and Kubernetes v1.9.2.
