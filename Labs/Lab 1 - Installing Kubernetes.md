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

