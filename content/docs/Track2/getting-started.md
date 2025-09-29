---
title: Getting Started with Track 2
---

# Overview

**This document will include challenge specific information and is useful to read even if you have prior experience with Kubernetes.**

Track 2 teams will be given access to a shared kubernetes compute cluster. This cluster makes available 6 nodes, 4 of which with 8 x H200 GPUs, and 2 of which with 8 x H100 GPUs. The cluster is to be used for training and tuning your models. 

Note that the cluster can only be interacted with through your team's VM. So, **all commands given should be run on your team's VM**. 

# Crash Course Kubernetes

In Kubernetes, state is expressed in the form of resources. Some common resource types are pods (like containers in docker), deployments (similar to a docker-compose file), and jobs. 

You can perform actions on resources using the `kubectl` utility. For instance,
```
# To list pods running:
kubectl -n eidf219ns get pods
# To create a resource from a file:
kubectl -n eidf219ns create -f myjob.yaml
# To see the logs emitted from a pod (pods are spawned by jobs)
kubectl -n eidf219ns logs <pod_name>
# To delete a broken or finished job:
kubectl -n eidf219ns delete job <job_name>
```

For this challenge, Job resources are the most relevent.

## Running your first job

Jobs can be specified with yaml files. This makes them reproducible and much easier to debug. To actually create a resource specified by a yaml file, you can use the command `kubectl -n eidf219ns create -f /path/to/file.yaml`. 

**When creating jobs, please prepend your team name to the job name. Failure to do so may result in your jobs being deleted.**

As an example:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
 generateName: [team-name]-jobtest-
 labels:
  kueue.x-k8s.io/queue-name:  eidf219ns-user-queue
spec:
 completions: 1
 backoffLimit: 1
 ttlSecondsAfterFinished: 1800
 template:
  metadata:
   name: job-test
  spec:
   containers:
   - name: cudasample
     image: nvcr.io/nvidia/k8s/cuda-sample:nbody-cuda11.7.1
     args: ["-benchmark", "-numbodies=512000", "-fp64", "-fullscreen"]
     resources:
      requests:
       cpu: 2
       memory: '1Gi'
      limits:
       cpu: 2
       memory: '4Gi'
       nvidia.com/gpu: 1
   restartPolicy: Never
```

## Claiming GPUs 

The following will attempt to claim a GPU and then display PCIE devices in the logs.
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  generateName: [team-name]-gpu-test-1-
  labels:
    kueue.x-k8s.io/queue-name: eidf219ns-user-queue
    app: gputest1
spec:
  # Selecter would go here
  template:  
    metadata:
      labels:
        app: gputest1
    spec:
      restartPolicy: Never
      containers:
        - name: ubuntu
          image: ubuntu:20.04
          command: ["/bin/bash", "-c", "apt-get update && apt-get install lshw -y && lshw -C display"]
          resources:
            limits:
              nvidia.com/gpu: 1
              cpu: 1
              memory: "4Gi"
```

By default, it will pick a GPU variant and attempt to select specifically nodes with that GPU, so if you see that your Job is marked as pending for long periods, it may be worth trying to select other types of GPUs. This can be done adding the following lines where it says "Selecter would go here" in the code sample above:
```yaml
nodeSelector:
    nvidia.com/gpu-product: '<GPU Type>'
```
See the [EIDF Doc's](https://docs.eidf.ac.uk/services/gpuservice/training/L1_getting_started/#specifying-gpu-requirements) for more information.

# FAQ / Common Issues

## Setting up my default namespace

If you'd like to avoid having to type `-n eidf219ns` every time, you can follow the [instructions here](https://docs.eidf.ac.uk/services/gpuservice/training/L1_getting_started/#change-the-default-kubectl-namespace-in-the-project-kubeconfig-file) to setup your default namespace.

Doing so allows you to simply type `kubectl get pods` for instance to list all pods, rather than `kubectl -n eidf219ns get pods`

## Persistent Volume Claims

If you'd like a PVC created for your team, please message @emily.747 on Discord or [email me](mailto:techsec@comp-soc.com).

# Further Documentation

I'd highly recommend the following resources and documentation when you're stuck:
- [EIDF's GPU Service Documentation](https://docs.eidf.ac.uk/services/gpuservice/)
- [Kubernetes Docs](https://kubernetes.io/docs/home/)

