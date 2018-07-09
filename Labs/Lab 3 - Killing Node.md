## Automated Self Healing

Kubernetes with DC/OS includes automated self-healing of Kubernetes infrastructure. 

We can demo this by killing some of the Kubernetes framework components, as well as nodes. 

In the command line enter

```
dcos task
```

Output should resemble:
```
$ dcos task
NAME                                HOST        USER  STATE  ID                                                                        MESOS ID                                      REGION          ZONE
etcd-0-peer                         10.0.5.79   root    R    etcd-0-peer__181046fd-6ed0-4ac2-bfaa-9ec15e944226                         4f6be22e-f057-4226-9c40-190496ea9218-S16  aws/us-west-2  aws/us-west-2a
etcd-1-peer                         10.0.7.236  root    R    etcd-1-peer__e4695eff-4318-4b26-98d7-b97db91ddfdd                         4f6be22e-f057-4226-9c40-190496ea9218-S7   aws/us-west-2  aws/us-west-2a
etcd-2-peer                         10.0.7.104  root    R    etcd-2-peer__0d63eb6c-4fe4-4498-b82c-d53d53b36000                         4f6be22e-f057-4226-9c40-190496ea9218-S4   aws/us-west-2  aws/us-west-2a
kube-apiserver-0-instance           10.0.5.243  root    R    kube-apiserver-0-instance__e780ad3f-2586-4905-8724-f15efd73851f           4f6be22e-f057-4226-9c40-190496ea9218-S5   aws/us-west-2  aws/us-west-2a
kube-apiserver-1-instance           10.0.7.236  root    R    kube-apiserver-1-instance__64a0f0a3-767b-4494-81e9-74988f0340d0           4f6be22e-f057-4226-9c40-190496ea9218-S7   aws/us-west-2  aws/us-west-2a
kube-apiserver-2-instance           10.0.5.151  root    R    kube-apiserver-2-instance__40bdca05-77da-44c9-ae83-9e0ecbcae7e3           4f6be22e-f057-4226-9c40-190496ea9218-S1   aws/us-west-2  aws/us-west-2a
kube-controller-manager-0-instance  10.0.6.4    root    R    kube-controller-manager-0-instance__21fbf104-a8d8-4d0c-951d-6a649fe3ca4b  4f6be22e-f057-4226-9c40-190496ea9218-S14  aws/us-west-2  aws/us-west-2a
kube-controller-manager-1-instance  10.0.5.209  root    R    kube-controller-manager-1-instance__0036194d-00da-4cf4-a825-5d5eceabf56d  4f6be22e-f057-4226-9c40-190496ea9218-S15  aws/us-west-2  aws/us-west-2a
kube-controller-manager-2-instance  10.0.7.176  root    R    kube-controller-manager-2-instance__011d0e3c-8082-47e5-98ca-407ee507e754  4f6be22e-f057-4226-9c40-190496ea9218-S3   aws/us-west-2  aws/us-west-2a
kube-node-0-coredns                 10.0.4.185  root    R    kube-node-0-coredns__992099bf-e867-4db7-be4a-aab4ca65edd7                 4f6be22e-f057-4226-9c40-190496ea9218-S9   aws/us-west-2  aws/us-west-2a
kube-node-0-kube-proxy              10.0.4.185  root    R    kube-node-0-kube-proxy__ca862dcd-2fbd-43de-b708-8c96de6109d2              4f6be22e-f057-4226-9c40-190496ea9218-S9   aws/us-west-2  aws/us-west-2a
kube-node-0-kubelet                 10.0.4.185  root    R    kube-node-0-kubelet__967c1659-a6d6-4e9a-a544-020b36dd27cb                 4f6be22e-f057-4226-9c40-190496ea9218-S9   aws/us-west-2  aws/us-west-2a
kube-node-1-coredns                 10.0.6.170  root    R    kube-node-1-coredns__db4df874-24fa-4717-9858-62e0cec0e8b1                 4f6be22e-f057-4226-9c40-190496ea9218-S10  aws/us-west-2  aws/us-west-2a
kube-node-1-kube-proxy              10.0.6.170  root    R    kube-node-1-kube-proxy__a1d35522-0d99-454b-b86c-96821be0d36d              4f6be22e-f057-4226-9c40-190496ea9218-S10  aws/us-west-2  aws/us-west-2a
kube-node-1-kubelet                 10.0.6.170  root    R    kube-node-1-kubelet__3ea399b3-f1b1-43d9-8064-2b663a6132bc                 4f6be22e-f057-4226-9c40-190496ea9218-S10  aws/us-west-2  aws/us-west-2a
kube-scheduler-0-instance           10.0.5.79   root    R    kube-scheduler-0-instance__85eee02c-2ffa-41d7-be72-41367d3785e9           4f6be22e-f057-4226-9c40-190496ea9218-S16  aws/us-west-2  aws/us-west-2a
kube-scheduler-1-instance           10.0.6.4    root    R    kube-scheduler-1-instance__8e9d3974-dad5-4268-8711-c79c86370779           4f6be22e-f057-4226-9c40-190496ea9218-S14  aws/us-west-2  aws/us-west-2a
kube-scheduler-2-instance           10.0.5.243  root    R    kube-scheduler-2-instance__50807610-679c-4dc2-a55b-cb9055a6b4c9           4f6be22e-f057-4226-9c40-190496ea9218-S5   aws/us-west-2  aws/us-west-2a
kubernetes                          10.0.5.243  root    R    kubernetes.d7696234-839a-11e8-95d4-de01896a7e14                           4f6be22e-f057-4226-9c40-190496ea9218-S5   aws/us-west-2  aws/us-west-2a
kubernetes-proxy                    10.0.4.167  root    R    kubernetes-proxy.65c752a2-8394-11e8-95d4-de01896a7e14                     4f6be22e-f057-4226-9c40-190496ea9218-S6   aws/us-west-2  aws/us-west-2a
```

