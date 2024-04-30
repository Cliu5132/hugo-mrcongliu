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

### How to taint a node and remove that taint

```bash
ubectl taint nodes node1 key1=value1:NoSchedule
```

```bash
kubectl taint nodes node1 key1=value1:NoSchedule-
```

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

then run the `kubectl create -f <FILE-NAME>.yaml` to create a pod.

### Remove the taint on `controlplane`, which currently has the taint effect of `NoSchedule`.

```bash
kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/c3f94d39-9fba-6245-c750-893ae3b46c8e.png)

---

## Scheduling - Practice Test - Node Affinity

### Apply a label `color=blue` to node `node01`

```bash
kubectl label node node01 color=blue
```

### Check if `controlplane` and `node01` have any taints on them that will prevent the pods to be scheduled on them.

```bash
kubectl describe node controlplane | grep -i taints

kubectl describe node node01 | grep -i taints
```

### Show taints on all nodes (alternative to the previous)

```bash
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints --no-headers
```

### Set Node Affinity to the deployment to place the pods on `node01` only.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
        - image: nginx
          imagePullPolicy: Always
          name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: color
                    operator: In
                    values:
                      - blue
```

### Create a new deployment named `red` with the `nginx` image and `2` replicas, and ensure it gets placed on the `controlplane` node only.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
        - image: nginx
          imagePullPolicy: Always
          name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: node-role.kubernetes.io/control-plane
                    operator: Exists
```

---

## Scheduling - Practice Test - Resource Limits

### Inspect `elephant` pod and identify the Reason why it is not running.

The reason for the pod not running is OOMKilled. This means that the pod ran out of memory.

```bash
root@controlplane:~# kubectl describe pod elephant  | grep -A5 State:
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    1
      Started:      Sun, 25 Apr 2021 15:13:07 +0000
      Finished:     Sun, 25 Apr 2021 15:13:07 +0000
    Ready:          False
root@controlplane:~#
```

The `grep -A5 State`: command is used to filter the output to show only the lines containing "State:", as well as the 5 lines that come after it. So, `A5` here means "after 5 lines." It's a command-line argument for the `grep` command to display additional context around the matching lines.

It's changing the status frequently so make use of the watch command to get the output every two seconds:

```bash
root@controlplane ~ ➜  watch kubectl get pods
```

The status `OOMKilled` indicates that it is failing because the pod ran out of memory. Identify the memory limit set on the POD.

```bash
controlplane ~ ✖ kubectl describe pod elephant  | grep -A5 Limits:
    Limits:
      memory:  10Mi
    Requests:
      memory:     5Mi
    Environment:  <none>
    Mounts:
```

### The `elephant` pod runs a process that consumes 15Mi of memory. Increase the limit of the `elephant` pod to 20Mi.

Create the file `elephant.yaml` by running command `kubectl get po elephant -o yaml > elephant.yaml` and edit the file such as memory limit is set to `20Mi` as follows:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: elephant
  namespace: default
spec:
  containers:
    - args:
        - --vm
        - "1"
        - --vm-bytes
        - 15M
        - --vm-hang
        - "1"
      command:
        - stress
      image: polinux/stress
      name: mem-stress
      resources:
        limits:
          memory: 20Mi
        requests:
          memory: 5Mi
```

then run `kubectl replace -f elephant.yaml --force`. This command will delete the existing one first and recreate a new one from the YAML file.

---

## Scheduling - Practice Test - DaemonSets

### How many `DaemonSets` are created in the cluster in all namespaces?

```bash
controlplane ~ ➜  kubectl get daemonsets --all-namespaces
NAMESPACE      NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   kube-flannel-ds   1         1         1       1            1           <none>                   104s
kube-system    kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   105s
```

### Identify all resources and their types

```bash
controlplane ~ ➜  kubectl get all --all-namespaces
NAMESPACE      NAME                                       READY   STATUS    RESTARTS   AGE
kube-flannel   pod/kube-flannel-ds-kqcw7                  1/1     Running   0          4m40s
kube-system    pod/coredns-69f9c977-cwbq2                 1/1     Running   0          4m39s
kube-system    pod/coredns-69f9c977-hv69g                 1/1     Running   0          4m39s
kube-system    pod/etcd-controlplane                      1/1     Running   0          4m52s
kube-system    pod/kube-apiserver-controlplane            1/1     Running   0          4m52s
kube-system    pod/kube-controller-manager-controlplane   1/1     Running   0          4m52s
kube-system    pod/kube-proxy-27wdh                       1/1     Running   0          4m40s
kube-system    pod/kube-scheduler-controlplane            1/1     Running   0          4m52s

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  4m55s
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   4m51s

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   4m52s
kube-system    daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   4m53s

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2/2     2            2           4m51s

NAMESPACE     NAME                               DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-69f9c977   2         2         2       4m40s
```

---

## Scheduling - Practice Test - Static PODs

### How many static pods exist in this cluster in all namespaces?

Run the command `kubectl get pods --all-namespaces` and look for those with `-controlplane` appended in the name

```bash
controlplane ~ ➜  kubectl get pods --all-namespaces
NAMESPACE      NAME                                   READY   STATUS    RESTARTS   AGE
kube-flannel   kube-flannel-ds-ccn9w                  1/1     Running   0          20m
kube-flannel   kube-flannel-ds-kxsnz                  1/1     Running   0          20m
kube-system    coredns-69f9c977-gswx4                 1/1     Running   0          20m
kube-system    coredns-69f9c977-r9m6q                 1/1     Running   0          20m
kube-system    etcd-controlplane                      1/1     Running   0          20m
kube-system    kube-apiserver-controlplane            1/1     Running   0          21m
kube-system    kube-controller-manager-controlplane   1/1     Running   0          20m
kube-system    kube-proxy-fd8h8                       1/1     Running   0          20m
kube-system    kube-proxy-vshjj                       1/1     Running   0          20m
kube-system    kube-scheduler-controlplane            1/1     Running   0          20m

controlplane ~ ➜  kubectl get pods -A
NAMESPACE      NAME                                   READY   STATUS    RESTARTS   AGE
kube-flannel   kube-flannel-ds-ccn9w                  1/1     Running   0          20m
kube-flannel   kube-flannel-ds-kxsnz                  1/1     Running   0          21m
kube-system    coredns-69f9c977-gswx4                 1/1     Running   0          21m
kube-system    coredns-69f9c977-r9m6q                 1/1     Running   0          21m
kube-system    etcd-controlplane                      1/1     Running   0          21m
kube-system    kube-apiserver-controlplane            1/1     Running   0          21m
kube-system    kube-controller-manager-controlplane   1/1     Running   0          21m
kube-system    kube-proxy-fd8h8                       1/1     Running   0          21m
kube-system    kube-proxy-vshjj                       1/1     Running   0          20m
kube-system    kube-scheduler-controlplane            1/1     Running   0          21m
```

- The `coredns` pods are created as part of the `coredns` deployment and hence, it is not a static pod.
- `kube-proxy` is deployed as a `DaemonSet` and hence, it is not a staic pod.
- By default, `static pods` are created for the controlplane components and hence, they are only created in the `controlplane` node.

### What is the path of the directory holding the static pod definition files?

First idenity the kubelet config file:

```bash
root@controlplane:~# ps -aux | grep /usr/bin/kubelet
root      3668  0.0  1.5 1933476 63076 ?       Ssl  Mar13  16:18 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2
root      4879  0.0  0.0  11468  1040 pts/0    S+   00:06   0:00 grep --color=auto /usr/bin/kubelet
root@controlplane:~#
```

From the output we can see that the kubelet config file used is `/var/lib/kubelet/config.yaml`.
Next, lookup the value assigned for `staticPodPath`:

```bash
root@controlplane:~# grep -i staticpod /var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests
root@controlplane:~#
```

As you can see, the path configured is the `/etc/kubernetes/manifests` directory.

---

## Scheduling - Practice Test - Multiple Schedulers

### What is the name of the POD that deploys the default kubernetes scheduler?

```bash
controlplane ~ ➜  kubectl get pods --namespace=kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-69f9c977-cwp8j                 1/1     Running   0          8m3s
coredns-69f9c977-lq49c                 1/1     Running   0          8m3s
etcd-controlplane                      1/1     Running   0          8m19s
kube-apiserver-controlplane            1/1     Running   0          8m16s
kube-controller-manager-controlplane   1/1     Running   0          8m16s
kube-proxy-lfwzw                       1/1     Running   0          8m3s
kube-scheduler-controlplane            1/1     Running   0          8m19s

controlplane ~ ➜
```

### create a POD with the new custom scheduler.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  schedulerName: my-scheduler # add
  containers:
    - image: nginx
      name: nginx
```

---

# Loggin and Monitoring

## Loggin and Monitoring - Practice Test - Monitor Cluster Components

### Download `metrics-server` from GitHub.

```bash
controlplane ~ ➜  git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
Cloning into 'kubernetes-metrics-server'...
remote: Enumerating objects: 31, done.
remote: Counting objects: 100% (19/19), done.
remote: Compressing objects: 100% (19/19), done.
remote: Total 31 (delta 8), reused 0 (delta 0), pack-reused 12
Receiving objects: 100% (31/31), 8.08 KiB | 8.08 MiB/s, done.
Resolving deltas: 100% (10/10), done.
```

### Deploy `metrics-server` by creating all the components downloaded.

```bash
controlplane ~ ➜  pwd
/root

