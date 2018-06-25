### Automated Self Healing

Kubernetes with DC/OS includes automated self-healing of Kubernetes infrastructure. 

We can demo this by killing some of the Kubernetes nodes. 

In the command line enter

```
dcos task exec -it kube-node-0-kubelet bash
```

Now enter

```
ps aux | grep "./kubelet" 
```

You should see some output that looks like

```
root         3  0.0  0.0  11960  2812 ?        S    21:53   0:00
```

In the command line output we are looking for the number after "root" (in this case 3).

Make sure to have the DC/OS UI > Services > Kubernetes open next to the terminal so you can see the components. 

Then modify and run the following command with the number from the last root output

```
kill -9 3
```

You may need to refresh brower. Watch as Kubernetes nodes are killed then come back online. 