### Kill etcd
Lets kill an instance of the etcd database to observe auto-healing capabilities:

First we need to identify the etcd-0 PID value. In the example below the etcd PID value is 3:
```
$ dcos task exec -it etcd-0-peer ps ax
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     0:00 /opt/mesosphere/active/mesos/libexec/mesos/mesos-cont
    3 ?        Sl     2:04 ./etcd-v3.3.3-linux-amd64/etcd --name=infra0 --cert-f
 7008 pts/0    Ss+    0:00 /opt/mesosphere/active/mesos/libexec/mesos/mesos-cont
 7009 pts/0    R+     0:00 ps ax
 ```
 
Navigate to the DC/OS UI > Services > Kubernetes tab and open next to the terminal so you can see the components in the DC/OS UI. Use the search bar to search for etcd


Kill the etcd manually and watch the UI auto-heal the etcd instance:
```
dcos task exec -it etcd-0-peer kill -9 3
```

You may need to refresh brower. Watch as Kubernetes etcd-0 instance is killed and respawned

### Kill a Kubelet
Next, lets kill a Kubernetes node to observe auto-healing capabilities:

First we need to identify the kube-node-0 PID value. Enter etcd PID value associated with the cmd: `sh -c ./bootstrap --resolve=false 2>&1  chmod +x kube`: In the example below the etcd PID value is 3:

```
$ dcos task exec -it kube-node-0-kubelet ps ax
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     0:00 /opt/mesosphere/active/mesos/libexec/mesos/mesos-cont
    3 ?        S      0:00 sh -c ./bootstrap --resolve=false 2>&1  chmod +x kube
   10 ?        S      0:00 /bin/bash ./kubelet-wrapper.sh
```

Navigate to the DC/OS UI > Services > Kubernetes tab and open next to the terminal so you can see the components in the DC/OS UI. Use the search bar to search for kube-node-0

Kill the etcd manually and watch the UI auto-heal the etcd instance:

```
dcos task exec -it kube-node-0-kubelet kill -9 3
```

You may need to refresh brower. Watch as Kubernetes Kubelet instance is killed and respawned
