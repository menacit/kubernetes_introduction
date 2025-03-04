---
SPDX-FileCopyrightText: © 2025 Menacit AB <foss@menacit.se>
SPDX-License-Identifier: CC-BY-SA-4.0

title: "Kubernetes introduction"
author: "Joel Rangsmo <joel@menacit.se>"
footer: "© Course authors (CC BY-SA 4.0)"
description: "Introduction to Kubernetes"
keywords:
  - "kubernetes"
color: "#ffffff"
class:
  - "invert"
style: |
  section.center {
    text-align: center;
  }

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Adam Lusch (CC BY-SA 2.0)" -->
# Kubernetes
## A somewhat gentle introduction

![bg right:30%](images/cube_houses.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Rod Waddington (CC BY-SA 2.0)" -->
What is Kubernetes?  
  
Why would(n't) I use it?  
  
How does it compare to IaaS,
like AWS EC2 or OpenStack?
  
Can I try it myself?

![bg right:30%](images/green_cables.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% C. Watts (CC BY 2.0)" -->
## Let's set the stage
In the beginning, there were
developers and system operators.  

Constant feuds and shared responsibilities
made IT fairly inefficient/unreliable.  

The "DevOps" term was coined to merge
the divide and take responsibility for
the whole application life-cycle.

![bg right:30%](images/old_computers.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% C. Watts (CC BY 2.0)" -->
## ...and the problem is?
Not all system operators are
great developers.  
  
Not all developers are
great system operators.  

Should they be?

![bg right:30%](images/old_computers.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Rod Waddington (CC BY-SA 2.0)" -->
Perhaps we should have
**AppOps** and **InfraOps** instead?

![bg right:30%](images/window_cleaners.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Bruno Cordioli (CC BY 2.0)" -->
## Introducing Kubernetes
Open-source project originally
developed by Google, released in 2014.  

Inspired by their internal
cluster management software "Borg".

Like "Docker Compose" on steroids,
but for the whole datacenter(s).  

Used by ~~everyone~~ most large
organizations to aid their IT operations.

![bg right:30%](images/capsule_house.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Pedro Ribeiro Simões (CC BY 2.0)" -->
## What makes Kubernetes great?
Separates operation of applications/services
and the underlying cluster infrastructure.  

Provides declarative configuration that can
be managed in a VCS, like Git, and deployed
using CI/CD.
  
Smart scheduling/failure recovery.
  
Available as managed solutions on all\*
public clouds and self-hostable -
same interface regardless!

![bg right:30%](images/machine_cables.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Takomabibelot (CC BY 2.0)" -->
## Big three offerings
**G**oogle **K**ubernetes **E**ngine.  
  
**A**zure **K**ubernetes **S**ervice.  

Amazon **E**lastic **K**ubernetes **S**ervice.

![bg right:30%](images/number_three.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% NASA/Bill Stafford (CC BY 2.0)" -->
## What's a Kubernetes cluster?
One or more Linux servers acting as
the **"control-plane"**.  

One or more Linux/Windows servers
acting as **"worker nodes"** that
run **"pods"** (\~containers).

Everything talks to the
**Kubernetes API server**.

![bg right:30%](images/space_station.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Charles Hoisington, GSFC (CC BY 2.0)" -->
## Accessing Kubernetes API
- Web dashboard (_meh!_)
- HTTP client _(like curl)_ / SDKs 
- [**kubectl**](https://kubernetes.io/docs/reference/kubectl/)

![bg right:30%](images/satellite_dish.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Ron Cogswell (CC BY 2.0)" -->
Nuff' talk,
let's do some demos!  

![bg right:30%](images/contrails.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Jonathan Brandt (CC0 1.0)" -->
## Our demo application
Simple Python application exposing
recipes for cocktails from the
**I**nternational **B**artenders **A**ssociation's
official list over HTTP.  

Enables us to filter cocktails
based their name.  

Enables us to exclude cocktails
containing specific ingredients.

![bg right:30%](images/tumbler.jpg)

---
### recipes\_server.py
```python
'''
> recipes - Service for fetching and exposing IBA cocktail recipes.

Example usage:

GET /api/list : Return list of cocktail recipes.
GET /api/list?filter=God : Return list of recipes with name containg "God".
GET / : Health/Readiness end-point.

Listens for HTTP on port 1338/TCP by default.
Settings configurable using environment variables:

"APP_EXCLUDED_INGREDIENTS":
Exclude cocktail recipes containing specific ingredient, separated by comma.
Default:
"Galliano,DiSaronno"
'''

[...]
```

---
### Building a container image
```dockerfile
FROM docker.io/library/python:3.13.1-alpine

RUN apk --no-cache add figlet

WORKDIR /usr/src/app
RUN pip install --no-cache-dir Flask==3.1.0 requests==2.32.3
COPY recipes_server.py .

USER 10000
EXPOSE 1338
CMD ["python", "recipes_server.py"]
````

```
$ docker build -t recipes_server:v1 .

[+] Building 1.7s (10/10) FINISHED
```

---
### Running it locally
```
$ docker run --publish 80:1338 recipes_server:v1

INFO: Downloaded recipies for 77 cocktails
INFO: Filtering recipes for excluded ingredients
INFO: Excluding recipe "Yellow Bird" as it contains ingredient "Galliano"
INFO: Excluding recipe "Barracuda" as it contains ingredient "Galliano"
INFO: Excluding recipe "French Connection" as it contains ingredient "DiSaronno"
[...]
INFO: Generating ASCII art for recipe names using Figlet
INFO: Starting recipes web server on host 648dc9db8306 (app v1)
```

---
### Trying it out locally
```
$ curl http://127.0.0.1/

Hello from recipes API server on host 648dc9db8306 (app v1)!
```

```
$ curl http://127.0.0.1/api/list?filter=Mai

[
  {
    "name": "Mai-tai",
    "category": "Longdrink",
    "glass": "highball",
    "ingredients": [
      {
        "amount": 4,
        "ingredient": "White rum",
        "unit": "cl"
[...]
```

---
<!-- _footer: "%ATTRIBUTION_PREFIX% William Warby (CC BY 2.0)" -->
Let's give it a spin on a Kubernetes cluster!

```
$ kubectl get nodes

NAME       STATUS   ROLES           AGE   VERSION
master-1   Ready    control-plane   29h   v1.31.5
master-2   Ready    control-plane   29h   v1.31.5
master-3   Ready    control-plane   29h   v1.31.5
worker-1   Ready    <none>          29h   v1.31.5
worker-2   Ready    <none>          29h   v1.31.5
worker-3   Ready    <none>          29h   v1.31.5
```

![bg right:30%](images/monkey.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Milan Bhatt (CC BY-SA 2.0)" -->
## What are pods?
Kubernetes schedules workloads
in a unit they call **"pod"**.  

To keep things simple for now,
imagine that a pod is the same
thing as an application container.

To get our application running,
we'll define a pod object!

![bg right:30%](images/whale.jpg)

---
### recipes\_pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  namespace: cocktails
  name: recipes
  labels:
    application: recipes
spec:
  containers:
    - name: application-container
      image: ghcr.io/menacit/demo_apps/recipes:v1
      ports:
        - name: http-api
          containerPort: 1338
[...]
```

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Milan Bhatt (CC BY-SA 2.0)" -->
### Submitting pod definition 
```
$ kubectl apply -f pod_recipes.yaml

pod/recipes created
```

```
$ kubectl --namespace cocktails get pods

NAME      READY   STATUS    RESTARTS   AGE
recipes   1/1     Running   0          18s
```

![bg right:30%](images/whale.jpg)

---
### Inspecting pod object
```
$ kubectl --namespace cocktails describe pod recipes 

Name:    recipes
Status:  Running
IP:      10.0.0.171
Node:    worker-3/10.13.38.23

[...]

Events:

Reason     Age   From       Message
------     ----  ----       -------
Scheduled  42s   scheduler  Successfully assigned cocktails/recipes to worker-3
Pulling    41s   kubelet    Pulling image "ghcr.io/menacit/demo_apps/recipes:v1"
Pulled     39s   kubelet    Successfully pulled image
                            "ghcr.io/menacit/demo_apps/recipes:v1" in 2s

Created    38s   kubelet    Created container application-container
Started    38s   kubelet    Started container application-container
```

---
### Inspecting pod object
```
$ kubectl --namespace cocktails logs pods/recipes 

INFO: Downloaded recipies for 77 cocktails
INFO: Filtering recipes for excluded ingredients
INFO: Excluding recipe "God Father" as it contains ingredient "DiSaronno"
INFO: Excluding recipe "Harvey Wallbanger" as it contains ingredient "Galliano"
[...]
INFO: Generating ASCII art for recipe names using Figlet
INFO: Starting recipes web server on pod recipes on node worker-3 (app v1)
```

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Gytis B (CC BY-SA 2.0)" -->
How do we access a network service
provided by a Kubernetes pod?  

Several options available,
let's begin with `kubectl port-forward`.

![bg right:30%](images/vechicle_graveyard.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Gytis B (CC BY-SA 2.0)" -->
```
$ kubectl \
  --namespace cocktails port-forward \
  pods/recipes :http-api &
  
Forwarding from 127.0.0.1:33345 -> 1338
[1] 1098787
```

```
$ curl http://127.0.0.1:33345/

Hello from recipes API server on
pod recipes on node worker-3 (app v1)!
```

![bg right:30%](images/vechicle_graveyard.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Freed eXplorer (CC BY 2.0)" -->
## Services
Another method for accessing
network services provided by pods
is to define **"service"** objects.

Acts as a layer four load balancer.

Forwards traffic to a sub-set of pods
based on their assigned labels.  

_("application=recipes", for example)_

![bg right:30%](images/tunnel.jpg)

---
### "ClusterIP" type
Provides a virtual IP address accessible to all pods running
in the cluster. Also configures DNS entries for `$service_name`,
`$service_name.$namespace_name` and similar for easy access.

### "NodePort" type
Extends "ClusterIP" by setting up a listener port on all (worker) nodes,
enabling external access to services.

### "LoadBalancer" type
Extends "NodePort" by interacting with the underlying cloud/infrastructure
provider to configure an external load balancer that forwards traffic to
the appropriate listener port on worker nodes.

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Freed eXplorer (CC BY 2.0)" -->
### service\_recipes.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: cocktails
  name: recipes
spec:
  type: NodePort
  selector:
    application: recipes
  ports:
    - port: 80
      nodePort: 31337
      targetPort: http-api
```

![bg right:30%](images/tunnel.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Freed eXplorer (CC BY 2.0)" -->
```
$ kubectl apply -f service_recipes.yaml

service/recipes created
```

```
$ kubectl \
  --namespace cocktails \
  describe service recipes

Name:         recipes
Selector:     application=recipes
Type:         NodePort
IP:           10.111.192.104
Port:         <unset>  80/TCP
NodePort:     <unset>  31337/TCP
TargetPort:   http-api/TCP
Endpoints:    10.0.0.171:1338
[...]
```

![bg right:30%](images/tunnel.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Freed eXplorer (CC BY 2.0)" -->
### Testing external access
```
$ curl http://${IP_OF_ANY_WORKER}:31337/

Hello from recipes API server on
pod recipes on node worker-3 (app v1)!
```

![bg right:30%](images/tunnel.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Håkan Dahlström (CC BY 2.0)" -->
We've basically recreated our
simple and local `docker run`,
albeit with a whole bunch of YAML.  

Manually defining pods is however
discouraged - they are not automatically
restarted/rescheduled if they
become unavailable.

A common solution is to defined
**"deployments"** instead.

![bg right:30%](images/containers.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% The Preiser Project (CC BY 2.0)" -->
## Cleaning up
```
$ kubectl \
  --namespace cocktails \
  delete pod recipes
```

_or_

```
$ kubectl delete -f pod_recipes.yaml
```

![bg right:30%](images/cleaning_hdd.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Håkan Dahlström (CC BY 2.0)" -->
## What are deployments?
Spawns one or more pods based
on a shared pod definition template.  

Monitors the availability of
pod replicas and creates new ones
if required.  

Performs "controlled rollouts" if
pod definition changes, due to a
new container image version tag
or similar modification.

![bg right:30%](images/containers.jpg)

---
### deployment\_recipes.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: cocktails
  name: recipes
spec:
  replicas: 2
  selector:
    matchLabels:
      application: recipes
  template:
    metadata:
      labels:
        application: recipes
    spec:
      containers:
        - name: application-container
          image: ghcr.io/menacit/demo_apps/recipes:v1
          ports:
            - name: http-api
              containerPort: 1338
[...]
```

---
### Deploying that deployment
```
$ kubectl apply -f deployment_recipes.yaml

deployment.apps/recipes created
```

```
$ kubectl --namespace cocktails get pods

NAME                       READY   STATUS    RESTARTS   AGE
recipes-75778547fd-9tgzv   1/1     Running   0          14s
recipes-75778547fd-m8spr   1/1     Running   0          14s
```

---
### Observing automatic healing
```
$ kubectl \
  --namespace cocktails \
  delete pod recipes-75778547fd-m8spr

pod "recipes-75778547fd-m8spr" deleted
```

```
$ kubectl --namespace cocktails get pods

NAME                       READY   STATUS    RESTARTS   AGE
recipes-75778547fd-6nhmh   1/1     Running   0          11s
recipes-75778547fd-9tgzv   1/1     Running   0          72s
```


---
<!-- _footer: "%ATTRIBUTION_PREFIX% Håkan Dahlström (CC BY 2.0)" -->
Let's modify that deployment and
watch what happens!

![bg right:30%](images/containers.jpg)

---
### deployment\_recipes.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: cocktails
  name: recipes
spec:
  replicas: 3
  selector:
    matchLabels:
      application: recipes
  template:
    metadata:
      labels:
        application: recipes
    spec:
      containers:
        - name: application-container
          image: ghcr.io/menacit/demo_apps/recipes:v2
          ports:
            - name: http-api
              containerPort: 1338
[...]
```

---
### Applying our changes
```
$ kubectl apply -f deployment_recipes.yaml

deployment.apps/recipes configured
```

```
$ kubectl --namespace cocktails get pods

NAME                       READY   STATUS              AGE
recipes-69669fc5d5-wlzd2   1/1     ContainerCreating   0s
recipes-75778547fd-6nhmh   1/1     Running             87s
recipes-75778547fd-9tgzv   1/1     Running             1m59s
```

---
### Observing deployment rollout

```
$ kubectl --namespace cocktails get pods

NAME                       READY   STATUS              AGE
recipes-69669fc5d5-vptxj   0/1     ContainerCreating   0s
recipes-69669fc5d5-wlzd2   1/1     Running             2s
recipes-75778547fd-6nhmh   1/1     Running             89s
recipes-75778547fd-9tgzv   1/1     Running             2m1s
```

```
$ kubectl --namespace cocktails get pods

NAME                       READY   STATUS              AGE
recipes-69669fc5d5-nz6tc   1/1     Running             8s
recipes-69669fc5d5-vptxj   1/1     Running             9s
recipes-69669fc5d5-wlzd2   1/1     Running             11s
recipes-75778547fd-6nhmh   1/1     Terminating         98s
recipes-75778547fd-9tgzv   1/1     Terminating         2m10s
```

---
### ...and we're finished!
```
$ kubectl --namespace cocktails get pods

NAME                       READY   STATUS              AGE
recipes-69669fc5d5-nz6tc   1/1     Running             23s
recipes-69669fc5d5-vptxj   1/1     Running             24s
recipes-69669fc5d5-wlzd2   1/1     Running             26s
```

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Steve Jurvetson (CC BY 2.0)" -->
That's neat, but perhaps
not too impressive.

Let's extend our demo application
to show-case some more neat
features of Kubernetes!

![bg right:30%](images/pyramid.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Jan Hrdina (CC BY-SA 2.0)" -->
## "analytics" application
Simple Python application that connects
to the "recipes" web server and
provides _analytical insights_.  

Enables us to answer questions like:

> What are the most common
> cocktail ingredients? 

Exposes results over an HTTP API.

![bg right:30%](images/optics.jpg)

---
### analytics\_server.py
```
'''
> analytics - Provides analytical insights into cocktail recipes.

Example usage:

GET /api/top/5 : Return a list of the five most common cocktail ingredients.
GET / : Health/Readiness end-point.

Listens for HTTP on port 1338/TCP by default.
Settings configurable using environment variables:

"APP_RECIPES_URL":
Base HTTP(S) URL to "recipes" API without path.
'''

[...]
```

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Jan Hrdina (CC BY-SA 2.0)" -->
### deployment\_analytics.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: cocktails
  name: analytics
spec:
  replicas: 3
  selector:
    matchLabels:
      application: analytics
  template:
    metadata:
      labels:
        application: analytics
    spec:
      containers:
        - name: application-container
          image: ghcr.io/menacit/demo_apps/analytics:v1
          ports:
            - name: http-api
              containerPort: 1338
          livenessProbe:
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 5
            httpGet:
              path: /
              port: http-api
          readinessProbe:
            periodSeconds: 3
            timeoutSeconds: 2
            failureThreshold: 3
            httpGet:
              path: /
              port: http-api
          env:
            - name: APP_RECIPES_URL
              value: http://recipes
[...]
```

![bg right:30%](images/optics.jpg)


---
<!-- _footer: "%ATTRIBUTION_PREFIX% Jan Hrdina (CC BY-SA 2.0)" -->
### deployment\_analytics.yaml (probes)
```yaml
[...]
    ports:
      - name: http-api
        containerPort: 1338

    livenessProbe:
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 5
      httpGet:
        path: /
        port: http-api

    readinessProbe:
      periodSeconds: 3
      timeoutSeconds: 2
      failureThreshold: 3
      httpGet:
        path: /
        port: http-api
[...]
```

![bg right:30%](images/optics.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Jan Hrdina (CC BY-SA 2.0)" -->
### deployment\_analytics.yaml (env)
```yaml
[...]
    env:
      - name: APP_RECIPES_URL
        value: http://recipes
[...]
```

_(could also be set to include
namespace name, like "http://recipes.cocktails"
or "http://recipes.cocktails.svc.cluster.local")_

![bg right:30%](images/optics.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Jan Hrdina (CC BY-SA 2.0)" -->
### service\_analytics.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: cocktails
  name: analytics
spec:
  selector:
    application: analytics
  ports:
    - port: 80
      targetPort: http-api
```

![bg right:30%](images/optics.jpg)

---
### Submitting deployment and service
```
$ kubectl apply -f deployment_analytics.yaml 

deployment.apps/analytics created
```

```
$ kubectl apply -f service_analytics.yaml 

service/analytics created
```

```
$ kubectl \
  --namespace cocktails \
  get pods --selector application=analytics

NAME                         READY   STATUS    RESTARTS   AGE
analytics-6fcc8dc8d8-d8bm2   0/1     Running   0          1s
analytics-6fcc8dc8d8-g2cks   1/1     Running   0          2s
analytics-6fcc8dc8d8-gqlf4   1/1     Running   0          2s
```

---
### Testing network access
```
$ kubectl \
  --namespace cocktails \
  port-forward services/analytics :80 &

Forwarding from 127.0.0.1:33691 -> 1338
[1] 1113393
```

```
$ curl http://127.0.0.1:33691/

Hello from analytics API server on
pod analytics-6fcc8dc8d8-kqz7p on node worker-1!
```

```
$ curl http://127.0.0.1:33691/api/top/5

["Gin","Lemon juice","Syrup","Vodka","Lime juice"]
```

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Jan Hrdina (CC BY-SA 2.0)" -->
What just happened?  

We connected to the "analytics" service
and traffic was forwarded to one of the
appropriate pods _(application=analytics)_.

An instance of the analytics application
connected to the "recipes" service and
traffic was forwarded to one of the
appropriate pods _(application=recipes)_.  

An instance of the recipes application
returned a list of cocktails which the
analytics application extracted some
statistics from.

![bg right:30%](images/optics.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Marcin Wichary (CC BY 2.0)" -->
Not everyone is a huge fan
of the text experience.  

Let's make it sparkle by adding a
web UI that ties the services together!

![bg right:30%](images/text_interface.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Guilhem Vellut (CC BY 2.0)" -->
## "frontend" application
Collects information from the
recipes and analytics applications.  

Presents data as an HTML web page!

![bg right:30%](images/greenhouse_dome.jpg)

---
```
$ kubectl apply -f deployment_frontend.yaml

deployment.apps/frontend created
```

```
$ kubectl apply -f service_frontend.yaml 
service/frontend created
```

```
$ kubectl \
  --namespace cocktails \
  port-forward services/frontend :80 &

Forwarding from 127.0.0.1:35167 -> 8000
[1] 1117417
```

![bg right:33%](images/frontend_port_forward-1.png)

---
```
$ kubectl apply -f deployment_frontend.yaml

deployment.apps/frontend created
```

```
$ kubectl apply -f service_frontend.yaml 
service/frontend created
```

```
$ kubectl \
  --namespace cocktails \
  port-forward services/frontend :80 &

Forwarding from 127.0.0.1:35167 -> 8000
[1] 1117417
```

![bg right:33%](images/frontend_port_forward-2.png)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Guilhem Vellut (CC BY 2.0)" -->
We can't expect our end-users to
run kubectl commands or remember
random node ports.

Sure, we could configure a NodePort
service to listen on port 80/443,
but that would only allow us to
expose one web app per cluster.  

A better way is to utilize
**"ingress"** objects...

![bg right:30%](images/greenhouse_dome.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Freed eXplorer (CC BY 2.0)" -->
## What are ingresses?
Similar to services, but enables
routing/filtering on layer seven
_(basically a reverse proxy)_.  

Most commonly used for providing
external access to HTTP-based
services running in the cluster.  

Requests to _host x_ and path _y_
can be forwarded to _service z_.

Typically provided by a special
proxy application exposed as
a "LoadBalancer" service.

![bg right:30%](images/rusty_factory.jpg)

---
### ingress\_frontend.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: cocktails
  name: frontend
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  name: http-server
```

![bg right:33%](images/frontend_ingress.png)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Randy Adams (CC BY-SA 2.0)" -->
We're really getting some traction,
our users seem to love it!  

Let's add the ability to mark
cocktails as favorites.

_(and explore more of Kubernetes)_

![bg right:30%](images/abstract_pattern.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% William Warby (CC BY 2.0)" -->
## "favorites" application
Stores and exposes the name of
favorite cocktails in a database.  

Requires that clients authenticate
themselves with a token before
storing/exposing information.  

![bg right:30%](images/lamps.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% William Warby (CC BY 2.0)" -->
### deployment\_favorites.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: cocktails
  name: favorites
spec:
  replicas: 3
  selector:
    matchLabels:
      application: favorites
  template:
    metadata:
      labels:
        application: favorites
    spec:
      containers:
        - name: application-container
          image: ghcr.io/menacit/demo_apps/favorites:v1
          ports:
            - name: http-api
              containerPort: 8000
          env:
            - name: APP_DATABASE_URL
              value: http://database:4001/?disableClusterDiscovery=true
            - name: APP_DATABASE_USER
              valueFrom:
                secretKeyRef:
                  name: database
                  key: user
            - name: APP_DATABASE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: database
                  key: password
            - name: APP_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  name: favorites
                  key: access_key
[...]
```

![bg right:30%](images/lamps.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% William Warby (CC BY 2.0)" -->
### deployment\_favorites.yaml (env)
```yaml
[...]
    env:
      - name: APP_ACCESS_KEY
        valueFrom:
          secretKeyRef:
            name: favorites
            key: access_key
[...]
```

```
$ kubectl \
  --namespace cocktails \
  create secret generic favorites \
  --from-literal access_key=DO_NOT_USE

secret/favorites created
```

![bg right:30%](images/lamps.jpg)

---
With a bit of modification to
the frontend application/deployment,
we can enjoy the new favorites feature!

_(We'll skip the database setup for now)_

![bg right:33%](images/frontend_favorites-1.png)


---
With a bit of modification to
the frontend application/deployment,
we can enjoy the new favorites feature!

_(We'll skip the database setup for now)_

![bg right:33%](images/frontend_favorites-2.png)

---
With a bit of modification to
the frontend application/deployment,
we can enjoy the new favorites feature!

_(We'll skip the database setup for now)_

![bg right:33%](images/frontend_favorites-3.png)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Asparukh Akanayev (CC BY 2.0)" -->
Let's wrap up with some
security hardening.  

Restricting network communication
may be a good start...

![bg right:30%](images/brick_hole.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Pelle Sten (CC BY 2.0)" -->
### What are network policies?
Feature to restrict network traffic
between pods and external networks
_(basically firewall rules)_.  

Filters traffic primarily using
label selectors, not IP addresses.  

Enables us to express things like:

> Permit communication between
> pods with the label app=X
> and app=Z in namespace Y

![bg right:30%](images/locks.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Pelle Sten (CC BY 2.0)" -->
### networkpolicy\_analytics.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  namespace: cocktails
  name: analytics
spec:
  podSelector:
    matchLabels:
      application: analytics
  ingress:
    - from:
        - podSelector:
            matchLabels:
              application: frontend
  egress:
    - to:
        - podSelector:
            matchLabels:
              application: recipes
```

![bg right:30%](images/locks.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Stig Nygaard (CC BY 2.0)" -->
## Further neatness
Automatically scale up/down number of
worker nodes/pods depending on load.  

Control spread of applications across
multiple availability zones.  

Fine-grained privilege management via
**R**ole-**b**ased **a**ccess **c**ontrol.  
  
Insanely extendable.  

Tons of pre-built manifests and
community resources.

![bg right:30%](images/holmen.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% William Warby (CC BY 2.0)" -->
Wanna try it yourself?  

Demo applications and YAML files
are available on [GitHub](https://github.com/menacit/demo_apps).

**MicroK8s** and **Kubeadm** are
decent options to get started! 

_(managed clusters may restrict
what you can inspect/modify)_

![bg right:30%](images/test_dummy.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Fredrik Rubensson (CC BY-SA 2.0)" -->
## MicroK8S
Distribution of Kubernetes designed for
developer workstations.  

Plug and play single node cluster.  
  
Developed by Canonical, creators of Ubuntu.

```
$ sudo snap install microk8s --classic
$ microk8s status --wait-ready
$ microk8s enable dns
$ microk8s kubectl get pods --all-namespaces
```

![bg right:30%](images/astronaut_streetart.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Pedro Mendes (CC BY-SA 2.0)" -->
## Kubeadm
Tool for installing and maintaining
Kubernetes clusters.  

Developed by the Kubernetes project
to provide a "reference setup" and
act as a building block for other
tools like **Kubespray**.  

Great for practical learning!

Enables playing with high-availability,
upgrades, alternatives runtimes, etc.

![bg right:30%](images/arecibo.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Kevin Dooley (CC BY 2.0)" -->
If you want something that
just works with sane defaults,
try [Red Hat OpenShift/OKD](https://www.okd.io/).

![bg right:30%](images/biosphere_2.jpg)

---
<!-- _footer: "%ATTRIBUTION_PREFIX% Thierry Ehrmann (CC BY 2.0)" -->
## Wrapping up
Kubernetes is a fun but complex beast.  

Hopefully this talk has given you
some inspiration to go on your
own glorious adventure!

_(Or at least confidence to nod when someone asks
if you know what Kubernetes is)_

![bg right:30%](images/thinking_streetart.jpg)
