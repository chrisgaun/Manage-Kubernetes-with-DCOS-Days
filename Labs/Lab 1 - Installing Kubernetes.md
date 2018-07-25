### Access the Cluster

The instructor will give you access to IP address and credentials that you will need to SSH into.

### Set Up DC/OS Command Line

Set up the DC/OS command line by clicking on the top left and choosing "install CLI"

![CLI](https://i.imgur.com/p4kqIj6.png)

Click in the dialogue box too copy the command

![Copy Command](https://i.imgur.com/3rQ2Unj.png)

Paste that command into your Terminal and press enter

Confirm that dcos is installed and connected to your cluster by running following command

```
dcos node
```

The output should be a list of nodes in the cluster

```

   HOSTNAME        IP                         ID                     TYPE                 REGION          ZONE       
  10.0.0.101   10.0.0.101  94141db5-28df-4194-a1f2-4378214838a7-S0   agent            aws/us-west-2  aws/us-west-2a  
  10.0.2.100   10.0.2.100  94141db5-28df-4194-a1f2-4378214838a7-S4   agent            aws/us-west-2  aws/us-west-2a
```

### Tour DC/OS Catalog

Your instructor will give you a tour of DC/OS UI and catalog. 

### Install Kubernetes 

To install Kubernetes enter this command into your terminal

```
dcos package install kubernetes --package-version=1.1.0-1.10.3
```

You can see the installation runbook automation and status of installation of each component with this command

```
dcos kubernetes plan status deploy
```

It should look like this when completed

```
$ dcos kubernetes plan status deploy
deploy (serial strategy) (COMPLETE)
├─ etcd (serial strategy) (COMPLETE)
│  └─ etcd-0:[peer] (COMPLETE)
├─ apiserver (parallel strategy) (COMPLETE)
│  └─ kube-apiserver-0:[instance] (COMPLETE)
├─ mandatory-addons (serial strategy) (COMPLETE)
│  ├─ mandatory-addons-0:[additional-cluster-role-bindings] (COMPLETE)
│  ├─ mandatory-addons-0:[kube-dns] (COMPLETE)
│  ├─ mandatory-addons-0:[metrics-server] (COMPLETE)
│  ├─ mandatory-addons-0:[dashboard] (COMPLETE)
│  └─ mandatory-addons-0:[ark] (COMPLETE)
├─ kubernetes-api-proxy (parallel strategy) (COMPLETE)
│  └─ kubernetes-api-proxy-0:[install] (COMPLETE)
├─ controller-manager (parallel strategy) (COMPLETE)
│  └─ kube-controller-manager-0:[instance] (COMPLETE)
├─ scheduler (parallel strategy) (COMPLETE)
│  └─ kube-scheduler-0:[instance] (COMPLETE)
├─ node (parallel strategy) (COMPLETE)
│  └─ kube-node-0:[kube-proxy, coredns, kubelet] (COMPLETE)
└─ public-node (parallel strategy) (COMPLETE)
```

When all steps are "COMPLETE", confirm that the "dcos kubernetes" CLI was installed.

```
dcos kubernetes
```

### Install Kubernetes kubectl Command Line

Install the Kubernetes command line by following instructions [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

**For Macs** with brew installed the command is

```
brew install kubectl
```

**For Ubuntu** the command is

```
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo touch /etc/apt/sources.list.d/kubernetes.list 
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

Confirm that kubectl is installed and in path /usr/local/bin (it will say it is not connected to dcos cluster yet which is ok)

```
kubectl version
```

### Connecting kubectl to DC/OS
Deploy Marathon-LB:
```
dcos package install marathon-lb
```

kubectl-proxy service:
```
$ cat <<EOF > kubectl-proxy.json
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
    "HAPROXY_0_MODE": "http",
    "HAPROXY_GROUP": "external",
    "HAPROXY_0_SSL_CERT": "/etc/ssl/cert.pem",
    "HAPROXY_0_PORT": "6443",
    "HAPROXY_0_BACKEND_SERVER_OPTIONS": "  server kube-apiserver apiserver.kubernetes.l4lb.thisdcos.directory:6443 ssl verify none\n"
  }
}
EOF
```

Deploy kubectl-proxy service:
```
dcos marathon app add kubectl-proxy.json
```

Find your public DC/OS agent IP (make sure to have jq installed - on Mac "brew install jq"

```
for id in $(dcos node --json | jq --raw-output '.[] | select(.attributes.public_ip == "true") | .id'); \
do dcos node ssh --option StrictHostKeyChecking=no --option LogLevel=quiet --master-proxy --mesos-id=$id "curl -s ifconfig.co" ; done 2>/dev/null
```

Connect kubectl to DC/OS:
```
dcos kubernetes kubeconfig \
    --apiserver-url https://<MARATHON-LB_PUBLIC_IP>:6443 \
    --insecure-skip-tls-verify
```

Confirm connection:

```
kubectl get nodes
```

### Kubernetes Dashboard (Official UI of Kubernetes)

To access the dashboard run:

```
kubectl proxy
```

Point your browser to:

```
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/
```
