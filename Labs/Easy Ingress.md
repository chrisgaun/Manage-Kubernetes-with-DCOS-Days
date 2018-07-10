[Kubernetes Service Documentation - External Ingress](https://docs.mesosphere.com/services/kubernetes/1.2.0-1.10.5/ingress/)

## Adding External Ingress

If you havent already done so, scale your Kubernetes `"public_node_count": 1,`. Go back to Lab 5 for detailed instructions on how to scale/update the Kubernetes Framework

If you want to expose HTTP/S (L7) apps to the outside world - at least outside the DC/OS cluster - you should create a Kubernetes Ingress resource. The package does not install such controller by default, so we give you the freedom to choose what Ingress controller your organization wants to use.
Options:
- Traefik
- NGINX
- HAProxy
- Envoy
- Istio

### For this Lab we will install Traefik as our Kubernetes Ingress Controller to show a Hello World example

Save the below Traefik Ingress Controller as `traefik.yaml`
```
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: traefik-ingress-lb
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      containers:
      - image: traefik
        name: traefik-ingress-lb
        args:
        - --web
        - --kubernetes
# NOTE: What follows are necessary additions to
# https://docs.traefik.io/user-guide/kubernetes
# Please check below for a detailed explanation
        ports:
        - containerPort: 80
          hostPort: 80
          name: http
          protocol: TCP
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: k8s-app
                operator: In
                values:
                - traefik-ingress-lb
            topologyKey: "kubernetes.io/hostname"
      nodeSelector:
        kubernetes.dcos.io/node-type: public
      tolerations:
      - key: "node-type.kubernetes.dcos.io/public"
        operator: "Exists"
        effect: "NoSchedule"
```


Save the below ingress controller configuation as `traefik-ingress.yaml`
```
apiVersion: v1
kind: Service
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-controller
  ports:
    - port: 80
      name: http
  type: NodePort
  ```
 
Deploy your Traefik Ingress Controller
```
kubectl create -f traefik.yaml
```

Create Ingress Service:
```
kubectl create -f traefik-ingress.yaml
```

Create a Hello World Service and name it `hello-world.yaml`:
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
        - -text="Hello from Kubernetes running on Mesosphere DC/OS!"
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
```

Deploy Hello World Service:
```
kubectl create -f hello-world.yaml
```

Create Hello World Service Ingress and name it `hw-ingress.yaml`:
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: traefik
  name: hello-world
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: hello-world
          servicePort: 80
```

Deploy Hello World Ingress:
```
kubectl create -f hw-ingress.yaml
```
  
Access Hello World Application by accessing `http://<PUBLIC_KUBELET_IP>`
  






