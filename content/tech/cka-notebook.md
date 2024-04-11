---
title: "Certified Kubernetes Administrator (CKA) Notebook"
date: 2024-04-10
---

### how to calculate how many lines are returned in linux?

```bash
<command> | wc -l
```

### how to create a pod with a specific image?

> for example: `nginx`

```bash
# kubectl run <podName> --image=<imageName>
kubectl run nginx --image=nginx
```

> what's its default port in this case?

### how to see the details of a pod?

> Such as `containerStatuses/image` or `nodeName`

```bash
# kubectl describe pod <podName>
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/8247835a-03e5-41c5-5f93-63720b355b77.png)

### how to grab a property out of a output in linux command?

```bash
# <command> | grep <propertyName>
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/235401d9-3e9d-cfcc-59f2-13e7b581d551.png)

### what does `-o wide` do?

> when running `kubectl get pods -o wide`

Will tell you additonal information: `IP` and `Node`

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/f9efa687-91e1-7971-a131-6add11d1e134.png)

### how to delete a pod?

```bash
# kubectl delete pod <podName>
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/274a1b3a-21f6-5524-d861-993e8fee9a59.png)

### how to create a yaml file?

We use `kubectl run` command with `--dry-run=client -o yaml` option to create a manifest file:

```bash
kubectl run redis --image=redis123 --dry-run=client -o yaml > redis-definition.yaml
```

After that, using `kubectl create -f` command to create a resource from the manifest file:

```bash
kubectl create -f redis-definition.yaml
```

Verify the work by running `kubectl get` command:

```bash
kubectl get pods
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/8ec955bb-f5b4-aa85-7b3e-b605e2e70068.png)

### how to update a pod?

Use the `kubectl edit` command to update the pod.

```bash
kubectl edit pod redis
```

Alternatively, use `vi` or `nano` to update the yaml file, then run `kubectl apply` to apply the change.

```bash
kubectl apply -f redis-definition.yaml
```
