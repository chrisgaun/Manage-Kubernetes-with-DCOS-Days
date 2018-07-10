The first step we’re going to need to do is to deploy a default backend service. This is used by the Nginx ingress controller in the event that services are unavailable. The default backend image needs to satisfy two requirements :

Serve a 404 page at /
Respond with a 200 for requests to /healthz

The ingress-nginx project provides us with a template yaml ( https://github.com/kubernetes/ingress-nginx/blob/master/deploy/default-backend.yaml ) to use for this purpose, and the only change we’ve made from the defaults is to remove the namespace. 

```
---

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: default-http-backend
  labels:
    app: default-http-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: default-http-backend
  template:
    metadata:
      labels:
        app: default-http-backend
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - name: default-http-backend
        # Any image is permissible as long as:
        # 1. It serves a 404 page at /
        # 2. It serves 200 on a /healthz endpoint
        image: gcr.io/google_containers/defaultbackend:1.4
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 30
          timeoutSeconds: 5
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: 10m
            memory: 20Mi
          requests:
            cpu: 10m
            memory: 20Mi
---

apiVersion: v1
kind: Service
metadata:
  name: default-http-backend
  labels:
    app: default-http-backend
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: default-http-backend

```

As we can see, this is going to deploy a single replica of the defaultbackend container, which we are getting from gcr.io, and our service will route port 80 requests to port 8080 on the target. 

```
Mattbook-Pro:ingress matt$ kubectl create -f default-backend.yaml 
deployment "default-http-backend" created
service "default-http-backend" created
```

Now we have our default backend, we can go ahead and deploy our Nginx ingress controller. This will deploy both Nginx and the additional code for Kubernetes to control it. 

Here is the YAML you can save as "nginx-controller-simple.yaml"

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata: 
  name: nginx-ingress-controller
spec: 
  replicas: 1
  revisionHistoryLimit: 3
  template: 
    metadata: 
      labels: 
        k8s-app: ingress-nginx
    spec: 
      containers: 
        - args: 
            - /nginx-ingress-controller
            - "--default-backend-service=$(POD_NAMESPACE)/default-http-backend"
          env: 
            - name: POD_NAME
              valueFrom: 
                fieldRef: 
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom: 
                fieldRef: 
                  fieldPath: metadata.namespace
          image: "quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.15.0"
          imagePullPolicy: Always
          livenessProbe: 
            httpGet: 
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            timeoutSeconds: 5
          name: nginx-ingress-controller
          ports: 
            - containerPort: 80
              hostPort: 80
              name: http
              protocol: TCP
      terminationGracePeriodSeconds: 60
      nodeSelector:
          kubernetes.dcos.io/node-type: public
      tolerations:
        - key: "node-type.kubernetes.dcos.io/public"
          operator: "Exists"
          effect: "NoSchedule"
---
```

Deploy YAML using following command

```
Mattbook-Pro:ingress matt$ kubectl create -f nginx-controller-simple.yaml 
deployment "nginx-ingress-controller" created
```

There are a few things worth noting in this configuration, which are specific to deploying this in DC/OS. Firstly, we need to ensure that this is deployed to our public agents, which is defined by :

```
  nodeSelector:
          kubernetes.dcos.io/node-type: public
```

We have also defined some tolerations, which define the behaviour we want to ensure if that condition can’t be met. In this case, we are defining that unless we have a public node available, then we’re not going to schedule the deployment.

```
 tolerations:
        - key: "node-type.kubernetes.dcos.io/public"
          operator: "Exists"
          effect: "NoSchedule"
```

We’ve also bound the deployment container directly to port 80 on the host it is deployed onto :

```
          ports: 
            - containerPort: 80
              hostPort: 80
              name: http
              protocol: TCP
```             

We could define a Service, and then have the ports allocated automatically by Kubernetes using a NodePort type, but DC/OS already exposes port 80 on public agents, so this simplifies our configuration. In a production environment, this may not be the best option, since we need to manually ensure that no other processes are using port 80 on any of our DC/OS public agents. This includes any scenario where you are running marathon-lb in it’s default configuration, since marathon-lb binds to port 80 on public agents and so will prevent this ingress configuration from working correctly.

Once our ingress controller is running, we can test connectivity to our public IP address. Find the public IP using:

```
```

Once we have our public IP, we can use curl to connect to port 80 on this host. 

Mattbook-Pro:ingress matt$ curl 54.171.202.99
default backend - 404

We can see from the response code that we’re hitting our default backend, which is exactly what the expected behaviour is since we don’t yet have any ingress configuration. 

Let’s go ahead and deploy a simple service we can use for our first test of the ingress controller. We’re going to use 

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
  labels:
    app: hello-world
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: echo-server
        image: hashicorp/http-echo
        args:
        - -listen=:80
        - -text="Hello from Kubernetes!"
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: hello-world
spec:
  selector:
    app: hello-world
  ports:
    - port: 80
      targetPort: 80
``

Lets go ahead and deploy that :

Mattbook-Pro:ingress matt$ kubectl create -f helloworld.yaml
deployment "hello-world" created
service "hello-world" created

We now need to define an Ingress for the ingress controller to configure external access to our test application. 

Mattbook-Pro:ingress matt$ cat helloworld-ingress.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: hello-world
          servicePort: 80

In this configuration, we use an annotation to define which ingress class Kubernetes should use, we define the backend we should use, in this case our hello-world service, and we define the host, path and port which the ingress should be used on. 

Mattbook-Pro:ingress matt$ kubectl create -f helloworld-ingress.yaml 
ingress "hello-world-ingress" created

In order to test this, we need to do some local configuration on our kubectl host to map api.example.com to the public IP of our DC/OS cluster. This will depend on your operating system, but on MacOS and Linux you will want to add an entry in your /etc/hosts file as below :

Mattbook-Pro:ingress matt$ cat /etc/hosts
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1	localhost
255.255.255.255	broadcasthost
::1             localhost
54.171.202.99 api.example.com

With that entry in our /etc/hosts file, we can now resolve api.example.com to the public IP of our DC/OS cluster, so we can use curl once again to test out the ingress rule. 

Mattbook-Pro:ingress matt$ curl api.example.com
"Hello from Kubernetes!"

Success ! We see that our request has been routed to the correct service. We can also test that if we use the IP address instead of the hostname, then we hit the default backend with a 404 response, as expected.

Mattbook-Pro:ingress matt$ curl 54.171.202.99
default backend - 404

Let’s add a second service, which does something slightly different :

```
Mattbook-Pro:ingress matt$ cat holamundo.yaml 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hola-mundo
  labels:
    app: hola-mundo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hola-mundo
  template:
    metadata:
      labels:
        app: hola-mundo
    spec:
      containers:
      - name: echo-server
        image: hashicorp/http-echo
        args:
        - -listen=:80
        - -text="Hola de Kubernetes!"
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: hola-mundo
spec:
  selector:
    app: hola-mundo
  ports:
    - port: 80
      targetPort: 80
```

Mattbook-Pro:ingress matt$ kubectl create -f holamundo.yaml 
deployment "hola-mundo" created
service "hola-mundo" created


And now let’s configure ingress to this on a different path to the same host :

Mattbook-Pro:ingress matt$ cat holamundo-ingress.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: holamundo-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /hola
        backend:
          serviceName: hola-mundo
          servicePort: 80


Mattbook-Pro:ingress matt$ kubectl create -f holamundo-ingress.yaml 
ingress "holamundo-ingress" created

Now when we curl our endpoints, we can see that on the root we have our first service, and on the /hola endpoint we have our new service. 

Mattbook-Pro:ingress matt$ curl api.example.com
"Hello from Kubernetes!"
Mattbook-Pro:ingress matt$ curl api.example.com/hola
"Hola de Kubernetes!"

Let’s try something different, first let’s add another couple of host entries to our /etc/hosts file :

Mattbook-Pro:ingress matt$ cat /etc/hosts
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1	localhost
255.255.255.255	broadcasthost
::1             localhost
54.171.202.99 api.example.com
54.171.202.99 es.example.com
54.171.202.99 gb.example.com

And now lets add some host based routing to our ingress configuration :

Mattbook-Pro:ingress matt$ cat hosts-ingress.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hosts-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: gb.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: hello-world
          servicePort: 80
  - host: es.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: hola-mundo
          servicePort: 80

Mattbook-Pro:ingress matt$ kubectl create -f hosts-ingress.yaml 
ingress "hosts-ingress" created

With this configuration we basically have name based virtual hosts, where depending on the host requested we route the traffic differently. Here we are just using the root path, but of course, we can also combine this with paths for a ton of flexibility. 

Mattbook-Pro:ingress matt$ curl gb.example.com
"Hello from Kubernetes!"
Mattbook-Pro:ingress matt$ curl es.example.com
"Hola de Kubernetes!"

We can even do path rewrites in our ingress rules :

Mattbook-Pro:ingress matt$ cat rewrite.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: rewrite
  annotations:
    ingress.kubernetes.io/rewrite-target: /old
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: "api.example.com"
    http:
      paths:
      - path: /new
        backend:
          serviceName: hello-world
          servicePort: 80

Using the ingress.kubernetes.io/rewrite-target annotation, we can ensure that any requests to /old will be rewritten to direct to /new.
