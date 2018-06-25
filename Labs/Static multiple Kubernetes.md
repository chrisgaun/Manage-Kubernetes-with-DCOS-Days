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

Currently, DC/OS supports multiple Kubernetes clusters with the limiation that there is one Kubernetes worker node per Mesos agent. In the early Fall, DC/OS will release a high density Kubernetes 

###############Install PreReqs Before Starting###############
dcos package install dcos-enterprise-cli --yes
dcos package install kafka --cli --yes
dcos package install kubernetes --package-version=1.0.3-1.9.7 --cli --yes
dcos security org service-accounts keypair private-key.pem public-key.pem
dcos security org service-accounts create -p public-key.pem -d 'kubernetes service account' kubernetes
dcos security secrets create-sa-secret private-key.pem kubernetes kubernetes/sa
dcos security org groups add_user superusers kubernetes

###############Show K8s Catalog Item###############
Show the various settings

###############Install K8s############### [10Min]
via the GUI
kubernetes
kubernetes/sa


###############Show Progress, Explain Deployment###############
dcos kubernetes plan status deploy
talk about fw scheduler, what its doing, etc

###############Make K8s HA############### [5Min]
Click HA box in GUI, show how easy it is to expan etcd, apiserver, etc.

###############Show Progress###############
dcos kubernetes plan status deploy

###############Show Secrets and Service Accounts###############
In GUI, walk through what has been created, why it is important, downloading the certs from endpoints tab
I don't need to worry about certs for the apiserver, etc, they are distributed to those new instances for me


###############Connect, Show using default API###############
dcos kubernetes kubeconfig
kubectl get nodes
kubectl get namespaces
kubectl get pods
kubectl create -f deployment.yaml
######Show that the yaml is no different, you do not need to change yamls for K8s on DCOS as it is vanilla K8s
kubectl get pods
####explain/show networking for pods, creation, allocation........

###############Upgrade K8s############### [15Min]
dcos package describe kubernetes --package-versions
dcos kubernetes update --package-version="1.0.3-1.9.7"

###############Show Progress###############
dcos kubernetes plan status deploy
show pods still running

###############Deploy Kafka(while waiting for K8s upgrade)###############
Talk about data services
Show options, TLS, zones, vips, JSON
Show deploying multiple instances of services
Deploy into AWS
Deploy into Azure
###############Scale K8s###############
dcos kubernetes update --options=k8s-package-options-scale.json
dcos kubernetes plan status deploy

###############Multi K8s###############
Deploy K8s 1.10.4 via GUI, adding following constraints:
name: kubernetes-2
service cidr: 10.200.0.0/16
[["@hostname", "unlike", "10.11.17.21|10.11.13.216|10.11.13.31|10.11.18.20|10.11.18.2"]]
Yay, two clusters.

###############Show Hybrid Cloud###############
Deploy /dcos-website
Container Image: mesosphere/dcos-website:cff383e4f5a51bf04e2d0177c5023e7cebcab3cc
Move website around, show that it respects HCs and stays available

###############Deploy App and tie it all together###############
dcos kafka topic create transactions --replication 3
dcos kafka topic create fraud --replication 3
Deploy Flink
Upload Jar to Flink
Submit job to Flink
dcos marathon app add https://raw.githubusercontent.com/dcos/demos/master/flink/1.11/generator/generator.json
#Look, no need to dockerize, how cool is that.
###############Show Output###############
dcos marathon app add https://raw.githubusercontent.com/dcos/demos/master/flink/1.11/actor/fraudDisplay.json
Show stdout log, will see transactions after about 1 min or so.
###############Move Workloads Around###############
move back and forth between local and remote regions
logs still work, after slight pause transactions resume

###############Delete and Move into K8s###############
Delete generator and display from marathon
dcos kubernetes kubeconfig to the original K8s cluster if needed
kubectl apply -f https://raw.githubusercontent.com/dcos/demos/master/flink-k8s/1.11/generator/flink-demo-generator.yaml
kubectl apply -f https://raw.githubusercontent.com/dcos/demos/master/flink-k8s/1.11/actor/flink-demo-actor.yaml
kubectl get deployments
kubectl get pods
kubectl logs flink-demo-generator<insert>
Show it is working the same.

###############Done###############



amrabdelra-tfbe9a-pub-agt-elb-2125765910.us-east-1.elb.amazonaws.com

amrabdelra-tfbe9a-pub-agt-elb-2125765910.us-east-1.elb.amazonaws.com





mesosphere/dcos-website:cff383e4f5a51bf04e2d0177c5023e7cebcab3cc

HAPROXY_GROUP
HAPROXY_0_VHOST