controlplane ~ ➜  cd kubernetes-metrics-server/

controlplane kubernetes-metrics-server on  master ➜  ls
aggregated-metrics-reader.yaml  metrics-apiservice.yaml         README.md
auth-delegator.yaml             metrics-server-deployment.yaml  resource-reader.yaml
auth-reader.yaml                metrics-server-service.yaml

controlplane kubernetes-metrics-server on  master ➜  kubectl create -f .
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created

controlplane kubernetes-metrics-server on  master ➜
```

### Display resource (CPU/memory) usage of nodes.

```bash
controlplane kubernetes-metrics-server on  master ➜  kubectl top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
controlplane   258m         0%     1096Mi          0%
node01         24m          0%     275Mi           0%

controlplane kubernetes-metrics-server on  master ➜
```

### Identify the node that consumes the `most` CPU(cores).

```bash
controlplane kubernetes-metrics-server on  master ➜  kubectl top node --sort-by='cpu' --no-headers | head -1
controlplane   265m   0%    1099Mi   0%

controlplane kubernetes-metrics-server on  master ➜
```

Here we have used `head -1` command to print the pod first in the order, which is the one that uses the most CPU(cores).

---

## Loggin and Monitoring - Practice Test - Managing Application Logs

### Identify the log recorded for USER5 accessing the pod `webapp-1`.

```bash
controlplane ~ ➜  kubectl logs webapp-1 | grep USER5
[2024-04-17 12:32:52,976] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2024-04-17 12:32:57,981] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
```

---

# Application Lifecycle Management

## Application Lifecycle Management - Practice Test - Rolling Updates and Rollbacks

### Inspect the deployment `frontend` and identify the number of PODs deployed by it.

Run the command `kubectl describe deployment` and look at `Desired Replicas`

```bash
controlplane ~ ➜  kubectl describe deployment frontend
Name:                   frontend
Namespace:              default
CreationTimestamp:      Wed, 17 Apr 2024 12:44:41 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=webapp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable # Desired Replicas
StrategyType:           RollingUpdate # Strategy Type
MinReadySeconds:        20
RollingUpdateStrategy:  25% max unavailable, 25% max surge # Max Unavailable Value
Pod Template:
  Labels:  name=webapp
  Containers:
   simple-webapp:
    Image:        kodekloud/webapp-color:v1 # Image
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   frontend-685dfcc44 (4/4 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  13m   deployment-controller  Scaled up replica set frontend-685dfcc44 to 4

controlplane ~ ➜
```

### Create a deployment with a strategy of `Recreate`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      name: webapp
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        name: webapp
    spec:
      containers:
        - image: kodekloud/webapp-color:v2
          name: simple-webapp
          ports:
            - containerPort: 8080
              protocol: TCP
```

---

## Application Lifecycle Management - Practice Test - Commands and Arguments

### Create a pod with the `ubuntu` image to run a container to sleep for 5000 seconds.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-2
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command:
        - "sleep"
        - "5000"
```

### How can pod's command override Dockerfile's?

Assume the image was created from the Dockerfile in this directory.
The `ENTRYPOINT` in the `Dockerfile` is overridden by the `command` in the pod definition file.
The command that will be run is just `--color green`.

```bash
controlplane ~/webapp-color-2 ➜  cat Dockerfile
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]

controlplane ~/webapp-color-2 ➜  cat webapp-color-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
  labels:
      name: webapp-green
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["--color","green"]
```

Using the `command` and `args` in the pod definition file.
Note, `args` will override the `CMD` inside Dockerfile, `command` will override `ENTRYPOINT` IN Dockerfile.
Below, the command that will run is `python app.py --color pink`.

```bash
controlplane ~/webapp-color-3 ➜  cat Dockerfile
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]

controlplane ~/webapp-color-3 ➜  cat webapp-color-pod-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
  labels:
      name: webapp-green
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["python", "app.py"]
    args: ["--color", "pink"]
```

---

## Application Lifecycle Management - Practice Test - Env Variables

### How to pass in env var `APP_COLOR=green` into pod `webapp-color`?

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
    - env:
        - name: APP_COLOR
          value: green
      image: kodekloud/webapp-color
      name: webapp-color
```

### How to get all configMaps?

```bash
kubectl get configmaps
```

### Create a ConfigMap `webapp-config-map`, using pod `webapp-color`, data `APP_COLOR=darkblue` and `APP_OTHER=disregard`.

```bash
kubectl create configmap  webapp-config-map --from-literal=APP_COLOR=darkblue --from-literal=APP_OTHER=disregard
```

Update Pod `webapp-color`

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
    - env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: webapp-config-map # Config Map Name
              key: APP_COLOR # key
      image: kodekloud/webapp-color
      name: webapp-color
```

---

## Application Lifecycle Management - Practice Test - Secrets

### How to get all secrets?

```bash
controlplane ~ ➜  k get secrets
NAME              TYPE                                  DATA   AGE
dashboard-token   kubernetes.io/service-account-token   3      73s
```

### How to get the details of a secret?

```bash
controlplane ~ ➜  k describe secret dashboard-token
Name:         dashboard-token
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-sa
              kubernetes.io/service-account.uid: b8bdfa74-fbf3-4772-82f9-d58b5f64b2dc

Type:  kubernetes.io/service-account-token # type

Data
====
ca.crt:     570 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkFnWFdoaUQ4aW1vbkhrME12RmMxZHlzMVB4bGN1SXQ1RmItN2wzd2pBY0EifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRhc2hib2FyZC10b2tlbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkYXNoYm9hcmQtc2EiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJiOGJkZmE3NC1mYmYzLTQ3NzItODJmOS1kNThiNWY2NGIyZGMiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQtc2EifQ.pS1kDefKVp3K1aR_l-j0iXbIlRZsSXrqXG0alWmkG4ZvlTN9j3l9pxYY9i_AIFWoaMVdSW9uubJaCeRCpUAjm4QyQxM0yukJtWqpvMtfQI0dYiJt927QBPWKqQ8nKvU9IjvRFc4n8dZR4dVBAjv7ZhCRI2uiJdc24JtB9LzIqaUMGiHmNJkNzjsrNJoUP-G582j6IyG-MFDtFEBfqIkXuS4HlupSoINskyEFfz4lkRuCEASCkVVFt-b_i3yFj50l-WBlx1RmygAdS7ffQzaTZWDt0tJdhHsDlScSKHHg8TR7Wnv-GEr3PTLtNNOBLOSmswxD6wdDM59Z6X1E9W-vGQ

controlplane ~ ➜
```

### Deploy an application with the below architecture

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/37d08679-b2b5-6c62-8864-4c3dce18f963.png)

### Create a new secret named `db-secret` with the data given below.

- Secret Name: `db-secret`

- Secret 1: `DB_Host=sql01`

- Secret 2: `DB_User=root`

- Secret 3: `DB_Password=password123`

```bash
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
```

### Configure `webapp-pod` to load environment variables from the newly created secret.

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-pod
  name: webapp-pod
  namespace: default
spec:
  containers:
    - image: kodekloud/simple-webapp-mysql
      imagePullPolicy: Always
      name: webapp
      envFrom:
        - secretRef:
            name: db-secret # here
```

---

## Application Lifecycle Management - Practice Test - Multi Container PODs

### Create a multi-container pod with 2 containers.

Solution manifest file to create a multi-container pod called `yellow` as follows:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: yellow
spec:
  containers:
    - name: lemon
      image: busybox
      command:
        - sleep
        - "1000"

    - name: gold
      image: redis
```

### Case Study - ELK Stack

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/9811ec8f-83e1-3057-f65d-bde2cdeafe78.png)

We have deployed an application logging stack in the `elastic-stack` namespace.

```bash
controlplane ~ ➜  kubectl get pod -n elastic-stack
NAME             READY   STATUS    RESTARTS   AGE
app              1/1     Running   0          23m
elastic-search   1/1     Running   0          23m
kibana           1/1     Running   0          23m

controlplane ~ ➜
```

You can inspect the Kibana logs by running:

```bash
kubectl -n elastic-stack logs kibana
```

Exec in to the container and inspect logs stored in `/log/app.log`

```bash
kubectl -n elastic-stack exec -it app -- cat /log/app.log
```

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/7a0b3606-6f21-9044-8845-ba706ff59a12.png)

Edit the pod in the `elastic-stack` namespace to add a sidecar container to send logs to Elastic Search. Mount the log volume to the sidecar container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
  namespace: elastic-stack
  labels:
    name: app
spec:
  containers:
    - name: app
      image: kodekloud/event-simulator
      volumeMounts:
        - mountPath: /log
          name: log-volume

    - name: sidecar
      image: kodekloud/filebeat-configured
      volumeMounts:
        - mountPath: /var/log/event-simulator/
          name: log-volume

  volumes:
    - name: log-volume
      hostPath:
        # directory location on host
        path: /var/log/webapp
        # this field is optional
        type: DirectoryOrCreate
```

Inspect the Kibana UI. You should now see logs appearing in the Discover section.

You might have to wait for a couple of minutes for the logs to populate. You might have to create an index pattern to list the logs. If not sure check this video: https://bit.ly/2EXYdHf

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/100bd628-7fd5-05c5-9ea5-b1a82568d822.png)

---

