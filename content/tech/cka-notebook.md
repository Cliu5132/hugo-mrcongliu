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
