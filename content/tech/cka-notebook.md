---
title: "Certified Kubernetes Administrator (CKA) Notebook"
date: 2024-04-10
---

# Core Concepts

## Core Concepts - Kubernetes Concepts

- Pod

  - Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

  - A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers.

- ReplicaSet
  - A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

---

## Core Concepts - Practice Test - Pods

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

`--dry-run`: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the `--dry-run=client` option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.
`-o yaml`: This will output the resource definition in YAML format on screen.

Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.

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

---

## Core Concepts - Practice Test - ReplicaSets

### how to return all the ReplicaSets?

```bash
kubectl get replicaset
# or
kubectl get rs
```

### how to get the details of a ReplicaSet?

```bash
# kubectl describe replicaset <ReplicaSetName>
kubectl describe replicaset new-replica-set
```

### common error in a ReplicaSet

- the `apiVersion` must include `apps/v1`
- the `tier` of `matchLabels` under `selector` must match exactly as the one under `labels`
- pods won't update even the replicaset has been udpated, you have to either re-create the rs or delete all the pods so they will be created
- scale down rs to 0 and then scale up to the number of replicaset you want will also update all the pods

```yaml
apiVersion: apps/v1 # fixed
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: nginx # match
  template:
    metadata:
      labels:
        tier: nginx # match
    spec:
      containers:
        - name: nginx
          image: nginx
```

### how to scale a ReplicaSet?

```bash
kubectl scale rs new-replica-set --replicas 5
```

### how to delete a ReplicaSet?

```bash
# kubectl delete rs <ReplicaSetName>
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/0d13f097-c72e-5838-d6cc-b385731d82f9.png)

---

## Core Concepts - Practice Test - Deployment

### how to create a Deployment with 4 ReplicaSets?

```bash
kubectl create deployment deploymentName --image=nginx --replicas=4 --dry-run=client -o yaml > randomFileName.yaml
```

---

## Core Concepts - Practice Test - Namespaces

### how to get the exact number of namespaces?

```bash
kubectl get ns --no-headers | wc -l
```

### how to create a pod in a specific namespace?

> for example: `finance` namespace

```bash
kubectl run redis --image=redis -n finance
```

### Which namespace has the blue pod in it?

```bash
kubectl get pods --all-namespaces | grep blue
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/8ca89d16-b7ff-ce28-7639-65b2eb324a73.png)

### how to access pod on another namespace?

```bash
# <service-name>.<namespace-name>.svc.cluster.local
```

---

## Core Concepts - Practice Test - Services

### how to get all the service?

```bash
kubectl get service
```

### how to see the details of a service?

```bash
kubectl describe service
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/61fd8945-7f8f-47b4-d4e8-deb7948681b5.png)

### how to create a service using a template file?

```yaml
# service-definition-1.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: default
spec:
  ports:
    - nodePort: 30080
      port: 8080
      targetPort: 8080
  selector:
    name: simple-webapp
  type: NodePort
```

```bash
kubectl apply -f /root/service-definition-1.yaml
```

---

## Core Concepts - Practice Test - Imperative Commands

### Deploy a pod named `nginx-pod` using the `nginx:alpine` image.

```bash
kubectl run nginx-pod --image=nginx:alpine
```

### Deploy a redis pod using the `redis:alpine` image with the labels set to `tier=db`.

- Use the imperative command:

```bash
kubectl run redis -l tier=db --image=redis:alpine
```

- Or, run the command to generate the definition file:

```bash
kubectl run redis --image=redis:alpine --dry-run=client -oyaml > redis-pod.yaml
```

Add given labels `tier=db` under the `metadata` section.

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    tier: db
  name: redis
spec:
  containers:
    - image: redis:alpine
      name: redis
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

Then run the command: `kubectl create -f redis-pod.yaml` to create the pod from the definition file.

### expose a pod through a service

```bash
kubectl expose pod redis --port=6379 --name redis-service
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/ce7a3e7c-5c4f-4b8a-e959-621a168c0480.png)

### create a deployment named `webapp` using the image `kodekloud/webapp-color` with `3` replicas.

```bash
kubectl create deployment  webapp --image=kodekloud/webapp-color --replicas=3
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/ad29c750-dadb-64e3-80ae-e143f943cc09.png)

### Create a new pod called `custom-nginx` using the `nginx` image and expose it on container port `8080`.

```bash
kubectl run custom-nginx --image=nginx --port=8080
```

Note, this is not the same as creating the pod and expose it in two seperate steps. (which only expose on the service level, not on the container level.)

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/84e19ee8-d2a8-f15f-0faf-fa1ab0883b36.png)

### Create a new deployment called `redis-deploy` in the `dev-ns` namespace with the `redis` image. It should have `2` replicas.

```bash
kubectl create deployment redis-deploy --image=redis --replicas=2 -n dev-ns
```

### Create a pod called `httpd` using the image `httpd:alpine` in the default namespace. Next, create a service of type `ClusterIP` by the same name `httpd`. The target port for the service should be `80`.

```bash
kubectl run httpd --image=httpd:alpine --port=80 --expose
```

This equals the following two steps:

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/f5ad0328-30c4-a2b5-8079-1034ba563a50.png)

---

# Scheduling

## Scheduling - Practice Test - Manual Scheduling

### Create a pod using a given file `nginx.yaml`.

```yaml
# nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
```

```bash
kubectl create -f nginx.yaml
```

### Manually schedule the pod on `node01`.

Delete the existing pod first. Run the below command:

```bash
kubectl delete pod nginx
```

To list and know the names of available nodes on the cluster:

```bash
kubectl get nodes
```

Add the nodeName field under the spec section in the nginx.yaml file with node01 as the value:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: node01
  containers:
    - image: nginx
      name: nginx
```

Then run the command `kubectl create -f nginx.yaml` to create a pod from the definition file.

To check the status of a `nginx` pod and to know the node name:

```bash
kubectl get pods -o wide
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/799ceb89-a782-b601-eb0a-17f39eb3ebd5.png)

---

## Scheduling - Practice Test - Labels and Selectors

### How to get all the pods with label `env=dev`?

```bash
kubectl get pods --selector env=dev
```

### How many pods have label `env=dev`?

```bash
kubectl get pods --selector env=dev --no-headers | wc -l
```

### How to get the that belongs to the prod environment, the finance BU, and frontend tier? (multi-selector)

```bash
kubectl get all --selector env=prod,bu=finance,tier=frontend
```

---

## Scheduling - Practice Test - Taints and Tolerations

### Check taints on a node `node01`.

```bash
kubectl describe node node01 | grep -i taints
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/258b2541-e916-2347-0699-4b6203cfe050.png)

### Create a taint on `node01` with key of `spray`, value of `mortein` and effect of `NoSchedule`

```bash
kubectl taint nodes node01 spray=mortein:NoSchedule
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/530f4c4a-680a-0df0-9608-7f6aa5b21dcf.png)

### Create a pod named `bee` with the `nginx` image, which has a toleration set to the taint `mortein`.

Solution manifest file to create a pod called bee as follows:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bee
spec:
  containers:
    - image: nginx
      name: bee
  tolerations:
    - key: spray
      value: mortein
      effect: NoSchedule
      operator: Equal
```

then run the kubectl create -f <FILE-NAME>.yaml to create a pod.
