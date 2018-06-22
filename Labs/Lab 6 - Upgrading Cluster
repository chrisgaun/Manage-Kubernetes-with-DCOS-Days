Below is information from documentation, which stronly suggest you backup before upgrading. You can find the full documentation on upgrading the cluster [here](https://docs.mesosphere.com/services/kubernetes/1.1.1-1.10.4/upgrade/)

### Updating the package version
Before starting the update process, it is mandatory to update the CLI to the new version:

```
$ dcos package install kubernetes --cli --package-version=1.1.1-1.10.4
Copy
DC/OS Enterprise Edition
```

Below is how the user starts the package version update:

```
$ dcos kubernetes update --package-version=1.1.1-1.10.4
About to start an update from version <CURRENT_VERSION> to <NEW_VERSION>

Updating these components means the Kubernetes cluster may experience some
downtime or, in the worst-case scenario, cease to function properly.
Before updating proceed cautiously and always backup your data.

This operation is long-running and has to run to completion.
Are you sure you want to continue? [yes/no] yes

2018/03/01 15:40:14 starting update process...
2018/03/01 15:40:15 waiting for update to finish...
2018/03/01 15:41:56 update complete!
```