## Application Lifecycle Management - Practice Test - Init Containers

> Init containers: specialized containers that run before app containers in a Pod.

### How to add `initContainers` to a pod?

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: red
  namespace: default
spec:
  containers:
    - command:
        - sh
        - -c
        - echo The app is running! && sleep 3600
      image: busybox:1.28
      name: red-container
  initContainers:
    - image: busybox
      name: red-initcontainer
      command:
        - "sleep"
        - "20"
```

{{< notice note >}}
In Kubernetes manifest files like **YAML**, strings that contain special characters (such as spaces) need to be enclosed in double quotes to be interpreted correctly. In the first command example, where the command is specified as a list of strings, each element is treated as an **individual string** and doesn't require double quotes. In the second command example, where the command is specified as a list of strings but includes a command with a **special character** ("sleep"), each string element needs to be enclosed in double quotes to ensure proper interpretation by Kubernetes.
{{< /notice >}}

---

# Cluster Maintenance

## Cluster Maintenance - Practice Test - OS Upgrades

### We need to take `node01` out for maintenance. Empty the node of all applications and mark it unschedulable.

```bash
kubectl drain node01 --ignore-daemonsets
```

When you use `kubectl drain` with the `--ignore-daemonsets` flag on a node, you're essentially telling Kubernetes to evict all the pods running on that node, excluding the ones that are part of a DaemonSet.

DaemonSets are a type of controller in Kubernetes that ensure that a specific pod runs on all (or some) nodes in a cluster. They're often used for system daemons like monitoring agents or networking components that need to be present on every node.

So, by using `kubectl drain node01 --ignore-daemonsets`, you're ensuring that all pods on `node01` are gracefully evicted, but the DaemonSet pods will remain running, ensuring that essential system services continue to operate across the cluster. This command is often used when you need to perform maintenance or decommission a node without disrupting critical system components.

```bash
controlplane ~ ➜  kubectl drain node01 --ignore-daemonsets
node/node01 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-f4dqq, kube-system/kube-proxy-snkqt
evicting pod default/blue-667bf6b9f9-rhtmv
evicting pod default/blue-667bf6b9f9-6sxqp
pod/blue-667bf6b9f9-rhtmv evicted
pod/blue-667bf6b9f9-6sxqp evicted
node/node01 drained

controlplane ~ ➜
```

In this case, all pods on `node01` will be moved to `controlplane`.

### Why?

```bash
root@controlplane:~# kubectl describe node controlplane | grep -i  taint
Taints:             <none>
root@controlplane:~#
```

Since there are no taints on the controlplane node, all the pods were started on it when we ran the kubectl drain node01 command.

### The maintenance tasks have been completed. Configure the node `node01` to be schedulable again.

```bash
kubectl uncordon node01
```

```bash
controlplane ~ ➜  kubectl uncordon node01
node/node01 uncordoned

controlplane ~ ➜
```

At this point, there are no pods on `node01`.

Running the `uncordon` command on a node will not automatically schedule pods on the node. When new pods are created, they will be placed on `node01`.

### To drain a node containing a pod which is not part of a replicaset (losing it)

```bash
controlplane ~ ➜  kubectl drain node01 --ignore-daemonsets
node/node01 cordoned
error: unable to drain node "node01" due to error:cannot delete Pods declare no controller (use --force to override): default/hr-app, continuing command...
There are pending nodes to be drained:
 node01
cannot delete Pods declare no controller (use --force to override): default/hr-app
```

Run: `kubectl get pods -o wide` and you will see that there is a single pod scheduled on node01 which is not part of a replicaset.

The drain command will not work in this case. To forcefully drain the node we now have to use the `--force` flag.

```bash
kubectl drain node01 --ignore-daemonsets --force
```

{{< notice note >}}
A forceful drain of the node will delete any pod that is not part of a replicaset.
{{< /notice >}}

### To drain a node containing a pod which is not part of a replicaset (keeping it)

`hr-app` is a critical app and we do not want it to be removed and we do not want to schedule any more pods on `node01`.
Mark `node01` as `unschedulable` so that no new pods are scheduled on this node.

```bash
kubectl cordon node01
```

Do not drain `node01`, instead use the `kubectl cordon node01` command. This will ensure that no new pods are scheduled on this node and the existing pods will not be affected by this operation.

## Cluster Maintenance - Practice Test - Cluster Upgrade Process

### What is the current version of the cluster?

```bash
controlplane ~ ✖ kubectl version
Client Version: v1.28.0
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.0

controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   53m   v1.28.0
node01         Ready    <none>          52m   v1.28.0

controlplane ~ ➜
```

### How many nodes can host workloads in this cluster?

```bash
controlplane ~ ➜  kubectl describe nodes  controlplane | grep -i taint
Taints:             <none>

controlplane ~ ➜  kubectl describe nodes  node01 | grep -i taint
Taints:             <none>

controlplane ~ ➜
```

We can see that neither nodes have taints. This means that both nodes have the ability to schedule workloads on them.

### How many deployments are in this cluster? Which pod are they on?

```bash
controlplane ~ ➜  kubectl get deployments
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
blue   5/5     5            5           10m

controlplane ~ ➜  kubectl get pod -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
blue-667bf6b9f9-bsm62   1/1     Running   0          12m   10.244.1.4   node01         <none>           <none>
blue-667bf6b9f9-gswl9   1/1     Running   0          12m   10.244.0.5   controlplane   <none>           <none>
blue-667bf6b9f9-hlbbv   1/1     Running   0          12m   10.244.0.4   controlplane   <none>           <none>
blue-667bf6b9f9-q46lw   1/1     Running   0          12m   10.244.1.2   node01         <none>           <none>
blue-667bf6b9f9-qg99l   1/1     Running   0          12m   10.244.1.3   node01         <none>           <none>

controlplane ~ ➜
```

### You are tasked to upgrade the cluster.

Users accessing the applications must not be impacted, and you cannot provision new VMs. What strategy would you use to upgrade the cluster?

> In order to ensure minimum downtime, upgrade the cluster **one node at a time**, while moving the workloads to another node.

### What is the latest version available for an upgrade with the current version of the kubeadm tool installed?

```bash
controlplane ~ ➜  kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.28.0
[upgrade/versions] kubeadm version: v1.28.0
I0419 22:47:23.381941   17369 version.go:256] remote version is much newer: v1.30.0; falling back to: stable-1.28
[upgrade/versions] Target version: v1.28.9
[upgrade/versions] Latest version in the v1.28 series: v1.28.9

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     2 x v1.28.0   v1.28.9

Upgrade to the latest version in the v1.28 series:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.28.0   v1.28.9
kube-controller-manager   v1.28.0   v1.28.9
kube-scheduler            v1.28.0   v1.28.9
kube-proxy                v1.28.0   v1.28.9
CoreDNS                   v1.10.1   v1.10.1
etcd                      3.5.9-0   3.5.9-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.28.9

Note: Before you can perform this upgrade, you have to update kubeadm to v1.28.9.

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________


controlplane ~ ➜
```

### We will be upgrading the controlplane node first.

Drain the controlplane node of workloads and mark it `UnSchedulable`.

```bash
kubectl drain controlplane --ignore-daemonsets
```

There are `daemonsets` created in this cluster, especially in the **kube-system** namespace. To ignore these objects and drain the node, we can make use of the `--ignore-daemonsets` flag.

### Upgrade the `controlplane` components to exact version `v1.29.0`

To seamlessly transition from Kubernetes v1.28 to v1.29 and gain access to the packages specific to the desired Kubernetes minor version, follow these essential steps during the upgrade process. This ensures that your environment is appropriately configured and aligned with the features and improvements introduced in Kubernetes v1.29.

On the `controlplane` node:

Use any text editor you prefer to open the file that defines the Kubernetes apt repository.

```bash
vim /etc/apt/sources.list.d/kubernetes.list
```

Update the version in the URL to the next available minor release, i.e v1.29.

```bash
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /
```

After making changes, save the file and exit from your text editor. Proceed with the next instruction.

```bash
root@controlplane:~# apt update
root@controlplane:~# apt-cache madison kubeadm
```

Based on the version information displayed by `apt-cache madison`, it indicates that for Kubernetes version `1.29.0`, the available package version is `1.29.0-1.1`. Therefore, to install kubeadm for Kubernetes `v1.29.0`, use the following command:

```bash
root@controlplane:~# apt-get install kubeadm=1.29.0-1.1
```

Run the following command to upgrade the Kubernetes cluster.

```bash
root@controlplane:~# kubeadm upgrade plan v1.29.0
root@controlplane:~# kubeadm upgrade apply v1.29.0
```

> Note that the above steps can take a few minutes to complete.

Now, upgrade the version and restart Kubelet. Also, mark the node (in this case, the "controlplane" node) as schedulable.

```bash
root@controlplane:~# apt-get install kubelet=1.29.0-1.1
root@controlplane:~# systemctl daemon-reload
root@controlplane:~# systemctl restart kubelet
root@controlplane:~# kubectl uncordon controlplane
```

### Upgrade the `node01` components to exact version `v1.29.0`

Drain the `node01` node of workloads and mark it `UnSchedulable`.

```bash
kubectl drain node01 --ignore-daemonsets
```

Make sure you are on `node01`.

> If you are on the `controlplane` node, run `ssh node01` to log in to the `node01`.

Use any text editor you prefer to open the file that defines the Kubernetes apt repository.

```bash
vim /etc/apt/sources.list.d/kubernetes.list
```

Update the version in the URL to the next available minor release, i.e v1.29.

```bash
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /
```

After making changes, save the file and exit from your text editor. Proceed with the next instruction.

```bash
root@node01:~# apt update
root@node01:~# apt-cache madison kubeadm
```

Based on the version information displayed by `apt-cache madison`, it indicates that for Kubernetes version `1.29.0`, the available package version is `1.29.0-1.1`. Therefore, to install kubeadm for Kubernetes `v1.29.0`, use the following command:

```bash
root@node01:~# apt-get install kubeadm=1.29.0-1.1
# Upgrade the node
root@node01:~# kubeadm upgrade node
```

Now, upgrade the version and restart Kubelet.

```bash
root@node01:~# apt-get install kubelet=1.29.0-1.1
root@node01:~# systemctl daemon-reload
root@node01:~# systemctl restart kubelet
root@node01:~# kubectl uncordon node01
```

> Type `exit` or `logout` or enter `CTRL + d` to go back to the `controlplane` node.

---

## Cluster Maintenance - Practice Test - Backup and Restore Methods

### What is the version of ETCD running on the cluster?

Look at the ETCD Logs OR check the image used by ETCD pod.

```bash
controlplane ~ ➜  kubectl -n kube-system describe pod etcd-controlplane | grep Image:
    Image:         registry.k8s.io/etcd:3.5.10-0

