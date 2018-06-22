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
dcos package install kubernetes --package-version=1.0.3-1.9.7
```

You can see the installation runbook automation and status of installation of each component with this command

```
dcos kubernetes plan show deploy
```

It should look like this when completed

```
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
   └─ kube-node-public-0:[kube-proxy, coredns, kubelet] (COMPLETE)
```

When all steps are "COMPLETE", confirm that the "dcos kubernetes" CLI was installed.

```
dcos kubernetes
```

### Kubernetes Dashboard (Official UI of Kubernetes)

DC/OS packages of Kubernetes versions 1.9.x automatically create an SSH tunnel to the Dashboard and connect to the API server

To access the Dashboard, hover over "kube-proxy" and click the new tab icon

![](https://i.imgur.com/rxUlcd7.png)
