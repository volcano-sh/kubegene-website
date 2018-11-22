+++
title = "Execution Deletion"
description = "This page show how execution is deleted."
weight = 10
toc = true
aliases = ["/docs/guides/"]
[menu.main]
  parent = "Guides"
  weight = 5
+++

## Understanding cascading deletion

When kube-dag schedule an execution, it will set the `metadata.ownerReference` to all jobs to specify relationships between owners and dependents. And kubernetes sets the value of `ownerReference` automatically for pod of a job.

So let's check one job of previously created execution `test-exec`.

```shell
kubectl get job test-exec.a.0 --output=yaml
```

The output shows that the job owner is a Execution named `test-exec`.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  ...
  ownerReferences:
  - apiVersion: execution.kubegene.io/v1alpha1
    blockOwnerDeletion: true
    controller: true
    kind: Execution
    name: test-exec
    uid: 5f144885-d04e-11e8-ad51-286ed488dc10
  ...

```

When want to clean up an execution, you only need to make one all to delete Execution(no need explicitly delete its dependents), the dependents will be deleted automatically by kubernetes. This is done by kubernetes [Garbage Collection](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/).


**Note:** The ownerReference is used to control the relationship between owner and dependents, do not update it unless you know what you do. Also donot delete/update the Execution dependents, especially when the execution is in `Running` phase.