controlplane ~ ➜  kubectl -n kube-system logs etcd-controlplane | grep -i 'etcd-version'
"etcd-version":"3.5.10"

controlplane ~ ➜
```

### At what address can you reach the ETCD cluster from the controlplane node?

```bash
kubectl -n kube-system describe pod etcd-controlplane | grep '\--listen-client-urls'
```

```bash
controlplane ~ ➜  kubectl -n kube-system describe pod etcd-controlplane | grep '\--listen-client-urls'
      --listen-client-urls=https://127.0.0.1:2379,https://192.6.109.3:2379
```

### Where is the ETCD server certificate file located?

```bash
kubectl -n kube-system describe pod etcd-controlplane | grep '\--cert-file'
```

```bash
controlplane ~ ➜  kubectl -n kube-system describe pod etcd-controlplane | grep '\--cert-file'
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
```

### Where is the ETCD CA Certificate file located?

```bash
kubectl -n kube-system describe pod etcd-controlplane | grep '\--trusted-ca-file'
```

```bash
controlplane ~ ➜  kubectl -n kube-system describe pod etcd-controlplane | grep '\--trusted-ca-file'
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

### Take a snapshot of the ETCD database using the built-in snapshot functionality.

The master node in our cluster is planned for a regular maintenance reboot tonight. While we do not anticipate anything to go wrong, we are required to take the necessary backups. Store the backup file at location `/opt/snapshot-pre-boot.db`

```bash
controlplane ~ ✖ ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/snapshot-pre-boot.db
Snapshot saved at /opt/snapshot-pre-boot.db

controlplane ~ ➜
```

Use the `etcdctl snapshot save` command. You will have to make use of additional flags to connect to the ETCD server.

- `--endpoints`: Optional Flag, points to the address where ETCD is running (127.0.0.1:2379)

- `--cacert`: Mandatory Flag (Absolute Path to the CA certificate file)

- `--cert`: Mandatory Flag (Absolute Path to the Server certificate file)

- `--key`: Mandatory Flag (Absolute Path to the Key file)

### Some Thing Went Wrong...

Luckily we took a backup. Restore the original state of the cluster using the backup file.

Restore the etcd to a new directory from the snapshot by using the `etcdctl snapshot restore` command. Once the directory is restored, update the ETCD configuration to use the restored directory.

1. Restore the snapshot:

```bash
controlplane ~ ✖  ETCDCTL_API=3 etcdctl  --data-dir /var/lib/etcd-from-backup \
snapshot restore /opt/snapshot-pre-boot.db
2024-04-20 04:44:39.173832 I | mvcc: restore compact to 1306
2024-04-20 04:44:39.179565 I | etcdserver/membership: added member 8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32

controlplane ~ ➜
```

> Note: In this case, we are restoring the snapshot to a different directory but in the same server where we took the backup (the controlplane node) As a result, the only required option for the restore command is the `--data-dir`.

2. Update the `/etc/kubernetes/manifests/etcd.yaml`:

We have now restored the etcd snapshot to a new path on the controlplane - `/var/lib/etcd-from-backup`, so, the only change to be made in the YAML file, is to change the hostPath for the volume called `etcd-data` from old directory (`/var/lib/etcd`) to the new directory (`/var/lib/etcd-from-backup`).

```yaml
volumes:
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
```

With this change, `/var/lib/etcd` on the container points to `/var/lib/etcd-from-backup` on the `controlplane` (which is what we want).

When this file is updated, the `ETCD` pod is automatically re-created as this is a static pod placed under the `/etc/kubernetes/manifests` directory.

> Note 1: As the ETCD pod has changed it will automatically restart, and also `kube-controller-manager` and `kube-scheduler`. Wait 1-2 to mins for this pods to restart. You can run the command: `watch "crictl ps | grep etcd"` to see when the ETCD pod is restarted.

> Note 2: If the etcd pod is not getting `Ready 1/1`, then restart it by `kubectl delete pod -n kube-system etcd-controlplane` and wait 1 minute.

> Note 3: This is the simplest way to make sure that ETCD uses the restored data after the ETCD pod is recreated. You don't have to change anything else.

If you do change `--data-dir` to `/var/lib/etcd-from-backup` in the ETCD YAML file, make sure that the `volumeMounts` for `etcd-data` is updated as well, with the mountPath pointing to `/var/lib/etcd-from-backup` (THIS COMPLETE STEP IS OPTIONAL AND NEED NOT BE DONE FOR COMPLETING THE RESTORE)

---

## Cluster Maintenance - Practice Test - Backup and Restore Methods 2

### How many `clusters` are defined in the kubeconfig on the `student-node`?

You can view the complete `kubeconfig` by running: `kubectl config view`

```bash
student-node ~ ➜  kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://cluster1-controlplane:6443
  name: cluster1
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.5.35.3:6443
  name: cluster2
contexts:
- context:
    cluster: cluster1
    user: cluster1
  name: cluster1
- context:
    cluster: cluster2
    user: cluster2
  name: cluster2
current-context: cluster1
kind: Config
preferences: {}
users:
- name: cluster1
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
- name: cluster2
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED

student-node ~ ➜
```

Alternatively, run the `kubectl config get-clusters` to just display the clusters:

```bash
student-node ~ ➜  kubectl config get-clusters
NAME
cluster2
cluster1

student-node ~ ➜
```

### How many nodes (both controlplane and worker) are part of `cluster1`?

Make sure to switch the context to `cluster1`:

```bash
kubectl config use-context cluster1
```

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  kubectl get nodes
NAME                    STATUS   ROLES           AGE   VERSION
cluster1-controlplane   Ready    control-plane   84m   v1.24.0
cluster1-node01         Ready    <none>          83m   v1.24.0

student-node ~ ➜
```

Therefore, `cluster1` has two nodes.

### How is ETCD configured for `cluster1`?

Remember, you can access the clusters from `student-node` using the `kubectl` tool. You can also `ssh` to the cluster nodes from the `student-node`.

Make sure to switch the context to `cluster1`:

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  kubectl get pods -n kube-system | grep etcd
etcd-cluster1-controlplane                      1/1     Running   0             88m

student-node ~ ➜
```

If you check out the pods running in the `kube-system` namespace in `cluster1`, you will notice that `etcd` is running as a pod:

This means that ETCD is set up as a `Stacked ETCD Topology` where the distributed data storage cluster provided by `etcd` is stacked on top of the cluster formed by the nodes managed by kubeadm that run control plane components.

### How is ETCD configured for `cluster2`?

Using the same approach as above, we can see there is no `etcd` pods running in this cluster.

```bash
student-node ~ ➜  kubectl config use-context cluster2
Switched to context "cluster2".

student-node ~ ➜   kubectl get pods -n kube-system  | grep etcd

student-node ~ ✖
```

So we can ssh into cluster2, only to see there is NO static pod configuration for etcd under the static pod path:

```bash
student-node ~ ✖ ssh cluster2-controlplane
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 5.4.0-1106-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

cluster2-controlplane ~ ➜  ls /etc/kubernetes/manifests/ | grep -i etcd

cluster2-controlplane ~ ✖
```

However, if you inspect the process on the controlplane for cluster2, you will see that that the process for the `kube-apiserver` is referencing an external etcd datastore:

```bash
cluster2-controlplane ~ ✖ ps -ef | grep etcd
root        1725    1389  0 03:40 ?        00:03:25 kube-apiserver --advertise-address=192.5.35.3 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.pem --etcd-certfile=/etc/kubernetes/pki/etcd/etcd.pem --etcd-keyfile=/etc/kubernetes/pki/etcd/etcd-key.pem --etcd-servers=https://192.5.35.14:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root       10257   10039  0 05:16 pts/0    00:00:00 grep etcd

cluster2-controlplane ~ ➜
```

You can see the same information by inspecting the `kube-apiserver` pod (which runs as a static pod in the kube-system namespace):

