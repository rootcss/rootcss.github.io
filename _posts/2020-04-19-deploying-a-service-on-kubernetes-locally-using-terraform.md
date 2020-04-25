---
layout: post
title: Deploying a service on Kubernetes locally using Terraform
tags:
- Kubernetes
- Terraform
- Devops
---

A couple of weeks back, I was trying to debug a Helm chart, but wasn't able to iterate and test the changes as fast as I wanted to. In such situation, I always prefer to test things on my own setup (usually on local machine) - for which I needed Kubernetes locally too. When I started installing minikube using official documentation, it didn't work in my first attempt because of a couple of errors related to Hypervisor and Virtualbox. Also, my debugging required me to use terarform and a custom application, hence this blog post :)

In this post, we'll deploy a small Go service on Kubernetes using Terraform, locally.

Versions:
```config
MacOS: 10.15.4
docker: 19.03.8
kubectl: client v1.15.5, server v1.14.10-gke.17
go: 1.13.9
minikube: v1.9.2
terraform: v0.12.24
```

You can install <a target="_blank" href="https://www.docker.com/products/docker-desktop">docker</a> and <a target="_blank" href="https://kubernetes.io/docs/tasks/tools/install-kubectl/">kubectl</a> first.


## Installing Minikube
Minikube helps us run a single-node Kubernetes cluster locally. You can go through full installation details <a target="_blank" href="https://kubernetes.io/docs/tasks/tools/install-minikube/">here</a>. The prerequisite to run minikube is a hypervisor for your system. Most modern systems have virtualization enabled by default, which you can verify using this command:

```bash
# for macOS
$ sysctl -a | grep -E --color 'machdep.cpu.features|VMX'
```
If you see VMX highlighted in the output, it's enabled for you. Hypervisor is the software that lets you run VM on an existing system. We're going to use <a href="https://github.com/moby/hyperkit" target="_blank">hyperkit</a> - which is lightweight and not from Oracle 😛


```bash
# Install hyperkit
$ brew install hyperkit
# Install minikube
$ brew install minikube
# Start minikube with hyperkit
$ minikube start --vm-driver=hyperkit
# Start minikube dashboard
$ minikube dashboard
```

<p><img class="img-responsive" src="{{ site.url }}/assets/images/k8s-dashboard.png" alt="k8s dashboard" /></p>

Minikube has an isolated docker environment other than the one installed by docker. To switch to that environment, use following command. All the next commands in this shell will be run on minikube docker environment.
```bash
$ eval $(minikube -p minikube docker-env)
```

## Building a small service
We'll now quickly build a small Go application and dockerize it.
```bash
# your directory structure will look like this
$ tree myapp
myapp
├── Dockerfile
├── Makefile
└── myapp.go
```
myapp.go
```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", HelloServer)
	fmt.Println("Starting server on localhost:8080")
	http.ListenAndServe(":8080", nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %s!", r.URL.Path[1:])
}
```

Makefile
```makefile
NAME=myapp
VERSION=0.1
all: build
build: go build -ldflags "-X main.Version=${VERSION}" myapp.go
```

Dockerfile
```dockerfile
FROM golang:1.13-stretch as base
WORKDIR /build/
COPY . .
RUN ["make"]

FROM alpine:latest
WORKDIR /opt/myapp
COPY --from=base /build/myapp /opt/myapp/bin/myapp
RUN ["apk", "update"]
EXPOSE 8080

RUN ["apk", "add", "libc6-compat"]
ENTRYPOINT ["/opt/myapp/bin/myapp"]
```

Build and start your container using following commands (you already know these):
```bash
$ docker build -t myapp:0.1 .
$ docker run -p 8080:8080 myapp:0.1
Starting server on localhost:8080

$ curl localhost:8080
Hello, !
# our app is working fine.
```
Now, we'll finally write our terraform scripts to deploy this service on minikube.


## Writing the Terraform script
```bash
# here are the files that we're going to create
$ tree deploy
deploy
├── main.tf
└── outputs.tf
```
main.tf
```terraform
provider "kubernetes" {
  config_context_cluster   = "minikube"
}

# In kubernetes, you can deploy with different type of objects
# like, Pod, ReplicationController, Deployment etc.
# While Deployment is highly recommended for production use-cases, 
# we'll simply use Pod object
resource "kubernetes_pod" "myapp" {
  metadata {
    name = "myapp"
    labels = {
      App = "myapp"
    }
  }

  spec {
    container {
      image = "myapp:0.1"
      name  = "myapp"

      port {
        container_port = 8080
      }
    }
  }
}

# Let's create a Service object - in simple words, this object helps
# you with loadbalancing and network abstraction on top of pods
resource "kubernetes_service" "myapp" {
  metadata {
    name = "myapp"
  }
  spec {
    selector = {
        App = kubernetes_pod.myapp.metadata[0].labels.App
    }
    port {
      port        = 80
      target_port = 8080
    }

    type = "NodePort" #"LoadBalancer"
  }
}
```

outputs.tf
```terraform
output "name" {
  value = "${kubernetes_pod.myapp.metadata.0.name}"
}
```



## Applying our Infrastructure
```bash
$ cd deploy
$ terraform init
$ terraform apply
```

At this point, the service should be up and running. You can verify the pods and their status. To get the IP of your service, use the following command:
```bash
$ minikube service myapp 
# this will open up your service in the brwoser window
```

<br>
In the next post, I'll try to post about the minor issue I was facing in helm charts, that I was able to reproduce in local and finally fix it :)