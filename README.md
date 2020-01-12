# CKA
Preparation for Cloud Native Certified Kubernetes Administrator

- [Understanding Kubernetes Architecture](#understanding-kubernetes-architecture)
    - [Kubernetes Cluster Architecture](#kubernetes-cluster-architecture)
    - [Kubernetes API Primitives](#kubernetes-api-primitives)
    - [Kubernetes Services and Network Primitives](#kubernetes-services-and-network-primitives)
    - [Exploring the Kubernetes Cluster via the Command Line](#exploring-the-kubernetes-cluster-via-the-command-line)
- [Building the Kubernetes Cluster](#building-the-kubernetes-cluster)
    - [Release Binaries, Provisioning, and Types of Clusters](#release-binaries-provisioning-and-types-of-clusters)

## Understanding Kubernetes Architecture
### Kubernetes Cluster Architecture
- The Kubernetes architecture has two main roles: the master role and the worker role.
    - You can have one or many **master** nodes
    - You can have zero or mane **worker** nodes

![img](https://github.com/Bes0n/CKA/blob/master/images/img1.png)

- Here what these components are used for:

![img](https://github.com/Bes0n/CKA/blob/master/images/img2.png)

Working with **kubectl**:

- ``` kubectl get nodes ``` - see your nodes
```
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   64s   v1.17.0
```
  
- ``` kubectl get pods --all-namespaces ``` - see all pods running on all namespaces

```
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-6955765f44-dwqsq           1/1     Running   0          4m52s
kube-system   coredns-6955765f44-sfcxj           1/1     Running   0          4m52s
kube-system   etcd-minikube                      1/1     Running   0          4m59s
kube-system   kube-addon-manager-minikube        1/1     Running   0          4m58s
kube-system   kube-apiserver-minikube            1/1     Running   0          4m59s
kube-system   kube-controller-manager-minikube   1/1     Running   0          4m59s
kube-system   kube-proxy-zqvl8                   1/1     Running   0          4m52s
kube-system   kube-scheduler-minikube            1/1     Running   0          4m59s
kube-system   storage-provisioner                1/1     Running   0          4m50s
```

- ``` kubectl get pods --all-namespaces -o wide ``` - get detailed information about pods

- ```kubectl get namespaces``` - get your kubernetes **namespaces**
```
NAME              STATUS   AGE
default           Active   9m41s
kube-node-lease   Active   9m42s
kube-public       Active   9m42s
kube-system       Active   9m42s
```

This is how Application Running on Kubernetes looks like:

![img](https://github.com/Bes0n/CKA/blob/master/images/img3.png)

- Let's deploy simple POD with nginx image. JSON and YAML syntax accepted

```
cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
EOF
```

- ``` kubectl get pods ``` - get information about pods
```
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          6m1s
```

- ``` kubectl describe pod nginx ``` - get detailed information about **nginx** pod
```
Name:         nginx
Namespace:    default
Priority:     0
Node:         minikube/192.168.99.100
Start Time:   Sat, 11 Jan 2020 13:18:18 +0100
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           172.17.0.4
IPs:
  IP:  172.17.0.4
Containers:
  nginx:
    Container ID:   docker://c072f93d7911a772f7fc09f35d6142abc0e730aa5f273b0f3e9d0615a16f282f
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:8aa7f6a9585d908a63e5e418dc5d14ae7467d2e36e1ab4f0d8f9d059a3d071ce
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 11 Jan 2020 13:18:28 +0100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-b4lz9 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-b4lz9:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-b4lz9
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  7m10s  default-scheduler  Successfully assigned default/nginx to minikube
  Normal  Pulling    7m8s   kubelet, minikube  Pulling image "nginx"
  Normal  Pulled     7m     kubelet, minikube  Successfully pulled image "nginx"
  Normal  Created    7m     kubelet, minikube  Created container nginx
  Normal  Started    7m     kubelet, minikube  Started container nginx
```

- ``` kubectl delete pod nginx ``` - delete your **nginx** pod

![img](https://github.com/Bes0n/CKA/blob/master/images/img4.png)

### Kubernetes API Primitives
Every component in the Kubernetes system makes a request to the API server. The kubectl command line utility processes those API calls for us and allows us to format our request in a certain way. In this lesson, we will learn how Kubernetes accepts the instructions to run deployments and go through the YAML script that is used to tell the control plane what our environment should look like.
  
- ```kubectl get componentstatus``` - get your components status 
```
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true"}
```

YAML file with configuration of nginx-deployment:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

- ``` kubectl create -f nginx-deployment.yml ``` - create POD with configuration stored in **nginx-deployment.yml**

- ``` kubectl get deployment nginx-deployment -o yaml ``` - get information about your deployment in YAML format

- Definitions from YAML file:
    - **apiVersion** - Kubernetes API version, which indicates the path to ensure the API presents a clear, consistent view of system resources and behavior.
    - **kind** - Represents the kind of object you want to create. This is a required field. Examples of kinds of objects include pod, deployment, job, DaemonSet, ReplicaSet, ReplicationController, and more. 
    - **metadata** - Data that helpds uniquely identify the object, including a name string, UID, and optional namespace. 
    - **spec** - Describes your desired state for the object and the characteristics you want the object to have. The format of the object spec is different for every object and contains nested fields specific to that object. 
    - **spec (container)** - Specifies the pod's container image, volumes, and exposed ports for the container
    - **status** - Describes the actual state of the object and is supplied and updated by the Kubernetes system. At any given time, the Kubernetes Control Plane actively manages an object's actual state to match the desired state you supplied.

- ``` kubectl get pods --show-labels ``` - get your pod's labels
```
NAME                                READY   STATUS    RESTARTS   AGE    LABELS
nginx-deployment-54f57cf6bf-hddbx   1/1     Running   0          127m   app=nginx,pod-template-hash=54f57cf6bf
nginx-deployment-54f57cf6bf-wbw22   1/1     Running   0          127m   app=nginx,pod-template-hash=54f57cf6bf
```

- ```kubectl label pods nginx-deployment-54f57cf6bf-hddbx env=prod``` - add **prod** label to your pod
  
```
pod/nginx-deployment-54f57cf6bf-hddbx labeled
```
  
- ``` kubectl get pods -L env ``` - get pods with env label key
```
NAME                                READY   STATUS    RESTARTS   AGE    ENV
nginx-deployment-54f57cf6bf-hddbx   1/1     Running   0          135m   prod
nginx-deployment-54f57cf6bf-wbw22   1/1     Running   0          135m
```

- ``` kubectl annotate deployment nginx-deployment mysite.com/someannotation="isituseful" ``` - add additional annotation after deployment creation. 

- ``` kubectl get pods --field-selector status.phase=Running``` - filter objects based on some fields. In our case filter **pods** with phase status **running**

- ``` kubectl get services --field-selector metadata.namespace=default ``` - select services with default namespace

- ``` kubectl get pods --field-selector=status.phase=Running,metadata.namespace=default ``` - combination of two previous commands. 

- ``` kubectl get pods --field-selector=status.phase!=Running,metadata.namespace!=default ``` - reverve command, where status **is not** running and **not equal** to default namespace

By **label selectors** you can perform several actions on PODs which have these labels. You can edit, remove that labeled objects. 

![img](https://github.com/Bes0n/CKA/blob/master/images/img5.png)


### Kubernetes Services and Network Primitives
Kubernetes services allow you to dynamically access a group of replica pods without having to keep track of which pods are moved, changed, or deleted. In this lesson, we will go through how to create a service and communicate from one pod to another.

- ``` kubectl get pods -o wide``` - get wide information about pods.
```
NAME                                READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
nginx-deployment-54f57cf6bf-hddbx   1/1     Running   0          3h33m   172.17.0.5   minikube   <none>           <none>
nginx-deployment-54f57cf6bf-wbw22   1/1     Running   0          3h33m   172.17.0.4   minikube   <none>           <none>
```

- ``` kubectl delete pods nginx-deployment-54f57cf6bf-hddbx ``` - let's delete pod and see what will happen. 
```
NAME                                READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
nginx-deployment-54f57cf6bf-wbw22   1/1     Running   0          3h34m   172.17.0.4   minikube   <none>           <none>
nginx-deployment-54f57cf6bf-xlcdh   1/1     Running   0          18s     172.17.0.5   minikube   <none>           <none>
```

As a result we will have new pod running with new IP assigned on it. Here **service** takes place - dynamically access a group of replica pods without having to keep track of which pods are moved, changed, or deleted.
  
- Yaml file for creating **service** object looks like this:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  selector:
    app: nginx
```

- Where:
    - **kind** - type of deployment
    - **NodePort** - reservation of port for all nodes
    - **selector** - assign service to pod by selector, in our case by labels

- ``` kubectl create -f .\nginx-nodeport.yml``` - create service
- ``` kubectl get services ``` - get list of services 
- ``` kubectl get services nginx-nodeport ``` - get selected service - **nginx-nodeport**
```
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1      <none>        443/TCP        4h23m
nginx-nodeport   NodePort    10.96.241.16   <none>        80:30080/TCP   73s
```

![img](https://github.com/Bes0n/CKA/blob/master/images/img6.png)

![img](https://github.com/Bes0n/CKA/blob/master/images/img7.png)

- ``` kubectl create -f .\busybox.yml ``` Let's create **busybox** pod

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    args:
    - sleep
    - "1000"
```

- ``` kubectl exec busybox -- curl 10.96.241.16:80 ``` - execute command from **busybox POD** to **NodePort service**. Here one POD reached another POD by using **kube-proxy**

![img](https://github.com/Bes0n/CKA/blob/master/images/img8.png)


### Exploring the Kubernetes Cluster via the Command Line
- LAB answers:
  - ``` kubectl get nodes ``` - List all the nodes in the cluster.
  - ``` kubectl get pods --all-namespaces ``` - List all the pods in all namespaces.
  - ``` kubectl get namespaces ``` - List all the namespaces in the cluster.
  - ``` kubectl get pods ``` - Check to see if there are any pods running in the default namespace.
  - ``` kubectl get pods --all-namespaces -o wide ``` - Find the IP address of the API server running on the master node.
  - ``` kubectl get deployments ``` - See if there are any deployments in this cluster.
  - ``` kubectl get pods --all-namespaces --show-labels -o wide ``` - Find the label applied to the etcd pod on the master node.


## Building the Kubernetes Cluster
### Release Binaries, Provisioning, and Types of Clusters
There are many choices when it comes to choosing where to create your Kubernetes cluster. In this lesson, we will focus on what you need to know for the CKA exam — specifically, how the cluster build relates to the types of clusters you will face in the exam.

![img](https://github.com/Bes0n/CKA/blob/master/images/img9.png)
  
![img](https://github.com/Bes0n/CKA/blob/master/images/img10.png)

### Installing Kubernetes Master and Nodes
Now that we’ve considered all the different types of clusters and where to locate the tools we need to install a cluster, let’s get to it! In this lesson, we will go through the all steps to install a three-node cluster.

![img](https://github.com/Bes0n/CKA/blob/master/images/img11.png)

Get the Docker gpg key:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add the Docker repository:
```
sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

Get the Kubernetes gpg key:
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Add the Kubernetes repository:
```
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

Update your packages:
```
sudo apt-get update
```

Install Docker, kubelet, kubeadm, and kubectl:
```
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu kubelet=1.13.5-00 kubeadm=1.13.5-00 kubectl=1.13.5-00
```

Hold them at the current version:
```
sudo apt-mark hold docker-ce kubelet kubeadm kubectl
```

Add the iptables rule to **sysctl.conf**:
```
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
```

Enable iptables immediately:
```
sudo sysctl -p
```

Initialize the cluster (run only on the master):
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Set up local kubeconfig (only master node):
```
mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Apply Flannel CNI network overlay:
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Join the worker nodes to the cluster:
```
sudo kubeadm join [your unique string from the kubeadm init command]
```

Verify the worker nodes have joined the cluster successfully:
```
kubectl get nodes
```

Compare this result of the kubectl get nodes command:
```
NAME                            STATUS   ROLES    AGE   VERSION
chadcrowell1c.mylabserver.com   Ready    master   4m18s v1.13.5
chadcrowell2c.mylabserver.com   Ready    none     82s   v1.13.5
chadcrowell3c.mylabserver.com   Ready    none     69s   v1.13.5
```