```bash
cluster2-controlplane ~ ➜  kubectl -n kube-system describe pod kube-apiserver-cluster2-controlplane
Name:                 kube-apiserver-cluster2-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 cluster2-controlplane/192.5.35.3
Start Time:           Sat, 20 Apr 2024 03:41:10 +0000
Labels:               component=kube-apiserver
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.5.35.3:6443
                      kubernetes.io/config.hash: 046fd3d7e89b50d726a714e5471a3269
                      kubernetes.io/config.mirror: 046fd3d7e89b50d726a714e5471a3269
                      kubernetes.io/config.seen: 2024-04-20T03:41:09.844426451Z
                      kubernetes.io/config.source: file
                      seccomp.security.alpha.kubernetes.io/pod: runtime/default
Status:               Running
IP:                   192.5.35.3
IPs:
  IP:           192.5.35.3
Controlled By:  Node/cluster2-controlplane
Containers:
  kube-apiserver:
    Container ID:  containerd://3c9c999d5407daf0fec09c4e5434cf3b7567e818a58788540dc1bb357458dd96
    Image:         k8s.gcr.io/kube-apiserver:v1.24.0
    Image ID:      k8s.gcr.io/kube-apiserver@sha256:a04522b882e919de6141b47d72393fb01226c78e7388400f966198222558c955
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-apiserver
      --advertise-address=192.5.35.3
      --allow-privileged=true
      --authorization-mode=Node,RBAC
      --client-ca-file=/etc/kubernetes/pki/ca.crt
      --enable-admission-plugins=NodeRestriction
      --enable-bootstrap-token-auth=true
      --etcd-cafile=/etc/kubernetes/pki/etcd/ca.pem
      --etcd-certfile=/etc/kubernetes/pki/etcd/etcd.pem
      --etcd-keyfile=/etc/kubernetes/pki/etcd/etcd-key.pem
      --etcd-servers=https://192.5.35.14:2379 # look at here
--------- End of Snippet---------
```

### What is the default data directory used the for ETCD datastore used in `cluster1`?

Inspect the value assigned to `data-dir` on the `etcd` pod:

```bash
student-node ~ ✖ kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  kubectl -n kube-system describe pod etcd-cluster1-controlplane | grep data-dir
      --data-dir=/var/lib/etcd

student-node ~ ➜
```

### What is the default data directory used the for ETCD datastore used in `cluster2`?

SSH to the `etcd-server` and inspect the `etcd` process as shown below:

```bash

student-node ~ ➜  ssh etcd-server
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 5.4.0-1106-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

etcd-server ~ ➜   kubectl -n kube-system describe pod etcd-cluster1-controlplane | grep data-dir
-bash: kubectl: command not found

etcd-server ~ ✖ ps -ef | grep etcd
etcd         833       1  0 03:40 ?        00:01:19 /usr/local/bin/etcd --name etcd-server --data-dir=/var/lib/etcd-data --cert-file=/etc/etcd/pki/etcd.pem --key-file=/etc/etcd/pki/etcd-key.pem --peer-cert-file=/etc/etcd/pki/etcd.pem --peer-key-file=/etc/etcd/pki/etcd-key.pem --trusted-ca-file=/etc/etcd/pki/ca.pem --peer-trusted-ca-file=/etc/etcd/pki/ca.pem --peer-client-cert-auth --client-cert-auth --initial-advertise-peer-urls https://192.5.35.14:2380 --listen-peer-urls https://192.5.35.14:2380 --advertise-client-urls https://192.5.35.14:2379 --listen-client-urls https://192.5.35.14:2379,https://127.0.0.1:2379 --initial-cluster-token etcd-cluster-1 --initial-cluster etcd-server=https://192.5.35.14:2380 --initial-cluster-state new
root        1105     983  0 05:29 pts/0    00:00:00 grep etcd

etcd-server ~ ➜
```

### How many nodes are part of the ETCD cluster that `etcd-server` is a part of?

```bash
etcd-server ~ ➜  ETCDCTL_API=3 etcdctl \
>  --endpoints=https://127.0.0.1:2379 \
>  --cacert=/etc/etcd/pki/ca.pem \
>  --cert=/etc/etcd/pki/etcd.pem \
>  --key=/etc/etcd/pki/etcd-key.pem \
>   member list
820681864077f749, started, etcd-server, https://192.5.35.14:2380, https://192.5.35.14:2379, false

etcd-server ~ ➜
```

### Take a backup of etcd on `cluster1` and save it on the `student-node` at the path `/opt/cluster1.db`

On the `student-node`:

1. First set the context to `cluster1`:
2. Next, inspect the endpoints and certificates used by the `etcd` pod. We will make use of these to take the backup.
3. SSH to the `controlplane` node of `cluster1` and then take the backup using the endpoints and certificates we identified above.
4. Finally, copy the backup to the `student-node`. To do this, go back to the `student-node` and use `scp` as shown below.

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  kubectl describe  pods -n kube-system etcd-cluster1-controlplane  | grep advertise-client-urls
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.5.35.22:2379
      --advertise-client-urls=https://192.5.35.22:2379

student-node ~ ➜  kubectl describe  pods -n kube-system etcd-cluster1-controlplane  | grep pki
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
    Path:          /etc/kubernetes/pki/etcd

student-node ~ ➜  ssh cluster1-controlplane
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 5.4.0-1106-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

cluster1-controlplane ~ ✖ ETCDCTL_API=3 etcdctl --endpoints=https://192.5.35.22:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/cluster1.db
Snapshot saved at /opt/cluster1.db

cluster1-controlplane ~ ➜  exit
logout
Connection to cluster1-controlplane closed.

student-node ~ ➜  scp cluster1-controlplane:/opt/cluster1.db /opt
cluster1.db                                                                        100% 2060KB 144.9MB/s   00:00

student-node ~ ➜
```

### An ETCD backup for cluster2 is stored at /opt/cluster2.db. Use this snapshot file to carryout a restore on cluster2 to a new path /var/lib/etcd-data-new.

Step 1. Copy the snapshot file from the student-node to the etcd-server. In the example below, we are copying it to the /root directory:

```bash
student-node ~ ➜  kubectl config use-context cluster2
Switched to context "cluster2".

student-node ~ ➜  scp /opt/cluster2.db etcd-server:/root
cluster2.db                                                                        100% 2052KB 168.0MB/s   00:00

student-node ~ ➜
```

Step 2: Restore the snapshot on the `cluster2`. Since we are restoring directly on the `etcd-server`, we can use the endpoint `https:/127.0.0.1`. Use the same certificates that were identified earlier. Make sure to use the `data-dir` as `/var/lib/etcd-data-new`:

```bash
etcd-server ~ ➜  ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd.pem --key=/etc/etcd/pki/etcd-key.pem snapshot restore /root/cluster2.db --data-dir /var/lib/etcd-data-new
{"level":"info","ts":1713591986.451862,"caller":"snapshot/v3_snapshot.go:296","msg":"restoring snapshot","path":"/root/cluster2.db","wal-dir":"/var/lib/etcd-data-new/member/wal","data-dir":"/var/lib/etcd-data-new","snap-dir":"/var/lib/etcd-data-new/member/snap"}
{"level":"info","ts":1713591986.469451,"caller":"mvcc/kvstore.go:388","msg":"restored last compact revision","meta-bucket-name":"meta","meta-bucket-name-key":"finishedCompactRev","restored-compact-revision":9495}
{"level":"info","ts":1713591986.4752784,"caller":"membership/cluster.go:392","msg":"added member","cluster-id":"cdf818194e3a8c32","local-member-id":"0","added-peer-id":"8e9e05c52164694d","added-peer-peer-urls":["http://localhost:2380"]}
{"level":"info","ts":1713591986.529377,"caller":"snapshot/v3_snapshot.go:309","msg":"restored snapshot","path":"/root/cluster2.db","wal-dir":"/var/lib/etcd-data-new/member/wal","data-dir":"/var/lib/etcd-data-new","snap-dir":"/var/lib/etcd-data-new/member/snap"}

etcd-server ~ ➜
```

Step 3: Update the systemd service unit file for `etcd` by running `vi /etc/systemd/system/etcd.service` and add the new value for `data-dir`:

```bash
etcd-server ~ ➜  vi /etc/systemd/system/etcd.service

etcd-server ~ ➜
```

```yaml
[Unit]
Description=etcd key-value store
Documentation=https://github.com/etcd-io/etcd
After=network.target

[Service]
User=etcd
Type=notify
ExecStart=/usr/local/bin/etcd \
  --name etcd-server \
  --data-dir=/var/lib/etcd-data-new \
---End of Snippet---
```

Step 4: make sure the permissions on the new directory is correct (should be owned by etcd user):

```bash
etcd-server ~ ➜  chown -R etcd:etcd /var/lib/etcd-data-new

etcd-server ~ ➜  ls -ld /var/lib/etcd-data-new/
drwx------ 3 etcd etcd 4096 Apr 20 05:46 /var/lib/etcd-data-new/

etcd-server ~ ➜
```

Step 5: Finally, reload and restart the etcd service.

```bash
etcd-server ~ ➜  systemctl daemon-reload

etcd-server ~ ➜   systemctl restart etcd

