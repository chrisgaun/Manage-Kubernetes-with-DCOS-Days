### Automated Self Healing

Kubernetes with DC/OS includes automated self-healing of Kubernetes infrastructure. 

We can demo this by killing some of the Kubernetes nodes. 

In the command line enter

```
dcos task exec -it kube-node-0-kubelet bash
```
Make sure to have the DC/OS UI > Services > Kubernetes open next to the terminal so you can see the components. 

Then run

```
kill -9 62
```
