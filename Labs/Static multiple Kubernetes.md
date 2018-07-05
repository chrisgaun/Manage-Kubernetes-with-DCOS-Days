## Why Multiple Kubernetes Clusters?

Having multiple Kubernetes clusters is a service reliability best practice. 

Kubernetes has some multi-tentancy from namespaces, but there are a variety of reason why best practices require any strategy to be combined with multiple Kubernetes clusters:

* Application owners can control upgrades, scaling and entire lifecyle
* Require multi-node cluster with full API access - e.g. development
* High availability in case of cluster outages
* Developement pipeline (e.g. dev, test, staging, production)
* Mutliple independent workloads 
* Different regions or availability zones
* Security and compliance (worried about noisy or nosy neighbors)

For operations in particular, althought there is no public guidance, individual lines of businesses wishing to have a Kubernetes cluster ofen require on order of 10 clusters to satisfy initial use cases. 

### Multiple Kubernetes Cluters in DC/OS

Before you start, make sure you have enough DC/OS nodes. If you want a 2 Kubernetes clusters with 7 nodes (including the Master and etcd nodes) then you will need 14 DC/OS nodes in current release. 

Currently, DC/OS supports multiple Kubernetes clusters with the limiation that there is one Kubernetes worker node per Mesos agent. In the early Fall of 2018, DC/OS will release a high density Kubernetes package that can include multiple kubelets per Mesos agent:

![](https://i.imgur.com/5xbyAQK.png)

Follows is instructions on how to install multiple Kubernetes clusters using DC/OS Placement Constraints. 

### Provision First Kubernetes Cluster 

If you do not already have a Kubernetes cluster on DC/OS, follow the instructions [here](https://github.com/chrisgaun/Manage-Kubernetes-with-DCOS-Days/blob/chrisgaun-patch-1/Labs/Lab%201%20-%20Installing%20Kubernetes.md) to get the first one up and running. 

### Provision Multiple Kubernetes Clusters

Multiple Kubernetes clusters is made possible via Placement Constraints. Before you start, navigate to the DC/OS Nodes tab in the GUI and record the IP addresses of 5 of the unused nodes. 

It is easiest to deploy multiple Kubernetes using the DC/OS GUI. Navigate to the DC/OS Catalog in the GUI (left menu) and search for "Kubernetes". Deploy K8s 1.10.4 using "Review & Run" 

![](https://i.imgur.com/PphTYDg.png)

Change the name of the Kubernetes service to something unique. 

```
kubernetes-2
```

Under Kubernetes change the CIDR to not overlap. 

```
service cidr: 10.200.0.0/16
```

Under Kubernetes -> "control plane placement" and "node placement" add the addresses of the occupied nodes. 

```
[["@hostname", "unlike", "<IP_ADDRESS_1>|<IP_ADDRESS_2>|<IP_ADDRESS_3>|<IP_ADDRESS_4>|<IP_ADDRESS_5>"]]
```