etcd-server ~ ➜
```

Step 6 (optional): It is recommended to restart controlplane components (e.g. kube-scheduler, kube-controller-manager, kubelet) to ensure that they don't rely on some stale data.

---

## Security - Practice Test - View Certificate Details

> 🤔 What's the best way to understand so many certificates & keys in K8s?

- If you install Kubernetes with `kubeadm`, most certificates are stored in `/etc/kubernetes/pki`. All paths in this documentation are relative to that directory, with the exception of user account certificates which kubeadm places in `/etc/kubernetes`.

### Identify the certificate file used for the `kube-api server`.

> The Kubernetes API server validates and configures data for the api objects which include pods, services, replicationcontrollers, and others. The API Server services REST operations and provides the frontend to the cluster's shared state through which all other components interact.

Run the command `cat /etc/kubernetes/manifests/kube-apiserver.yaml` and look for the line `--tls-cert-file`.

```bash
controlplane /etc/kubernetes/manifests ➜  ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml

controlplane /etc/kubernetes/manifests ➜  cat kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.5.145.3:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.5.145.3
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt # next question
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt # here, this question
```

### Identify the Certificate file used to authenticate `kube-apiserver` as a client to `ETCD` Server.

Run the command `cat /etc/kubernetes/manifests/kube-apiserver.yaml`(same as above) and look for value of `etcd-certfile` flag.

### Identify the key used to authenticate `kubeapi-server` to the `kubelet` server.

Look for kubelet-client-key option in the file `/etc/kubernetes/manifests/kube-apiserver.yaml`.

### Identify the ETCD Server Certificate used to host ETCD server.

Look for `cert-file` option in the file `/etc/kubernetes/manifests/etcd.yaml`.

```bash
controlplane /etc/kubernetes/manifests ➜  ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml

controlplane /etc/kubernetes/manifests ➜  cat etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.5.145.3:2379
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.5.145.3:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt # here, this question
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --experimental-initial-corrupt-check=true
    - --experimental-watch-progress-notify-interval=5s
    - --initial-advertise-peer-urls=https://192.5.145.3:2380
    - --initial-cluster=controlplane=https://192.5.145.3:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.5.145.3:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://192.5.145.3:2380
    - --name=controlplane
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt # next question
```

### Identify the ETCD Server CA Root Certificate used to serve ETCD Server.

> ETCD can have its own CA. So this may be a different CA certificate than the one used by kube-api server.

Look for CA Certificate (`trusted-ca-file`) in file `/etc/kubernetes/manifests/etcd.yaml`.

### What is the Common Name (CN) configured on the Kube API Server Certificate?

OpenSSL Syntax: `openssl x509 -in file-path.crt -text -noout`

Run the command `openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text` and look for Subject CN.

```bash
controlplane / ➜  openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 798145043669162787 (0xb1394b03f239b23)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes # next question
        Validity
            Not Before: Apr 22 07:36:58 2024 GMT
            Not After : Apr 22 07:41:58 2025 GMT
        Subject: CN = kube-apiserver # here, this question
```

### What is the name of the CA who issued the Kube API Server Certificate?

Run the command `openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text` and look for issuer.

### Which alternate names are configured on the Kube API Server Certificate?

Run the command `openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text` and look at Alternative Names.

```bash
controlplane / ✖ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text | grep -A2 "Alternative Name:"
            X509v3 Subject Alternative Name:
                DNS:controlplane, DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, IP Address:10.96.0.1, IP Address:192.5.145.3
    Signature Algorithm: sha256WithRSAEncryption

controlplane / ➜
```

### What is the Common Name (CN) configured on the ETCD Server certificate?

Run the command `openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text` and look for `Subject CN`.

```bash
controlplane ~ ✖ openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text | grep "Subject:"
        Subject: CN = controlplane

controlplane ~ ➜
```

# How long, from the issued date, is the Kube-API Server Certificate valid for?

> File: `/etc/kubernetes/pki/apiserver.crt`

Run the command `openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text` and check on the `Expiry date`.

```bash
controlplane ~ ➜  openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text | grep -A2 Validity
        Validity
            Not Before: Apr 22 17:36:36 2024 GMT
            Not After : Apr 22 17:41:36 2025 GMT

controlplane ~ ➜
```

# How long, from the issued date, is the Root CA Certificate valid for?

> File: `/etc/kubernetes/pki/ca.crt`

```bash
controlplane ~ ➜  openssl x509 -in /etc/kubernetes/pki/ca.crt -text | grep -A2 Validity
        Validity
            Not Before: Apr 22 17:36:36 2024 GMT
            Not After : Apr 20 17:41:36 2034 GMT

controlplane ~ ➜
```

### Kubectl suddenly stops responding to your commands. Check it out! Someone recently modified the `/etc/kubernetes/manifests/etcd.yaml` file

> You are asked to investigate and fix the issue. Once you fix the issue wait for sometime for kubectl to respond. Check the logs of the ETCD container.

The certificate file used here is incorrect. It is set to `/etc/kubernetes/pki/etcd/server-certificate.crt` which does not exist. As we saw in the previous questions the correct path should be `/etc/kubernetes/pki/etcd/server.crt`.

```bash
root@controlplane:~# ls -l /etc/kubernetes/pki/etcd/server* | grep .crt
-rw-r--r-- 1 root root 1188 May 20 00:41 /etc/kubernetes/pki/etcd/server.crt
root@controlplane:~#
```

Update the YAML file with the correct certificate path and wait for the ETCD pod to be recreated. wait for the `kube-apiserver` to get to a `Ready` state.

### The kube-api server stopped again! Check it out. Inspect the kube-api server logs and identify the root cause and fix the issue.

> Run `crictl ps -a` command to identify the kube-api server container. Run `crictl logs container-id` command to view the logs.

```bash
controlplane ~ ➜  crictl ps -a | grep kube-apiserver
6ed2117c764cb       1443a367b16d3       45 seconds ago      Exited              kube-apiserver            7                   f84b6be3df02d       kube-apiserver-controlplane

controlplane ~ ➜   crictl logs --tail=2 6ed2117c764cb
W0422 18:26:54.724441       1 logging.go:59] [core] [Channel #2 SubChannel #4] grpc: addrConn.createTransport failed to connect to {Addr: "127.0.0.1:2379", ServerName: "127.0.0.1:2379", }. Err: connection error: desc = "transport: authentication handshake failed: tls: failed to verify certificate: x509: certificate signed by unknown authority"
F0422 18:26:56.878763       1 instance.go:290] Error creating leases: error creating storage factory: context deadline exceeded

controlplane ~ ➜
```

This indicates an issue with the ETCD CA certificate used by the `kube-apiserver`. Correct it to use the file `/etc/kubernetes/pki/etcd/ca.crt`.

Once the YAML file has been saved, wait for the `kube-apiserver` pod to be Ready. This can take a couple of minutes.

---

## Security - Practice Test - Certificates API

### Inspect a Certificate Signing Request(CSR)

A new member `akshay` joined our team. He requires access to our cluster. The Certificate Signing Request is at the `/root` location.

```bash
controlplane ~ ➜  ls
akshay.csr  akshay.key

controlplane ~ ✖ cat akshay.csr
-----BEGIN CERTIFICATE REQUEST-----
MIICVjCCAT4CAQAwETEPMA0GA1UEAwwGYWtzaGF5MIIBIjANBgkqhkiG9w0BAQEF
AAOCAQ8AMIIBCgKCAQEAzbGDtwIwULAtsBHz+jj4xgN6rK+rzIiB+blB7y1oNadJ
IfF4Dq+i8y+QY4ID4ewnjO8XemDqZejICG3CVxxxBo89AcUC7xE0+WU3EaJIp/5N
wLim6wm8vJ9VFTSexLh2RHY/FqqJHVTgYTGiQvr+L/VJ11bgZC6kLfEAAUbgizTH
CWS89J/zoMQJ0MkU4DuwMwfXseliHb3gPZCLUjeV95JZt22g+Hzrc2sKDDrQ+DEO
/k4VeR2Rt4rE8Xnh2NPXF+k5iPdNipRyWgfWpQzJal+R7swJfJSlFvrTpAdYWpiw
V3+4VwOzm1i5KfULKUvU6suIQZS2UUYSQ+jqvJtUZQIDAQABoAAwDQYJKoZIhvcN
AQELBQADggEBAA/NAbtsiU+BiEf15wozUqGxiC/GwdpKQlrScRiCQcwPch9k5C0P
6JrSHHnyd8yZuR+c2Tu7+k2OJm0BIDtr5Fvkb58cdypvq23QWLSgg2QQpg0d2a/Y
usnzzofQAwCCiXdOPJsQIbEhxcwwAsxqJWpqPFg6HOOlLlsy4k+aYRVJWqnJYTeH
iLQLXcAoTCivI1hAeeDz4PsJOHAKc9eYNPFPNimhO2JjHW5OA3I6Cggpy2je9mwW
Bm2FwTwTv5Rn2krYaORzH/vNKMtjIVo2Oxs81CGQiZsln3dEwdsH9dWGNyXFLDU8
WOPohNY5vvuagjolxIVdv7ZvAdR4T4IZ5ZQ=
-----END CERTIFICATE REQUEST-----

