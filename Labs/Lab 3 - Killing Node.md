### Automated Self Healing

Kubernetes with DC/OS includes automated self-healing of Kubernetes infrastructure. 

We can demo this by killing some of the Kubernetes nodes. 

In the command line enter

```
dcos task exec -it kube-node-0-kubelet bash
```

```
ps aux | grep "./kubelet " root 62  2.3  0.6 741984 100580 ? Sl 15:16 3:25 ./kubelet --address=10.0.2.172 --hostname-override=kube-node-1-kubelet.kubernetes.mesos
```

```
kill -9 62
```
