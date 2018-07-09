### Access the Cluster

The Lab instructor will give you access the web UI via cluster IP address as well as any login credentials necessary to complete these labs

### Set Up DC/OS Command Line

Set up the DC/OS command line by clicking on the top left and choosing "install CLI"

![CLI](https://i.imgur.com/p4kqIj6.png)

Click in the dialogue box to copy the command

![Copy Command](https://i.imgur.com/3rQ2Unj.png)

Paste that command into your Terminal and press enter

You will be prompted for the cluster username/password. These will be provided by the lab instructor

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
dcos package install kubernetes --package-version=1.0.3-1.9.7
```

You can see the installation runbook automation and status of installation of each component with this command.

```
dcos kubernetes plan status deploy
```

It should look like this when completed:

```
deploy (serial strategy) (COMPLETE)
├─ etcd (serial strategy) (COMPLETE)
│  └─ etcd-0:[peer] (COMPLETE)
├─ apiserver (parallel strategy) (COMPLETE)
│  └─ kube-apiserver-0:[instance] (COMPLETE)
├─ kubernetes-api-proxy (parallel strategy) (COMPLETE)
│  └─ kubernetes-api-proxy-0:[install] (COMPLETE)
├─ controller-manager (parallel strategy) (COMPLETE)
│  └─ kube-controller-manager-0:[instance] (COMPLETE)
├─ scheduler (parallel strategy) (COMPLETE)
│  └─ kube-scheduler-0:[instance] (COMPLETE)
├─ node (parallel strategy) (COMPLETE)
│  ├─ kube-node-0:[kube-proxy] (COMPLETE)
│  ├─ kube-node-0:[coredns] (COMPLETE)
│  └─ kube-node-0:[kubelet] (COMPLETE)
├─ public-node (parallel strategy) (COMPLETE)
└─ mandatory-addons (serial strategy) (COMPLETE)
   ├─ mandatory-addons-0:[kube-dns] (COMPLETE)
   ├─ mandatory-addons-0:[metrics-server] (COMPLETE)
   ├─ mandatory-addons-0:[dashboard] (COMPLETE)
   └─ mandatory-addons-0:[ark] (COMPLETE)
```

When all steps are "COMPLETE", confirm that the DC/OS Kubernetes CLI was installed.

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

Once installed, connect kubectl to cluster

```
dcos kubernetes kubeconfig
```

Confirm connection

```
kubectl get nodes
```


### Kubernetes Dashboard (Official UI of Kubernetes)

DC/OS packages of Kubernetes versions 1.9.x automatically create an SSH tunnel to the Dashboard and connect to the API server

To access the Dashboard, hover over "kube-proxy" and click the new tab icon

![](https://i.imgur.com/EAlNXAy.png)

Your instructor will now give you an overview of Kubernetes constructs using the UI