controlplane ~ ➜
```

### Create a CertificateSigningRequest object

Create a CertificateSigningRequest object with the name `akshay` with the contents of the `akshay.csr` file.

As of kubernetes 1.19, the API to use for CSR is `certificates.k8s.io/v1`.

Please note that an additional field called `signerName` should also be added when creating CSR. For client authentication to the API server we will use the built-in signer `kubernetes.io/kube-apiserver-client`.

Use this command to generate the base64 encoded format as following:

```bash
cat akshay.csr | base64 -w 0 # not to insert any line breaks in the output.
```

Finally, save the below YAML in a file and create a CSR name `akshay` as follows:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  groups:
    - system:authenticated
  request: <Paste the base64 encoded value of the CSR file>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
    - client auth
```

&

```bash
controlplane ~ ➜  kubectl apply -f akshay-csr.yaml
certificatesigningrequest.certificates.k8s.io/akshay created

controlplane ~ ➜
```

### What is the Condition of the newly created Certificate Signing Request object?

```bash
controlplane ~ ➜  kubectl get csr
NAME        AGE   SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
akshay      97s   kubernetes.io/kube-apiserver-client           kubernetes-admin           <none>              Pending # here
csr-zj4bp   13m   kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued

controlplane ~ ➜
```

### Approve the CSR Request

```bash
controlplane ~ ➜  kubectl certificate approve akshay
certificatesigningrequest.certificates.k8s.io/akshay approved

controlplane ~ ➜  kubectl get csr
NAME        AGE     SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
akshay      5m33s   kubernetes.io/kube-apiserver-client           kubernetes-admin           <none>              Approved,Issued # here
csr-zj4bp   17m     kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued

controlplane ~ ➜
```

### Check the details about a CSR request. Preferebly in YAML.

```bash
kubectl get csr agent-smith -o yaml
```

### Reject a CSR request.

```bash
controlplane ~ ✖ kubectl certificate deny agent-smith
certificatesigningrequest.certificates.k8s.io/agent-smith denied

controlplane ~ ➜
```

### Delete a CSR request.

```bash
controlplane ~ ➜  kubectl delete csr agent-smith
certificatesigningrequest.certificates.k8s.io "agent-smith" deleted

controlplane ~ ➜
```

---

## Security - Practice Test - KubeConfig

### Identify the location of the default kubeconfig file (in current environment)

Find the current home directory by looking at the HOME environment variable.

```bash
controlplane ~ ➜  ls -a
.  ..  .bash_profile  .bashrc  .cache  CKA  .config  .kube  my-kube-config  .profile  sample.yaml  .ssh  .vim  .vimrc  .wget-hsts

controlplane ~ ➜  cd .kube

controlplane ~/.kube ➜  ls
cache  config # /root/.kube/config
```

### Get the number of clusters defined in the default kubeconfig file

```bash
controlplane ~/.kube ➜  kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://controlplane:6443
  name: kubernetes # just one cluster
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes # just one context
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin # just one user
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED

controlplane ~/.kube ➜
```

### Inspect a kubeconfig file `my-kube-config`

```bash
kubectl config view --kubeconfig my-kube-config
```

### Use context (not the default one)

I would like to use the `dev-user` to access `test-cluster-1`. Set the current context to the right one so I can do that.

```bash
controlplane ~ ➜  kubectl config --kubeconfig=/root/my-kube-config use-context research
Switched to context "research".

controlplane ~ ➜  kubectl config --kubeconfig=/root/my-kube-config current-context
research

controlplane ~ ➜
```

### Change the default kubeconfig

We don't want to have to specify the kubeconfig file option on each command.

Set the `my-kube-config` file as the default kubeconfig by overwriting the content of `~/.kube/config` with the content of the `my-kube-config` file.

```bash
cp my-kube-config ~/.kube/config # just a simple `cp <source> <destination>` command
```

---

## Security - Practice Test - Role Based Access Controls

### Identify the authorization modes configured on the cluster.

```bash
controlplane ~ ✖ kubectl describe pod kube-apiserver-controlplane -n kube-system | grep authorization-mode
      --authorization-mode=Node,RBAC

controlplane ~ ➜
```

### How many roles exist in the `default` namespace?

```bash
kubectl get roles
```

### How many roles exist in all namespaces together?

```bash
kubectl get roles --all-namespaces
```

or

```bash
kubectl get roles -A
```

```bash
controlplane ~ ✖ kubectl get roles -A
NAMESPACE     NAME                                             CREATED AT
blue          developer                                        2024-04-27T00:24:56Z
kube-public   kubeadm:bootstrap-signer-clusterinfo             2024-04-27T00:11:38Z
kube-public   system:controller:bootstrap-signer               2024-04-27T00:11:36Z
kube-system   extension-apiserver-authentication-reader        2024-04-27T00:11:36Z
kube-system   kube-proxy                                       2024-04-27T00:11:38Z
kube-system   kubeadm:kubelet-config                           2024-04-27T00:11:36Z
kube-system   kubeadm:nodes-kubeadm-config                     2024-04-27T00:11:36Z
kube-system   system::leader-locking-kube-controller-manager   2024-04-27T00:11:36Z
kube-system   system::leader-locking-kube-scheduler            2024-04-27T00:11:36Z
kube-system   system:controller:bootstrap-signer               2024-04-27T00:11:36Z
kube-system   system:controller:cloud-provider                 2024-04-27T00:11:36Z
kube-system   system:controller:token-cleaner                  2024-04-27T00:11:36Z

controlplane ~ ➜  kubectl get roles -A | wc -l
13

controlplane ~ ✖ kubectl get roles -A --no-headers | wc -l
12
```

### What are the resources the `kube-proxy` role in the `kube-system` namespace is given access to?

```bash
controlplane ~ ➜  kubectl describe role kube-proxy -n kube-system
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources   Non-Resource URLs  Resource Names  Verbs
  ---------   -----------------  --------------  -----
  configmaps  []                 [kube-proxy]    [get]  # meaning `kube-proxy` role can perform `get` action for `configmaps` resource

controlplane ~ ➜
```

### Which account is the `kube-proxy` role assigned to?

```bash
controlplane ~ ➜  kubectl describe rolebinding kube-proxy -n kube-system
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  kube-proxy
Subjects:
  Kind   Name                                             Namespace
  ----   ----                                             ---------
  Group  system:bootstrappers:kubeadm:default-node-token

controlplane ~ ➜

```

### Check if the user can list pods in the default namespace.

A user `dev-user` is created. User's details have been added to the `kubeconfig` file. Inspect the permissions granted to the user.

```bash
controlplane ~ ➜  kubectl get pod --as dev-user
Error from server (Forbidden): pods is forbidden: User "dev-user" cannot list resource "pods" in API group "" in the namespace "default"

controlplane ~ ✖
```

### Create the necessary roles and role bindings required for the `dev-user` to create, list and delete pods in the `default` namespace.

1. Imperative

To create a Role:

```bash
kubectl create role developer --namespace=default --verb=list,create,delete --resource=pods
```

To create a RoleBinding:

```bash
kubectl create rolebinding dev-user-binding --namespace=default --role=developer --user=dev-user
```

```bash
controlplane ~ ✖ kubectl create role developer --namespace=default --verb=list,create,delete --resource=pods
role.rbac.authorization.k8s.io/devloper created

controlplane ~ ➜  kubectl create rolebinding dev-user-binding --namespace=default --role=developer --user=dev-user
rolebinding.rbac.authorization.k8s.io/dev-user-binding created

controlplane ~ ✖ kubectl get pod --as dev-user
NAME                   READY   STATUS    RESTARTS   AGE
red-5bd8fd8c7f-rpl47   1/1     Running   0          53m
red-5bd8fd8c7f-sn4jl   1/1     Running   0          53m

controlplane ~ ➜

```

2. Declarative

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: developer
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list", "create", "delete"]

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-binding
subjects:
  - kind: User
    name: dev-user
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

---

## Security - Practice Test - Cluster Roles

### How many `ClusterRoles` do you see defined in the cluster?

```bash
controlplane ~ ➜  kubectl get clusterroles --no-headers  | wc -l
71

controlplane ~ ➜  kubectl get clusterroles --no-headers  -o json | jq '.items | length'
71
```

### How many `ClusterRoleBindings` exist on the cluster?

```bash
controlplane ~ ➜  kubectl get clusterrolebinding --no-headers  | wc -l
56

controlplane ~ ➜  kubectl get clusterrolebindings --no-headers  -o json | jq '.items | length'
56

controlplane ~ ➜
```

### What namespace is the `cluster-admin` clusterrole part of?

ClusterRole is a non-namespaced resource. You can check via the `kubectl api-resources --namespaced=false` command. So the correct answer would be `Cluster Roles are cluster wide and not part of any namespace`.

