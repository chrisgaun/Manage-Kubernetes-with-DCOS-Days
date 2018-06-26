## Marathon-lb

Marathon-lb provides wrapper around HA proxy. Make sure DC/OS and K8s have public nodes. 

Create a Marathon-lb service in the service catalog

```
dcos package install marathon-lb
```

Create a Marathon application with following JSON. 
```
{
  "id": "/kubectl-proxy",
  "instances": 1,
  "cpus": 0.001,
  "mem": 16,
  "cmd": "tail -F /dev/null",
  "container": {
    "type": "MESOS"
  },
  "portDefinitions": [
    {
      "protocol": "tcp",
      "port": 0
    }
  ],
  "labels": {
    "HAPROXY_GROUP": "external",
    "HAPROXY_0_MODE": "http",
    "HAPROXY_0_PORT": "6443",
    "HAPROXY_0_SSL_CERT": "/etc/ssl/cert.pem",
    "HAPROXY_0_FRONTEND_HEAD": "\nfrontend {backend}\n  bind {bindAddr}:{servicePort}{sslCert}{bindOptions}\n  mode {mode}\n  timeout connect 5s\n  timeout client 15m\n  timeout server 15m\n  timeout tunnel 5s\n",
    "HAPROXY_0_BACKEND_SERVER_OPTIONS": "  timeout connect 5s\n  timeout client 15m\n  timeout server 15m\n  timeout tunnel 5s\n  server kube-apiserver apiserver.kubernetes.l4lb.thisdcos.directory:6443 ssl verify none\n"   
  }
}
```

Here is how this works:
* Marathon-LB identifies that the application kubeapi-proxy has the HAPROXY_GROUP label set to external (change this if you're using a different HAPROXY_GROUP for your Marathon-LB configuration).
* The instances, cpus, mem, cmd, and container fields basically create a dummy container that takes up minimal space and performs no operation.
* The single port indicates that this application has one "port" (this information is used by Marathon-LB)
* "HAPROXY_0_MODE": "http" indicates to Marathon-LB that the frontend and backend configuration for this particular service should be configured with http.
* "HAPROXY_0_PORT": "6443" tells Marathon-LB to expose the service on port 6443 (rather than the randomly-generated service port, which is ignored)
* "HAPROXY_0_SSL_CERT": "/etc/ssl/cert.pem" tells Marathon-LB to expose the service with the self-signed Marathon-LB certificate (which has no CN)
* The last label HAPROXY_0_BACKEND_SERVER_OPTIONS indicates that Marathon-LB should forward traffic to the endpoint apiserver.kubernetes.l4lb.thisdcos.directory:6443 rather than to the dummy application, and that the connection should be made using TLS without verification.

Find your public DC/OS agent IP

```
for id in $(dcos node --json | jq --raw-output '.[] | select(.attributes.public_ip == "true") | .id'); \
do dcos node ssh --option StrictHostKeyChecking=no --option LogLevel=quiet --master-proxy --mesos-id=$id "curl -s ifconfig.co" ; done 2>/dev/null
```

Connect kubectl using following command and replacing “https://kube-apiserver.example.com:6443” with the public IP address 

```
$ dcos kubernetes kubeconfig \
    --apiserver-url https://kube-apiserver.example.com:6443 \
    --insecure-skip-tls-verify
```

You will be able to access the Kubernetes Dashboard by running:

```
$ kubectl proxy
```

Then open the Dashboard with following URL:

```
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/
```

