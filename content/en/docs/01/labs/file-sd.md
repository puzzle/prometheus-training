---
title: "1.1 Tasks: File-Based Service Discovery"
weight: 1
onlyWhen: baloise

sectionnumber: 1
---

In this first lab you are going to configure Prometheus to scrape the OpenShift-external targets by using file-based service discovery.

### Task {{% param sectionnumber %}}.1: Identify your monitoring repository

Before we get started, take the time to familiarize yourself with the config repository of your team - it should already be available as described in [Deploying the Baloise Monitoring Stack](https://confluence.baloisenet.com/atlassian/display/BALMATE/01+-+Deploying+the+Baloise+Monitoring+Stack).

The working directory for this training is the folder in your team's config repository with the `-monitoring` suffix.

{{% alert title="Note" color="warning" %}}
Please name all files created in this training with the filename prefix `training-`. This naming pattern will help in cleaning up all related files after training completion.
{{% /alert %}}

### Task {{% param sectionnumber %}}.2: Create static targets

We are going to use the file-based service discovery mechanism that has been deployed on OpenShift. As file input you will create a ConfigMap defining the static targets.

In the monitoring folder within you repository, create a YAML file defining a ConfigMap and add the file to your repository. You can take the below example as inspiration.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mylinuxvms # provide a name
  labels:
    monitoring: external # provide label to match monitoring procedure
data:
  auth-mylinuxvms.yaml: | # provide an unique file name (filename starting  with auth_* will use default credentials | filename starting with nonauth_ will not use authentication)
    - targets: # provide targets
        - myhost1.balgroupit.com:9100 # path defaults to /metrics
      labels: # provide additional labels (optional)
        cmdbName: ServerLinux
        location: OMO
        purpose: VM
```

In our example we added the host `myhost1.balgroupit.com` with an exporter running on port 9100 as static target. We also added custom labels to help us identify our metrics.

### Task {{% param sectionnumber %}}.2: Verify

As soon as the ConfigMap has been synchronized by ArgoCD, your defined targets should appear in Prometheus in the "Status -> Targets" submenu.

FIXME: Prometheus URL

![Prometheus UI - Target Down](../target-down.png)

As you can see, the target is down and cannot be scraped by Prometheus. The reason is provided in the error message: "Get "https://myhost1.balgroupit.com:9100/metrics": dial tcp: lookup myhost1.balgroupit.com on 172.24.0.10:53: no such host"

In our example we used a non-existing host `myhost1.balgroupit.com`. To fix this, use an existing host as your target.