```bash
controlplane ~ ➜  kubectl api-resources --namespaced=false
NAME                              SHORTNAMES   APIVERSION                        NAMESPACED   KIND
componentstatuses                 cs           v1                                false        ComponentStatus
namespaces                        ns           v1                                false        Namespace
nodes                             no           v1                                false        Node
persistentvolumes                 pv           v1                                false        PersistentVolume
mutatingwebhookconfigurations                  admissionregistration.k8s.io/v1   false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io/v1   false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1           false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io/v1         false        APIService
selfsubjectreviews                             authentication.k8s.io/v1          false        SelfSubjectReview
tokenreviews                                   authentication.k8s.io/v1          false        TokenReview
selfsubjectaccessreviews                       authorization.k8s.io/v1           false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io/v1           false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io/v1           false        SubjectAccessReview
certificatesigningrequests        csr          certificates.k8s.io/v1            false        CertificateSigningRequest
flowschemas                                    flowcontrol.apiserver.k8s.io/v1   false        FlowSchema
prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1   false        PriorityLevelConfiguration
etcdsnapshotfiles                              k3s.cattle.io/v1                  false        ETCDSnapshotFile
nodes                                          metrics.k8s.io/v1beta1            false        NodeMetrics
ingressclasses                                 networking.k8s.io/v1              false        IngressClass
runtimeclasses                                 node.k8s.io/v1                    false        RuntimeClass
clusterrolebindings                            rbac.authorization.k8s.io/v1      false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io/v1      false        ClusterRole # here
priorityclasses                   pc           scheduling.k8s.io/v1              false        PriorityClass
csidrivers                                     storage.k8s.io/v1                 false        CSIDriver
csinodes                                       storage.k8s.io/v1                 false        CSINode
storageclasses                    sc           storage.k8s.io/v1                 false        StorageClass
volumeattachments                              storage.k8s.io/v1                 false        VolumeAttachment

controlplane ~ ➜
```

### What user/groups are the `cluster-admin` role bound to?

```bash
controlplane ~ ➜  k describe clusterrolebinding cluster-admin
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind   Name            Namespace
  ----   ----            ---------
  Group  system:masters # here

controlplane ~ ➜
```

### Inspect the `cluster-admin` role's privileges.

```bash
controlplane ~ ➜  k describe clusterrole cluster-admin
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *.*        []                 []              [*]
             [*]                []              [*]

controlplane ~ ➜
```

### Grant permission to access nodes

A new user `michelle` joined the team. She will be focusing on the `nodes` in the cluster. Create the required `ClusterRoles` and `ClusterRoleBindings` so she gets access to the `nodes`.

```yaml
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: node-admin
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "watch", "list", "create", "delete"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-binding
subjects:
  - kind: User
    name: michelle
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-admin
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl auth can-i list nodes --as michelle.
```

### `michelle`'s responsibilities are growing and now she will be responsible for storage as well. Create the required `ClusterRoles` and `ClusterRoleBindings` to allow her access to Storage.

```bash
controlplane ~ ➜  kubectl auth can-i list storageclasses --as michelle
Warning: resource 'storageclasses' is not namespace scoped in group 'storage.k8s.io'

no

controlplane ~ ✖ kubectl create clusterrole storage-admin --resource=persistentvolumes,storageclasses --verb=* --dry-run=client -o yaml > storage-admin-clusterrole.yaml

controlplane ~ ➜  vi storage-admin-clusterrole.yaml

controlplane ~ ➜  kubectl create clusterrolebinding michelle-storage-admin --user=michelle --clusterrole=storage-admin --dry-run=client -o yaml > storage-admin-rolebinding.yaml

controlplane ~ ➜  vi storage-admin-rolebinding.yaml

controlplane ~ ➜  kubectl create -f storage-admin-clusterrole.yaml
clusterrole.rbac.authorization.k8s.io/storage-admin created

controlplane ~ ➜  kubectl create -f storage-admin-rolebinding.yaml
clusterrolebinding.rbac.authorization.k8s.io/michelle-storage-admin created

controlplane ~ ➜  kubectl auth can-i list storageclasses --as michelle
Warning: resource 'storageclasses' is not namespace scoped in group 'storage.k8s.io'

yes

controlplane ~ ➜
```

---

## Security - Practice Test - Service Accounts

### How many Service Accounts exist in the default namespace?

```bash
kubectl get serviceaccounts
```

### What is the secret token used by the default service account?

```bash
kubectl describe serviceaccount default # look for `Tokens` field
```

### Create a new Service Account `dashboard-sa`

```bash
kubectl create serviceaccount dashboard-sa
```

### Create a token for the Service Account `dashboard-sa`

```bash
kubectl create token dashboard-sa
```

---

## Security - Practice Test - Image Security

### What secret type must be chosen for `docker registry`?

```bash
kubectl create secret --help # answer is `docker-registry`
```

### Create a secret object with the credentials required to access to a private registry.

- Name: `private-reg-cred`
- Username: `dock_user`
- Password: `dock_password`
- Server: `myprivateregistry.com:5000`
- Email: `dock_user@privateregistry.com`

```bash
kubectl create secret docker-registry private-reg-cred --docker-username=dock_user --docker-password=dock_password --docker-server=myprivateregistry.com:5000 --docker-email=dock_user@myprivateregistry.com
```

---

## Security - Practice Test - Security Contexts

### Check which user is used to execute commands inside the `ubuntu-sleeper` pod?

```bash
kubectl exec ubuntu-sleeper -- whoami
```

### Change Security Conext of this pod to use User ID 1010

```yaml
# In pod yaml, add under spec
spec:
  securityContext:
    runAsUser: 1010
```

### Inheritance of Security Context in multi-container pod

```yaml
apiVersion: v1
kind: pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
    - image: ubuntu # this container uses user 1002
      name: web
      command: ["sleep", "5000"]
      securityContext:
        runAsUser: 1002

    - image: ubuntu # this container uses user 1001, as inherited from top-level's user.
      name: sidecar
      command: ["sleep", "5000"]
```

### Add Capabilities to a pod via Security Context

```yaml
spec:
  containers:
    - image: ubuntu
      name: ubuntu-sleeper
      securityContext:
        capabilities:
          add: ["SYS_TIME"]
```

---

## Security - Practice Test - Network Policies

### Get all Network Policies

```bash
kubectl get networkpolicy

# or
kubectl get netpol
```

### Get details of a Network Policy

```bash
kubectl describe networkpolicy
```

### Create Ingress and Egress

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/efc9bc9e-e9ea-25dc-f3cc-0bb6258ce367.png)

Create a network policy to allow traffic from the `Internal` application only to the `payroll-service` and `db-service`.
Use the spec given below. You might want to enable ingress traffic to the pod.
Also, ensure that you allow egress traffic to DNS ports TCP and UDP (port 53) to enable DNS resolution from the internal pod.

- Policy Name: internal-policy
- Policy Type: Egress
- Egress Allow: payroll
- Payroll Port: 8080
- Egress Allow: mysql
- MySQL Port: 3306

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
    - Egress
    - Ingress
  ingress:
    - {}
  egress:
    - to:
        - podSelector:
            matchLabels:
              name: mysql
      ports:
        - protocol: TCP
          port: 3306

    - to:
        - podSelector:
            matchLabels:
              name: payroll
      ports:
        - protocol: TCP
          port: 8080

    - ports:
        - port: 53
          protocol: UDP
        - port: 53
          protocol: TCP
```

{{< notice note >}}
We have also allowed `Egress` traffic to `TCP` and `UDP` port. This has been added to ensure that the internal DNS resolution works from the `internal` pod.
{{< /notice >}}

The `kube-dns` service is exposed on port `53`:

```bash
root@controlplane:~> kubectl get svc -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   18m

root@controlplane:~>
```

---

## Storage - Practice Test - Persistent Volume Claims

### Exec into container

Currently, there is a pod `webapp`. We can see its log using:

```bash
# kubectl exec <podName> -- <command to execute inside container>
kubectl exec webapp -- cat /log/app.log
```

However, the logs will disapear if the pod is deleted.

### Configure a volume to store these logs at `/var/log/webapp` on the host.

- Name: webapp
- Image Name: kodekloud/event-simulator
- Volume HostPath: /var/log/webapp
- Volume Mount: /log

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: event-simulator
      image: kodekloud/event-simulator
      env:
        - name: LOG_HANDLERS
          value: file
      volumeMounts:
        - mountPath: /log # container file location
          name: log-volume

volumes:
  - name: log-volume
    hostPath:
      path: /var/log/webapp # host file location
      type: Directory # optional
```

### Create a `Persistent Volume` with the given specification

- Volume Name: pv-log
- Storage: 100Mi
- Access Modes: ReadWriteMany
- Host Path: /pv/log
- Reclaim Policy: Retain

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath:
    path: /pv/log
```

### Create a `Persistent Volume Claim` with the given specification

- Volume Name: claim-log-1
- Storage Request: 50Mi
- Access Modes: ReadWriteOnce

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

### Get all `Persistent Volume` and `Persistent Volume Claim`

```bash
kubectl get pvc

kubectl get pv
```

### Why is the claim not bound to the available PV?

Access Mode Mismatch

### Update webapp pod to use this pv

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: event-simulator
      image: kodekloud/event-simulator
      env:
        - name: LOG_HANDLERS
          value: file
      volumeMounts:
        - mountPath: /log # container file location
          name: log-volume

volumes:
  - name: log-volume
    # hostPath:
    #   path: /var/log/webapp # host file location
    #   type: Directory # optional
    persistentVolumeClaim:
      claimName: claim-log-1
```

### Relationship between PersistentVolume (PV), PersistentVolumeClaim (PVC), and Pod

- If a PVC is being used by a Pod, you typically can't delete the PVC without first deleting the Pod. This is because the Pod is bound to the PVC, and deleting the PVC would disrupt the Pod's ability to access its required storage.

- If the Pod using a PVC is deleted, the PVC will be in a "Pending" state. This means the PVC is not bound to any Pod, and it's available for use by other Pods.

- If the PVC is deleted, the associated PV (if there is one) will be released. This means the storage resources associated with the PV will become available for use by other PVCs.

---
