# What do I need to start with Kubernetes?

This book is a mix of theoretical explanations and practical examples. You’ll get the most out of this book if you follow along with the practical examples. To do that, you will need the following tools:

- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- Access to a Kubernetes cluster, see [Which Kubernetes cluster should I use?](#_bookmark1)
- [Kubernetes CLI (**kubectl**)](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

For some sections, you might also need other tools that I mention in the specific chapters.

# Which Kubernetes cluster should I use?

You have multiple choices. The most 'real-world' option would be to get a Kubernetes cluster from one cloud provider. However, for numerous reasons, that might not be an option for everyone.

The next best option is to run a Kubernetes cluster on your computer. Assuming you have some memory and CPU to spare, you can use one of the following tools to run a single-node Kubernetes cluster on your computer:

- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

You could go with any of the above options. Creating Kubernetes ReplicaSets, Deployments, and Pods works with any of them. You can also create Kubernetes Services. However, things get a bit complicated when you’re trying to use a LoadBalancer service type.

With the cloud-managed cluster, creating a LoadBalancer service type creates an actual instance of the load balancer, and you would get an external/public IP address you can use to access your services.

The one solution from the above list closest to simulating the LoadBalancer service type is Docker Desktop. With [Docker Desktop](https://www.docker.com/products/docker-desktop) your service gets exposed on an external IP, **localhost**. You can access these services using both [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) and [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) as well; however, it requires you to run additional commands.

For that purpose, I’ll be mostly using [Docker Desktop](https://www.docker.com/products/docker-desktop). I’ll specifically call out when I am using either Minikube or a cloud-managed cluster. You can follow the installation instructions for all options from their respective websites.

# Kubernetes and contexts

After you’ve installed one of these tools, make sure you download the [Kubernetes CLI (**kubectl**)](https://kubernetes.io/docs/tasks/tools/install-kubectl/). Kubernetes CLI is a single binary called **kubectl**, and it allows you to run commands against your cluster. To make sure everything is working correctly, you can run **kubectl get nodes** to list all nodes in the Kubernetes cluster. The output from the command should look like this:

```
$ kubectl get nodes
NAME	STATUS	ROLES	AGE	VERSION
docker-desktop	Ready	master	63d	v1.16.6-beta.0
```

You can also check that the context is set correctly to **docker-desktop**. Kubernetes uses  a  configuration file called **config** to find the information it needs to connect to the cluster. Kubernetes CLI reads this file from your home folder - for example **$HOME/.kube/config**. Context is an element inside that config file, and it contains a reference to the cluster, namespace, and the user. If you’re accessing or running a single cluster, you will only have one context in the config file. However, you can have multiple contexts defined that point to different clusters.

Using the **kubectl config** command, you can view these contexts and switch between them. You can run the **current-context** command to view the current context:

```
$ kubectl config current-context
 docker-desktop
 ```

 There are other commands such as **use-context**, **set-context**, **view-contexts**, etc. I prefer to use a tool called kubectx. This tool allows you to switch between different Kubernetes contexts quickly. For example, if I have three clusters (contexts) set in the config file, running **kubectx** outputs this:

 ```
 $ kubectx
  docker-
  desktop 
  peterj-cluster 
  minikube
  ```

The tool highlights the currently selected context when you run the command. To switch to the
**minikube** context, I can run: **kubectx minikube**.

The equivalent commands you can run with **kubectl** would be **kubectl config get-contexts** to view  all contexts, and **kubectl config use-context minikube** to switch the context.

Before you continue, make sure your context is set to **docker-desktop** if you’re using Docker for Mac/Windows or **minikube** if you’re using Minikube.

# What is container orchestration?

Containers are everywhere these days. People use tools such as [Docker](https://docker.com/) for packaging anything from applications to databases. With the growing popularity of microservice architecture and moving away from the monolithic applications, a monolith application is now a collection of multiple smaller services.

Managing a single application has its issues and challenges, let alone managing tens of smaller services that have to work together. You need a way to automate and manage your deployments, figure out how to scale individual services, use the network, connect them, and so on.

The container orchestration can help you do this. Container orchestration can help you manage the lifecycles of your containers. Using a container orchestration system allows you to do the following:

- Provision and deploy containers based on available resources
- Perform health monitoring on containers
- Load balancing and service discovery
- Allocate resources between different containers
- Scaling the containers up and down

A couple of examples of container orchestrators are [Marathon](https://mesosphere.github.io/marathon/), [Docker Swarm](https://docs.docker.com/get-started/swarm-deploy/) and the one discussed in this course, [Kubernetes](https://kubernetes.io/).

[Kubernetes](https://kubernetes.io/) is an open-source project and one of the popular choices for cluster management and scheduling container-centric workloads. You can use Kubernetes to run your containers, do zero- downtime deployments where you can update your application without impacting your users, and bunch of other cool stuff.

![Kubernetes](https://i.gyazo.com/cebe3325d26cb2ded19a216f4b22c385.png)

**NOTE**:

Frequently, people refer to Kubernetes as "K8S". K8S is a __numeronym__ for Kubernetes. The first (K) and the last letter (S) are the first, and the last letters in the word Kubernetes, and 8 is the number of characters between those two letters. Other popular numeronyms are "i18n" for internationalization or "a11y" for accessibility.

# What is the difference Kubernetes and Docker?

Using Docker, you can package your application. This package is called an image or a Docker image. You can think of an image as a template. Using Docker, you can create containers from  your images. For example, if your Docker image contains a Go binary or a Java application, then the container is a running instance of that application. If you want to learn more about Docker, check out the [Beginners Guide to Docker](https://www.learncloudnative.com/blog/2020-04-29-beginners-guide-to-docker/).

Kubernetes, on the other hand, is a container orchestration tool that knows how to manage Docker (and other) containers. Kubernetes uses higher-level constructs such as Pods to wrap Docker (or other) containers and gives you the ability to manage them.

# Kubernetes vs. Docker Swarm?

Docker Swarm is a container orchestration tool, just like Kubernetes is. You can use it to manage Docker containers. Using Swarm, you can connect multiple Docker hosts into a virtual host. You can then use Docker CLI to talk to multiple hosts at once and run Docker containers on it.

# Kubernetes architecture

A Kubernetes cluster is a set of physical or virtual machines and other infrastructure resources needed to run your containerized applications. Each machine in a Kubernetes cluster is called a __node__. There are two types of node in each Kubernetes cluster:

- Master node(s): this node hosts the Kubernetes control plane and manages the cluster
- Worker node(s): runs your containerized applications

![Kubernetes architecture](https://i.gyazo.com/02806f2e435cc399aa8a99113d8debe6.png)
*Figure 2. Kubernetes Architecture*

## Master nodes

One of the main components on the master node is called the __API server__. The API server is the
endpoint that Kubernetes CLI (**kubectl**) talks to when you’re creating Kubernetes resources or
managing the cluster.

The __scheduler component__ works with the API server to schedule the applications or workloads on
to the worker nodes. It also knows about resources available on the nodes and the resources
requested by the workloads. Using this information, it can decide on which worker nodes your
workloads end up.

![Kubernetes Master Node](https://i.gyazo.com/ad589c1a7c799fb42c9cab1d84874fd1.png)
*Figure 3. Kubernetes Master Node*

There are two types of controller managers running on master nodes.

The **kube controller manager** runs multiple controller processes. These controllers watch the state
of the cluster and try to reconcile the current state of the cluster (e.g., "5 running replicas of
workload A") with the desired state (e.g. "I want ten running replicas of workload A"). The
controllers include a node controller, replication controller, endpoints controller, service account
and token controllers.

The **cloud controller manager** runs controllers specific to the cloud provider and can manage
resources outside of your cluster. This controller only runs if your Kubernetes cluster is running in
the cloud. If you’re running Kubernetes cluster on your computer, this controller won’t be running.
The purpose of this controller is for the cluster to talk to the cloud providers to manage the nodes,
load balancers, or routes.

Finally, [etcd](https://etcd.io/) is a distributed key-value store. Kubernetes stores the state of the cluster and API in the
etcd.

## Worker nodes

Just like on the master node, worker nodes have different components running as well. The first
one is __kubelet__. This service runs on each worker node, and its job is to manage the container. It
makes sure containers are running and healthy, and it connects back to the control plane. Kubelet
talks to the API server, and it is responsible for managing resources on the node it’s running on.

When you add a new worker node to the cluster, the kubelet introduces itself to the API server and
provides its resources (e.g., "I have X CPU and Y memory"). Then, it asks the API server if there are
any containers to run. You can think of the kubelet as a worker node manager.

Kubelet uses the container runtime interface (CRI) to talk to the container runtime. The container
runtime is responsible for working with the containers. In addition to Docker, Kubernetes also
supports other container runtimes, such as [containerd](https://containerd.io/) or [cri-o](https://cri-o.io/).

![ Kubernetes Worker Node](https://i.gyazo.com/9b9fe83059fa0b8997ed5bf3593e6ea7.png)
*Figure 4. Kubernetes Worker Node*

The containers are running inside pods, represented by the blue rectangles in the above figure (containers are the red rectangles inside each pod). A pod is the smallest deployable unit you can create, schedule, and manage on a Kubernetes cluster. A pod is a logical collection of containers that make up your application. The containers running inside the same pod also share the network and storage space.

Each worker node also has a proxy that acts as a network proxy and a load balancer for workloads
8
running on the worker nodes. Through these proxies, the external load balancer redirects client
requests to containers running inside the pod.

# Kubernetes Resources

The Kubernetes API defines many objects called resources, such as namespaces, pods, services,
secrets, config maps, etc.

Of course, you can also define your custom resources using the custom resource definition or CRD.

After you’ve configured Kubernetes CLI and your cluster, you should try and run **kubectl api-resources** command. It will list all defined resources - there will be a lot of them.

Resources in Kubernetes can be defined using YAML. People commonly use YAML (YAML Ain’t
Markup Language) for configuration files. It is a superset of JSON format, which means you can also
use JSON to describe Kubernetes resources.

Every Kubernetes resource has an **apiVersion** and **kind** fields to describe which version of the
Kubernetes API you’re using when creating the resource (for example, **apps/v1**) and what kind of a
resource you are creating (for example, **Deployment**, **Pod**, **Service**, etc.).

The **metadata** includes the data that can help to identify the resource you are creating. Commonly
found fields in the metadata section include the **name** (for example **mydeployment**) and the **namespace**
where Kubernetes creats the resource.

Other fields you can have in the metadata section are **labels** and **annotations**. Kubernetes also adds
a couple of fields automatically after creating the resource (such as **creationTimestamp**, for example).

## Labels and selectors

You can use labels (key/value pairs) to label resources in Kubernetes. They are used to organize,
query, and select objects and attach identifying metadata to them. You can specify labels at creation
time or add, remove, or update them later after you’ve created the resource.

![Figure 5. Kubernetes Labels](https://i.gyazo.com/892035b7229d276aeaf54633ced5115e.png)
*Figure 5. Kubernetes Labels*

The labels have two parts: the key name and the value. The key name can have an optional prefix
and the name, separated by a slash (**/**).

The **startkuberenetes.com** portion in the figure is the optional prefix, and the key name is **app-name**.
The prefix, if specified, must be a series of DNS labels separated by dots (.). It can’t be longer than
253 characters.

The key name portion is required and must be 63 characters or less. It has to start and end with an
alphanumeric character. The key name is made of alphanumerical values and can include dashes (-
), underscores (_), and dots (.).

Just like the key name, a valid label value must be 63 characters or less. It can be empty or being
and end with an alphanumeric character and can include dashes (-), underscores (_), and dots (.).

Here’s an example of a couple of valid labels on a Kubernetes resource:
```
metadata:
  labels:
  startkubernetes.com/app-name: my-application
  blog.startkubernetes.com/version: 1.0.0
  env: staging-us-west
  owner: ricky
  department: AB1
```
__Selectors__ are used to query for a set of Kubernetes resources. For example, you could use a selector
to identify all Kubernetes cluster objects with a label **env** set to **staging-us-west**. You could write that
selector as **env = staging-us-west**. This selector is called an equality-based selector.

Selectors also support multiple requirements. For example, we could query for all resources with
the **env** label set to **staging-us-west** and are not of version **1.0.0**. The requirements have to be
separated by commas that act as a logical AND operator. We could write the above two
requirements like this: **env = staging-us-west, blog.startkubernetes.com/version != 1.0.0**.

The second type of selector is called set-based selectors. These selectors allow filtering label keys
based on a set of values. The following three operators are supported: in, notin, and exists. Here’s
an example:
```
env in (staging-us-west,staging-us-east)
owner notin (ricky, peter)
department
department!
```
he first example selects all objects with the **env** label set to either **staging-us-west** or **staging-us-east**. The second example uses the **notin** operator and selects all objects where the **owner** label
values are not **ricky** or **peter**. The third example selects all objects with a **department** label set,
regardless of it’s value, and the last example selects all objects without the department label set.

Later on, we will see the practical use of labels and selectors when discussing how Kubernetes
services know which pods to include in their load balancing pools.

Labeling resources is essential, so make sure you take your time to decide on the core set of labels
you will use in your organization. Setting labels on all resources make it easier to do bulk
operations against them later on.

Kubernetes provides a list of recommended labels that share a common prefix **app.kubernetes.io**:

*Table 1. Recommended Labels*

| Name | Value | Description |
|-----|-----|-----|
| **app.kubernetes.io/name** |  my-app | Application name |
| **app.kubernetes.io/instance** | my-app-1122233 | Identifying instance of the application |
| **app.kubernetes.io/version** | 1.2.3 | Application version |
| **app.kubernetes.io/component** | website | The name of the component |
| **app.kubernetes.io/part-of** | carts | The name of the higher-level application this component is part of |
| **app.kubernetes.io/managed-by** | helm | The tools used for managing the application |

Additionally, you could create an maintain a list of your own labels:

- Project ID (**carts-project-123**)
- Owner (**Ricky**, **Peter**, or **team-a**, **team-b**, …)
- Environment (**dev**, **test**, **staging**, **prod**})
- Release identifer (**release**-1.0.0`)
- Tier (**backend**, **frontend**)

## Annotations

Annotations are similar to labels as they also add metadata to Kubernetes objects. However, you
don’t use annotations for identifying and selecting objects. The annotations allow you to add non-identifying metadata to Kubernetes objects. Examples would be information needed for debugging,
emails, or contact information of the on-call engineering team, port numbers or URL paths used by
monitoring or logging systems, and any other information that might be used by the client tools or
DevOps teams.

For example, annotations are used by ingress controllers to claim the Ingress resource ([see
Exposing multiple applications with Ingress]()).

Similar to labels, annotations are key/value pairs. The key name has two parts and the same rules
apply as with the label key names.

However, the annotation values can be both structured or unstructured and can include characters
that are not valid in the labels.

```
metadata:
  annotations:
  startkubernetes.com/debug-info: |
    { "debugTool": "sometoolname", "portNumber": "1234", "email":
"hello@example.com" }
```
The above example defines an annotation called **startkubernetes.com/debug-info** that contains a
JSON string.

## Working with Pods

Pods are probably one of the most common resources in Kubernetes. They are a collection of one or
more containers. The containers within the Pod share the same network and storage. That means
any containers within the same Pod can talk to each other through **localhost** and access the same
Volumes.

You always design your Pods as temporary, disposable entities that can get deleted and rescheduled
to run on different nodes. Whenever you or Kubernetes restart the Pod, you are also restarting
containers within the Pod. We will discuss how to persist data between Pod restarts in [Stateful
Workloads](), [What are Volumes?](), and [Creating Persistent Volumes]().

![Figure 6. Kubernetes Pod](https://i.gyazo.com/a547b2339615414f8f842d734a1a9a77.png)
*Figure 6. Kubernetes Pod*

When created, each Pod gets assigned a unique IP address. The containers inside your Pod can
listen to different ports. To access your containers, you can use the Pods' IP address. Using the
example from the above figure, you could run **curl 10.1.0.1:3000** to talk to the one container and
**curl 10.1.0.1:5000** to talk to the other container. However, if you wanted to talk between
containers - for example, calling the top container from the bottom one, you could use
http://localhost:3000.

If your Pod restarts, it will get a different IP address. Therefore, you cannot rely on the IP address.
Talking to your Pods directly by the IP is not the right way to go.

An abstraction called a Kubernetes Service is what you can to communicate with your Pods. A
Kubernetes Service gives you a stable IP address and DNS name. I’ll talk about services later on.

## Scaling Pods

All containers within the Pod get scaled together. The figure below shows how scaling from a single
Pod to four Pods would look like. Note that you cannot scale individual containers within the Pods.
The Pod is the unit of scale, which means that whenever you scale a Pod, you will scale all
containers inside the Pod as well.

![Scaling Pods](https://i.gyazo.com/edd1fe8e1d0bd188930cc96ba02dc905.png)

__WARNING__  
"Awesome! I can run my application and a database in the same Pod!!" No! Do not do that.

First, in most cases, your database will not scale at the same rate as your application. Remember,
you’re scaling a Pod and all containers inside that Pod, not just a single container.

Second, running a stateful workload in Kubernetes, such as a database, differs from running
stateless workloads. For example, you need to ensure that data is persistent between Pod restarts
and that the restarted Pods have the same network identity. You can use resources like persistent
volumes and stateful sets to accomplish this. We will discuss running stateful workloads in
Kubernetes in [Stateful Workloads]() section.

## Creating Pods

Usually, you shouldn’t be creating Pods manually. You can do it, but you really should not. The
reason being is that if the Pod crashes or if it gets deleted, it will be gone forever. That said,
throughout this book, we will be creating Pods directly for the sake of simplicity and purposes of
learning and explaining different concepts. However, if you’re planning to run your applications
inside Kubernetes, make sure you aren’t creating Pods manually.

Let’s look at how a single Pod can be defined using YAML.

_ch3/pod.yaml_
```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
  labels:
    app.kubernetes.io/name: hello
spec:
  containers:
    - name: hello-container
      image: busybox
      command: ["sh", "-c", "echo Hello from my container! && sleep 3600"]
```

In the first couple of lines, we define the kind of resource (**Pod**) and the metadata. The metadata
includes the name of our Pod (**hello-pod**) and a set of labels that are simple key-value pairs
(**app.kubernetes.io/name=hello**).

In the **spec** section, we are describing how the Pod should look. We will have a single container
inside this Pod, called **hello-container**, and it will run the image called **busybox**. When the container
starts, it executes the command defined in the **command** field.

To create the Pod, you can save the above YAML to a file called **pod.yaml**. Then, you can use
Kubernetes CLI (**kubectl**) to create the Pod:

```
$ kubectl apply -f pod.yaml
pod/hello-pod created
```
Kubernetes responds with the resource type and the name it created. You can use **kubectl get pods**
to get a list of all Pods running the **default** namespace of the cluster.

```
$ kubectl get pods
NAME READY STATUS RESTARTS AGE
hello-pod 1/1 Running 0 7s
```
You can use the **logs** command to see the output from the container running inside the Pod:

```
$ kubectl logs hello-pod
Hello from my container!
```
__TIP__:

When you have multiple containers running inside the same Pod, you can use the **-c**
flag to specify the container name whose logs you want to get. For example: **kubectl
logs hello-pod -c hello-container**

If we delete this Pod using kubectl **delete pod hello-pod**, the Pod will be gone forever. There’s
nothing that would automatically restart it. If you run the **kubectl get pods** again, you will notice
15 the Pod is gone:

```
$ kubectl get pod
No resources found in default namespace.
```
Not automatically recreating the Pod is the opposite of the behavior you want. If you have your
containers running in a Pod, you would want Kubernetes to reschedule and restart them if
something goes wrong automatically.

To make sure the crashed Pods get restarted, you need a **controller** to manage the Pods' lifecycle.
This controller automatically reschedules your Pod if it gets deleted or if something terrible
happens (cluster nodes go down and Kubernetes need to evict the Pods).

## Managing Pods with ReplicaSets

The job of a ReplicaSet is to maintain a stable number of Pod copies. The word __replicas__ is often
used to refer to the number of Pod copies. The ReplicaSet controller guarantees that a specified
number of identical Pods (or replicas) is running . The number of replicas is controlled by the
**replicas** field in the Pod resource definition.

Let’s say you start with a single Pod, and you want to scale to five Pods. The single Pod is the
current state in the cluster, and the five Pods is the desired state. The ReplicaSet controller uses the
current and desired state and goes to create four more Pods to meet the desired state (five pods).
The ReplicaSet also keeps an eye on the Pods. If you scale the Pod up or down (add a Pod replica or
remove a Pod replica), the ReplicaSet does the necessary to meet the desired number of replicas. To
create the Pods, the ReplicaSet uses the Pod template that’s part of the resource definition.

__But how does ReplicaSet know which Pods to watch and control?__

To explain that, let’s look at how the ReplicaSet gets defined:

_ch3/replicaset.yaml_
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: hello
  labels:
    app.kubernetes.io/name: hello
spec:
  replicas: 5
  selector:
    matchLabels:
      app.kubernetes.io/name: hello
  template:
    metadata:
      labels:
        app.kubernetes.io/name: hello
    spec:
      containers:
        - name: hello-container
          image: busybox
          command: ['sh', '-c', 'echo Hello from my container! && sleep 3600']
```

Every Pod that’s created by a ReplicaSet has a field called **metadata.ownerReferences**. This field
specifies which ReplicaSet owns the Pod. When you delete a Pod, the ReplicaSet that owns it will
know about it and act accordingly (i.e., re-creates the Pod).

The ReplicaSet also uses the **selector** object and **matchLabel** to check for any new Pods that it might
own. If there’s a new Pod that matches the selector labels and it doesn’t have an owner reference,
or the owner is not a controller (i.e., if we manually created a pod), the ReplicaSet will take it over
and start controlling it.

![Figure 7. ReplicaSet Details](https://i.gyazo.com/83cc97b0c8fabf44d06f4cd86a50175d.jpg)
_Figure 7. ReplicaSet Details_

Let’s save the above contents in replicaset.yaml file and run:
```
$ kubectl apply -f replicaset.yaml
replicaset.apps/hello created
```
You can view the ReplicaSet by running the following command:
```
$ kubectl get replicaset
NAME  DESIRED CURRENT READY AGE
hello 5       5       5     30s
```
The command will show you the name of the ReplicaSet and the number of desired, current, and
ready Pod replicas. If you list the Pods, you will notice that five Pods are running:

```
$ kubectl get po
NAME        READY   STATUS    RESTARTS  AGE
hello-dwx89 1/1     Running   0         31s
hello-fchvr 1/1     Running   0         31s
hello-fl6hd 1/1     Running   0         31s
hello-n667q 1/1     Running   0         31s
hello-rftkf 1/1     Running   0         31s
```
You can also list the Pods by their labels. For example, if you run **kubectl get po
-l=app.kubernetes.io/name=hello**, you will get all Pods that have **app.kubernetes.io/name: hello**
label set. The output includes the five Pods we created.

Let’s also look at the **ownerReferences** field. We can use the **-o yaml** flag to get the YAML
representation of any object in Kubernetes. Once we get the YAML, we will search for the
**ownerReferences** string:

```
$ kubectl get po hello-dwx89 -o yaml | grep -A5 ownerReferences
...
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: hello
```

In the **ownerReferences**, Kubernetes sets the **name** to **hello**, and the **kind** to **ReplicaSet**. The name
corresponds to the ReplicaSet that owns the Pod.

__TIP__:

Notice how we used **po** in the command to refer to Pods? Some Kubernetes resources
have short names you can use in place of the 'full names'. You can use **po** when you
mean **pods** or **deploy** when you mean **deployment**. To get the full list of supported short
names, you can run **kubectl api-resources**.

Another thing you will notice is how the Pods are named. Previously, where we were creating a
single Pod directly, the name of the Pod was **hello-pod**, because that’s what we specified in the
YAML. When using the ReplicaSet, Pods are created using the combination of the ReplicaSet name
(**hello**) and a semi-random string such as **dwx89**, **fchrv** and so on.

__NOTE__:

Semi-random? Yes, Kubernetes maintainers removed the [vowels](https://github.com/kubernetes/kubernetes/pull/37225), and [numbers 0,1,
and 3](https://github.com/kubernetes/kubernetes/pull/50070) to avoid creating 'bad words'.

Let’s try and delete one of the Pods. To **delete** a resource from Kubernetes you use the delete
keyword followed by the resource (e.g. **pod**) and the resource name **hello-dwx89**:

```
$ kubectl delete po hello-dwx89
pod "hello-dwx89" deleted
```

Once you’ve deleted the Pod, you can run **kubectl get pods** again. Did you notice something? There
are still five Pods running.

```
$ kubectl get po
NAME          READY     STATUS      RESTARTS    AGE
hello-fchvr   1/1       Running     0           14m
hello-fl6hd   1/1       Running     0           14m
hello-n667q   1/1       Running     0           14m
hello-rftkf   1/1       Running     0           14m
hello-vctkh   1/1       Running     0           48s
```
If you look at the **AGE** column, you will notice four Pods created 14 minutes ago and one created
more recently. ReplicaSet created this new Pod. When we deleted one Pod, the number of actual
replicas decreased from five to four. The replica set controller detected that and created a new Pod
to match the replicas' desired number (5).

Let’s try something different now. We will manually create a Pod with labels that match the
ReplicaSets selector labels (**app.kubernetes.io/name: hello**).

_ch3/stray-pod.yaml_
```
apiVersion: v1
kind: Pod
metadata:
  name: stray-pod
  labels:
    app.kubernetes.io/name: hello
spec:
  containers:
    - name: stray-pod-container
      image: busybox
      command: ["sh", "-c", "echo Hello from stray pod! && sleep 3600"]
```
Save the above YAML in the **stray-pod.yaml** file, then create the Pod by running:

```
$ kubectl apply -f stray-pod.yaml
pod/stray-pod created
```

The Pod gets created, and all looks good. However, if you run **kubectl get pods** you will notice that
the **stray-pod** has dissapeared. What happened?

The ReplicaSet makes sure only five Pod replicas are running. When we manually created the
**stray-pod** with the label **app.kubernetes.io/name=hello** that matches the selector label on the
ReplicaSet, the ReplicaSet took that new Pod for its own. Remember, manually created Pod didn’t
have the owner. With this new Pod under ReplicaSets' management, the number of replicas was six
and not five, as stated in the ReplicaSet. Therefore, the ReplicaSet did what it’s supposed to do; it
deleted the new Pod to maintain the desired state of five replicas.

__Zero-downtime updates?__

I mentioned zero-downtime deployments and updates earlier. How can that be done using a replica
set? Well, it you can’t do it with a replica set. At least not in a zero-downtime manner.

Let’s say we want to change the Docker image used in the original replica set from **busybox** to
**busybox:1.31.1**. We could use **kubectl edit rs hello** to open the replica set YAML in the editor, then
update the image value.

Once you save the changes - nothing will happen. Five Pods will still keep running as if nothing has
happened. Let’s check the image used by one of the Pods:

```
$ kubectl describe po hello-fchvr | grep   image
  Normal Pulling      14m         kubelet, docker-desktop Pulling image "busybox"
  Normal Pulled       13m         kubelet, docker-desktop Successfully pulled image
"busybox"
```

Notice it’s referencing the **busybox** image, but there’s no sign of the **busybox:1.31.1** anywhere. Let’s see what happens if we delete this same Pod:

```
$ kubectl delete po hello-fchvr
pod "hello-fchvr" deleted
```

We already know that ReplicaSet will bring up a new Pod (**hello-q8fnl** in our case) to match the
desired replica count from the previous test we did. If we run **describe** against the new Pod that
came up, you will notice how the image is changed this time:

```
$ kubectl describe po hello-q8fnl | grep image
  Normal  Pulling     74s        kubelet, docker-desktop Pulling image "busybox:1.31"
  Normal  Pulled      73s        kubelet, docker-desktop Successfully pulled image
"busybox:1.31"
```

A similar thing will happen if we delete the other Pods that are still using the old image (**busybox**).
The ReplicaSet would start new Pods, and this time, the Pods would use the new image
**busybox:1.31.1**.

We can use another resource to manage the ReplicaSets, allowing us to update Pods in a controlled
manner. Upon changing the image name, it can start Pods using the new image names in a
controlled way. This resource is called a __Deployment__.

To delete all Pods, you need to delete the ReplicaSet by running: **kubectl delete rs hello. rs** is the
short name for replicaset. If you list the Pods (kubectl get po) right after you issued the delete
command, you will see the Pods getting terminated:

```
NAME          READY   STATUS        RESTARTS  AGE
hello-fchvr   1/1     Terminating   0         18m
hello-fl6hd   1/1     Terminating   0         18m
hello-n667q   1/1     Terminating   0         18m
hello-rftkf   1/1     Terminating   0         18m
hello-vctkh   1/1     Terminating   0         7m39s
```
Once the replica set terminates all Pods, they will be gone, and so will be the ReplicaSet.

## Creating Deployments

A deployment resource is a wrapper around the ReplicaSet that allows doing controlled updates to your Pods. For example, if you want to update image names for all Pods, you can edit the Pod template, and the deployment controller will re-create Pods with the new image.

If we continue with the same example as we used before, this is how a Deployment would look like:

_ch3/deployment.yaml_
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
  labels:
    app.kubernetes.io/name: hello
spec:
  replicas: 5
  selector:
    matchLabels:
      app.kubernetes.io/name: hello
  template:
    metadata:
      labels:
        app.kubernetes.io/name: hello
    spec:
      containers:
        - name: hello-container
          image: busybox
          command: ["sh", "-c", "echo Hello from my container! && sleep 3600"]
```

The YAML for Kubernetes Deployment looks almost the same as for a ReplicaSet. There’s the replica
count, the selector labels, and the Pod template.

Save the above YAML contents in **deployment.yaml** and create the deployment:
```
$ kubectl apply -f deployment.yaml --record
deployment.apps/hello created
```

**NOTE**:

Why the **--record** flag? Using this flag, we are telling Kubernetes to store the
command we executed in the annotation called **kubernetes.io/change-cause**. Record
flag is useful to track the changes or commands that you executed when the
deployment was updated. You will see this in action later on when we do rollouts.

To list all deployments, we can use the get command:

```
$ kubectl get deployment
NAME    READY   UP-TO-DATE    AVAILABLE   AGE
hello   5/5     5             5           2m8s
```

The output is the same as when we were listing the ReplicaSets. When we create the deployment,
controller also creates a ReplicaSet:

```
$ kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
hello-6fcbc8bc84  5         5         5       3m17s
```

Notice how the ReplicaSet name has the random string at the end. Finally, let’s list the Pods:

```
$ kubectl get po
NAME                    READY   STATUS      RESTARTS      AGE
hello-6fcbc8bc84-27s2s  1/1     Running     0             4m2s
hello-6fcbc8bc84-49852  1/1     Running     0             4m1s
hello-6fcbc8bc84-7tpvs  1/1     Running     0             4m2s
hello-6fcbc8bc84-h7jwd  1/1     Running     0             4m1s
hello-6fcbc8bc84-prvpq  1/1     Running     0             4m2s
```

When we created a ReplicaSet previously, the Pods got named like this: **hello-fchvr**. However, this
time, the Pod names are a bit longer - **hello-6fcbc8bc84-27s2s**. The random middle section in the
name **6fcbc8bc84** corresponds to the random section of the ReplicaSet name, and the Pod names get
created by combining the deployment name, ReplicaSet name, and a random string.

_Figure 8. Deployment, ReplicaSet, and Pod naming_

Just like before, if we delete one of the Pods, the Deployment and ReplicaSet will make sure awalys
to maintain the number of desired replicas:

```
$ kubectl delete po hello-6fcbc8bc84-27s2s
pod "hello-6fcbc8bc84-27s2s" deleted

$ kubectl get po
NAME                      READY     STATUS    RESTARTS    AGE
hello-6fcbc8bc84-49852    1/1       Running   0           46m
hello-6fcbc8bc84-58q7l    1/1       Running   0           15s
hello-6fcbc8bc84-7tpvs    1/1       Running   0           46m
hello-6fcbc8bc84-h7jwd    1/1       Running   0           46m
hello-6fcbc8bc84-prvpq    1/1       Running   0           46m
```

### **How to scale the Pods up or down?**

There’s a handy command in Kubernetes CLI called **scale**. Using this command, we can scale up (or
down) the number of Pods controlled by the Deployment or a ReplicaSet.

Let’s scale the Pods down to three replicas:

```
$ kubectl scale deployment hello --replicas=3
deployment.apps/hello scaled

$ kubectl get po
NAME                    READY   STATUS    RESTARTS    AGE
hello-6fcbc8bc84-49852  1/1     Running   0           48m
hello-6fcbc8bc84-7tpvs  1/1     Running   0           48m
hello-6fcbc8bc84-h7jwd  1/1     Running   0           48m
```

Similarly, we can increase the number of replicas back to five, and ReplicaSet will create the Pods.

```
$ kubectl scale deployment hello --replicas=5
deployment.apps/hello scaled

$ kubectl get po
NAME                      READY     STATUS    RESTARTS    AGE
hello-6fcbc8bc84-49852    1/1       Running   0           49m
hello-6fcbc8bc84-7tpvs    1/1       Running   0           49m
hello-6fcbc8bc84-h7jwd    1/1       Running   0           49m
hello-6fcbc8bc84-kmmzh    1/1       Running   0           6s
hello-6fcbc8bc84-wfh8c    1/1       Running   0           6s
```

### Updating the Pod templates

When we were using a ReplicaSet we noticed that ReplicaSet did not automatically restart the Pods when we updated the image name. However, Deployment can do this.

Let’s use the **set image** command to update the image in the Pod templates from **busybox** to
**busybox:1.31.1**.

__TIP__:

You can use the set command to update the parts of the Pod template, such as image name, environment variables, resources, and a couple of others.

```
$ kubectl set image deployment hello hello-container=busybox:1.31.1 --record
deployment.apps/hello image updated
```
If you run the **kubectl get pods** right after you execute the **set** command, you might see something
like this:

```
$ kubectl get po
NAME                      READY         STATUS        RESTARTS  AGE
hello-6fcbc8bc84-49852    1/1           Terminating   0         57m
hello-6fcbc8bc84-7tpvs    0/1           Terminating   0         57m
hello-6fcbc8bc84-h7jwd    1/1           Terminating   0         57m
hello-6fcbc8bc84-kmmzh    0/1           Terminating   0         7m15s
hello-84f59c54cd-8khwj    1/1           Running       0         36s
hello-84f59c54cd-fzcf2    1/1           Running       0         32s
hello-84f59c54cd-k947l    1/1           Running       0         33s
hello-84f59c54cd-r8cv7    1/1           Running       0         36s
hello-84f59c54cd-xd4hb    1/1           Running       0         35s
```

The controller terminated the Pods and has started five new Pods. Notice something else in the Pod
names? The ReplicaSet section looks different, right? That’s because Deployment scaled down the
Pods controlled by the previous ReplicaSet and create a new ReplicaSet that uses the latest image
we defined.

Remember that **--record** flag we set? We can now use **rollout history** command to view the
previous rollouts.

```
$ kubectl rollout history deploy hello
deployment.apps/hello
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=deployment.yaml --record=true
2         kubectl set image deployment hello hello-container=busybox:1.31.1 --record=true
```

The history command shows all revisions you made to the deployment. The first revision is when
we initially created the resource and the second one is when we updated the image.

Let’s say we rolled out this new image version, but for some reason, we want to go back to the
previous state. Using the **rollout** command, we can also roll back to an earlier revision of the
resource.

To roll back, you can use the **rollout undo** command, like this:

```
$ kubectl rollout undo deploy hello
deployment.apps/hello rolled back
```

With the undo command, we rolled back the changes to the previous revision, which is the original
state we were in before we updated the image:

```
$ kubectl rollout history deploy hello
deployment.apps/hello
REVISION CHANGE-CAUSE
2        kubectl set image deployment hello hello-container=busybox:1.31.1 --record=true
3        kubectl apply --filename=deployment.yaml --record=true
```

Let’s remove the deployment by running:

```
$ kubectl delete deploy hello
deployment.apps "hello" deleted
```

### Deployment strategies

You might have wondered what logic or strategy Deployment controller used to bring up new Pods or terminate the old ones when we scaled the deployments up and down and updated the image names.

There are two different strategies used by deployments to replace old Pods with new ones. The
__Recreate__ strategy and the __RollingUpdate__ strategy. The latter is the default strategy.

Here’s a way to explain the differences between the two strategies. Imagine you work at a bank, and your job is to manage the tellers and ensure there’s always someone working that can handle customer requests. Since this is an imaginary bank, let’s say you have 10 desks available, and at the moment, five tellers are working.

![Figure 9. Five Bank Tellers at Work](https://i.gyazo.com/8a74ce63515cf370924701dd8116bb8b.png)

_Figure 9. Five Bank Tellers at Work_

Time for a shift change! The current tellers have to leave their desks and let the new shift come in to take over.

One way you can do that is to tell the current tellers to stop working. They will put up the "Closed" sign in their booth, pack up their stuff, get off their seats, and leave. Only once all of them have left their seats, the new tellers can come in, sit down, unpack their stuff, and start working.

![Figure 10. New Shift Waiting](https://i.gyazo.com/db2072aa74ca440e63e7e064a26bb78b.png)

_Figure 10. New Shift Waiting_

Using this strategy, there will be downtime where you won’t be able to serve any customers. As shown in the figure above, you might have one teller working, and once they pack up, it will take time for the new tellers to sit down and start their work. This is how the __Recreate__ strategy works.

The __Recreate__ strategy terminates all existing (old) Pods (shift change happens), and only when they are all terminated (they leave their booths), it starts creating the new ones (new shift comes in).

Using a different strategy, you can keep serving all of your customers while the shift is changing. Instead of waiting for all tellers to stop working first, you can utilize the empty booths and put your new shift to work right away. That means you might have more than five booths working at the same time during the shift change.

![Figure 11. Seven Tellers Working](https://i.gyazo.com/3b4f49029012f6dfadbd57b1498346a8.png)

_Figure 11. Seven Tellers Working_

As soon as you have seven tellers working, for example (five from the old shift, two from the new shift), more tellers from the old shift can start packing up, and new tellers can start replacing them. You could also say that you always want at least three tellers working during the shift change, and you can also accommodate more than five tellers working at the same time.

![Figure 12. Mix of Tellers Working](https://i.gyazo.com/ba2b7eab06c62614038bb640604a6c71.png)

_Figure 12. Mix of Tellers Working_

This is how the __RollingUpdate__ strategy works. The **maxUnavailable** and **maxSurge** settings specify the maximum number of Pods that can be unavailable and the maximum number of old and new Pods that can be running at the same time.

### **Recreate strategy**

Let’s create a deployment that uses the recreate strategy - notice the highlighted lines show where we specified the strategy.

_ch3/deployment-recreate.yaml_
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
  labels:
    app.kubernetes.io/name: hello
spec:
  replicas: 5
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app.kubernetes.io/name: hello
  template:
    metadata:
      labels:
        app.kubernetes.io/name: hello
    spec:
      containers:
        - name: hello-container
          image: busybox
          command: ["sh", "-c", "echo Hello from my container! && sleep 3600"]
```

Copy the above **YAML to deployment-recreate.yaml** file and create the deployment:

```
kubectl apply -f deployment-recreate.yaml
```

To see the recreate strategy in action, we will need a way to watch the changes that are happening
to the Pods as we update the image version, for example.

You can open a second terminal window and use the **--watch** flag when listing all Pods - the **--watch**
flag will keep the command running, and any changes to the Pods are written to the screen.

```
kubectl get pods --watch
```

From the first terminal window, let’s update the Docker image from **busybox** to **busybox:1.31.1** by
running:

```
kubectl set image deployment hello hello-container=busybox:1.31.1
```

The second terminal window’s output where we are watching the Pods should look like the one
below. Note that I have added the line breaks between the groups.

```
NAME                      READY       STATUS      RESTARTS    AGE
hello-6fcbc8bc84-jpm64    1/1         Running     0           54m
hello-6fcbc8bc84-wsw6s    1/1         Running     0           54m
hello-6fcbc8bc84-wwpk2    1/1         Running     0           54m
hello-6fcbc8bc84-z2dqv    1/1         Running     0           54m
hello-6fcbc8bc84-zpc5q    1/1         Running     0           54m

hello-6fcbc8bc84-z2dqv    1/1         Terminating 0           56m
hello-6fcbc8bc84-wwpk2    1/1         Terminating 0           56m
hello-6fcbc8bc84-wsw6s    1/1         Terminating 0           56m
hello-6fcbc8bc84-jpm64    1/1         Terminating 0           56m
hello-6fcbc8bc84-zpc5q    1/1         Terminating 0           56m
hello-6fcbc8bc84-wsw6s    0/1         Terminating 0           56m
hello-6fcbc8bc84-z2dqv    0/1         Terminating 0           56m
hello-6fcbc8bc84-zpc5q    0/1         Terminating 0           56m
hello-6fcbc8bc84-jpm64    0/1         Terminating 0           56m
hello-6fcbc8bc84-wwpk2    0/1         Terminating 0           56m
hello-6fcbc8bc84-z2dqv    0/1         Terminating 0           56m

hello-84f59c54cd-77hpt    0/1         Pending     0           0s
hello-84f59c54cd-77hpt    0/1         Pending     0           0s
hello-84f59c54cd-9st7n    0/1         Pending     0           0s
hello-84f59c54cd-lxqrn    0/1         Pending     0           0s
hello-84f59c54cd-9st7n    0/1         Pending     0           0s
hello-84f59c54cd-lxqrn    0/1         Pending     0           0s
hello-84f59c54cd-z9s5s    0/1         Pending     0           0s
hello-84f59c54cd-8f2pt    0/1         Pending     0           0s
hello-84f59c54cd-77hpt    0/1         ContainerCreating 0     0s
hello-84f59c54cd-z9s5s    0/1         Pending           0     0s
hello-84f59c54cd-8f2pt    0/1         Pending           0     0s
hello-84f59c54cd-9st7n    0/1         ContainerCreating 0     1s
hello-84f59c54cd-lxqrn    0/1         ContainerCreating 0     1s
hello-84f59c54cd-z9s5s    0/1         ContainerCreating 0     1s
hello-84f59c54cd-8f2pt    0/1         ContainerCreating 0     1s
hello-84f59c54cd-77hpt    1/1         Running           0     3s
hello-84f59c54cd-lxqrn    1/1         Running           0     4s
hello-84f59c54cd-9st7n    1/1         Running           0     5s
hello-84f59c54cd-8f2pt    1/1         Running           0     6s
hello-84f59c54cd-z9s5s    1/1         Running           0     7s
```

The first couple of lines show all five Pods running. Right at the first empty line above (I added that
for clarity), we ran the set image command. The controller terminated all Pods first. Once they were
terminated (second empty line in the output above), the controller created the new Pods.

The apparent downside of this strategy is that once controller terminates old Pods and stastarts up
the new ones, there are no running Pods to handle any traffic, which means that there will be
downtime. Make sure you delete the deployment **kubectl delete deploy hello** before continuing.
You can also press CTRL+C to stop running the **--watch** command from the second terminal window
(keep the window open as we will use it again shortly).


### **Rolling update strategy**

The second strategy called __RollingUpdate__ does the rollout in a more controlled way. There are two
settings you can tweak to control the process: **maxUnavailable** and **maxSurge**. Both of these settings
are optional and have the default values set - 25% for both settings.

The **maxUnavailable** setting specifies the maximum number of Pods that can be unavailable during
the rollout process. You can set it to an actual number or a percentage of desired Pods.

Let’s say **maxUnavailable** is set to 40%. When the update starts, the old ReplicaSet is scaled down to
60%. As soon as new Pods are started and ready, the old ReplicaSet is scaled down again and the
new ReplicaSet is scaled up. This happens in such a way that the total number of available Pods (old
and new, since we are scaling up and down) is always at least 60%.

The **maxSurge** setting specifies the maximum number of Pods that can be created over the desired
number of Pods. If we use the same percentage as before (40%), the new ReplicaSet is scaled up
right away when the rollout starts. The new ReplicaSet will be scaled up in such a way that it does
not exceed 140% of desired Pods. As old Pods get killed, the new ReplicaSet scales up again, making
sure it never goes over the 140% of desired Pods.

Let’s create the deployment again, but this time we will use the **RollingUpdate** strategy

_ch3/deployment-rolling.yaml_

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
  labels:
    app.kubernetes.io/name: hello
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 40%
      maxSurge: 40%
  selector:
    matchLabels:
      app.kubernetes.io/name: hello
  template:
    metadata:
      labels:
        app.kubernetes.io/name: hello
    spec:
      containers:
        - name: hello-container
          image: busybox
          command: ["sh", "-c", "echo Hello from my container! && sleep 3600"]
  ```

Save the contents to **deployment-rolling.yaml** and create the deployment:

```
$ kubectl apply -f deployment-rolling.yaml
deployment.apps/hello created
```

Let’s do the same we did before, run the **kubectl get po --watch** from the second terminal window
to start watching the Pods.

```
kubectl set image deployment hello hello-container=busybox:1.31.1
```

This time, you will notice that the new ReplicaSet is scaled up right away and the old ReplicaSet is
scaled down at the same time:

```
$ kubectl get po --watch
NAME                    READY         STATUS      RESTARTS      AGE
hello-6fcbc8bc84-4xnt7  1/1           Running     0             37s
hello-6fcbc8bc84-bpsxj  1/1           Running     0             37s
hello-6fcbc8bc84-dx4cg  1/1           Running     0             37s
hello-6fcbc8bc84-fx7ll  1/1           Running     0             37s
hello-6fcbc8bc84-fxsp5  1/1           Running     0             37s
hello-6fcbc8bc84-jhb29  1/1           Running     0             37s
hello-6fcbc8bc84-k8dh9  1/1           Running     0             37s
hello-6fcbc8bc84-qlt2q  1/1           Running     0             37s
hello-6fcbc8bc84-wx4v7  1/1           Running     0             37s
hello-6fcbc8bc84-xkr4x  1/1           Running     0             37s
hello-84f59c54cd-ztfg4  0/1           Pending     0             0s
hello-84f59c54cd-ztfg4  0/1           Pending     0             0s
hello-84f59c54cd-mtwcc  0/1           Pending     0             0s
hello-84f59c54cd-x7rww  0/1           Pending     0             0s
hello-6fcbc8bc84-dx4cg  1/1           Terminating 0             46s
hello-6fcbc8bc84-fx7ll  1/1           Terminating 0             46s
hello-6fcbc8bc84-bpsxj  1/1           Terminating 0             46s
hello-6fcbc8bc84-jhb29  1/1           Terminating 0             46s
...
```
Using the rolling strategy, you can keep a percentage of Pods running at all times while you’re doing
updates. That means that there will be no downtime for your users.

Make sure you delete the deployment before continuing:

```
kubectl delete deploy hello
```

## Accessing and exposing Pods with Services

Pods are supposed to be temporary or short-lived. Once they crash, they are gone, and the ReplicaSet ensures to bring up a new Pod to maintain the desired number of replicas.

Let’s say we are running a web frontend in a container within Pods. Each Pod gets a unique IP address. However, due to their temporary nature, we cannot rely on that IP address.

Let’s create a deployment that runs a web frontend:

_ch3/web-frontend.yaml_
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-frontend
  labels:
    app.kubernetes.io/name: web-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: web-frontend
  template:
    metadata:
      labels:
        app.kubernetes.io/name: web-frontend
  spec:
    containers:
      - name: web-frontend-container
        image: learncloudnative/helloworld:0.1.0
        ports:
          - containerPort: 3000
```

Comparing this deployment to the previous one, you will notice we changed the resource names
and the image we are using.

One new thing we added to the deployment is the **ports** section. Using the **containerPort** field, we
set the port number website server listens on. The **learncloudnative/helloworld:0.1.0** is a simple
Node.js Express application.
Save the above YAML in **web-frontend.yaml** and create the deployment:

```
$ kubectl apply -f web-frontend.yaml
deployment.apps/web-frontend created
```

Run **kubectl get pods** to ensure Pod is up and running and then get the logs from the container:

```
$ kubectl get po
NAME                            READY     STATUS    RESTARTS    AGE
web-frontend-68f784d855-rdt97   1/1       Running   0           65s

$ kubectl logs web-frontend-68f784d855-rdt97

> helloworld@1.0.0 start /app
> node server.js

Listening on port 3000
```

From the logs, you will notice that the container is listening on port **3000**. If we set the output flag to
gives up the wide output (**-o wide**), you’ll notice the Pods' IP address - **10.244.0.170**:

```
$ kubectl get po -o wide
NAME                          READY   STATUS    RESTARTS    AGE   IP    NODE
NOMINATED NODE READINESS GATES
web-frontend-68f784d855-rdt97 1/1     Running   0           15s    172.17.0.4
minikube <none> <none>
```

If we delete this Pod, a new one will take its' place, and it will get a brand new IP address as well:

```
$ kubectl delete po web-frontend-68f784d855-rdt97
pod "web-frontend-68f784d855-rdt97" deleted

$ kubectl get po -o wide
NAME                            READY     STATUS    RESTARTS    AGE     IP    NODE
NOMINATED NODE READINESS GATES
web-frontend-68f784d855-8c76m   1/1       Running   0           15s     172.17.0.5
minikube <none> <none>
```

Similarly, if we scale up the deployment to four Pods, we will four different IP addresses:

```
$ kubectl scale deploy web-frontend --replicas=4
deployment.apps/web-frontend scaled

$ kubectl get pods -o wide
NAME                            READY     STATUS      RESTARTS    AGE     IP       NODE
NOMINATED NODE READINESS GATES
web-frontend-68f784d855-8c76m   1/1       Running     0           5m23s   172.17.0.5
minikube <none> <none>
web-frontend-68f784d855-jrqq4   1/1       Running     0           18s     172.17.0.6
minikube <none> <none>
web-frontend-68f784d855-mftl6   1/1       Running     0           18s     172.17.0.7
minikube <none> <none>
web-frontend-68f784d855-stfqj   1/1       Running     0           18s     172.17.0.8
minikube <none> <none>
```

### **How to access the Pods without a service?**

If you try to send a request to one of those IP addresses, it’s not going to work:

```
$ curl -v 172.17.0.5:3000
*   Trying 172.17.0.5...
* TCP_NODELAY set
* Connection failed
* connect to 172.17.0.5 port 3000 failed: Network is unreachable
* Failed to connect to 172.17.0.5 port 3000: Network is unreachable
* Closing connection 0
curl: (7) Failed to connect to 172.17.0.5 port 3000: Network is unreachable
```

The Pods are running within the cluster, and that IP address is only accessible from within the
cluster.

![Figure 13. Can’t Access Pods from Outside](https://i.gyazo.com/b622b360a7aab8e97aa42c5ecb308c71.png)

_Figure 13. Can’t Access Pods from Outside_

For the testing purposes, you can run a pod inside the cluster and then get shell access to that Pod.
Yes, that is possible!

![Figure 14. Accessing Pods from a Pod](https://i.gyazo.com/eed34e3ad687049d59cf6606bb8d345d.png)

_Figure 14. Accessing Pods from a Pod_

The **radialbusyboxplus:curl** is the image I frequently run inside the cluster if I need to check
something or debug things. Using the **-i** and **--tty** flags, we are want to allocate a terminal (**tty**),
and that we want an interactive session so that we can run commands directly inside the container.

I usually name this Pod **curl**, but you can name it whatever you like:

```
$ kubectl run curl --image=radial/busyboxplus:curl -i --tty
If you dont see a command prompt, try pressing enter.
[ root@curl:/ ]$
```

Now that we have access to the the terminal running inside the container that' inside the cluster,
we can run the same cURL command as before:

```
[ root@curl:/ ]$ curl -v 172.17.0.5:3000
> GET / HTTP/1.1
> User-Agent: curl/7.35.0
> Host: 172.17.0.5:3000
> Accept: */*
>
< HTTP/1.1 200 OK
< X-Powered-By: Express
< Content-Type: text/html; charset=utf-8
< Content-Length: 111
< ETag: W/"6f-U4ut6Q03D1uC/sbzBDyZfMqFSh0"
< Date: Wed, 20 May 2020 22:10:49 GMT
< Connection: keep-alive
<
<link rel="stylesheet" type="text/css" href="css/style.css" />

<div class="container">
  Hello World!
</div>[ root@curl:/ ]$
```

This time, we get a response from the Pod! Make sure you run **exit** to return to your terminal. The
**curl** Pod will continue to run and to reaccess it, you can use the **attach** command:

```
kubectl attach curl -c curl -i -t
```

__TIP__:

You can get a terminal session to any container running inside the cluster using the
**attach** command.

### **Using a Kubernetes Service**

The Kubernetes Service is an abstraction that gives us a way to reach the Pod IP’s reliably.

The service controller (similar to the ReplicaSet controller) maintains a list of endpoints or the Pod
IP addresses. The controller uses a selector and labels to watch the Pods. Whenever the controller
creates or deletes a Pod that matches the selector, the controller adds or removes the Pods' IP
address from the endpoints list.

Let’s look at how would the Service look like for our website:

_ch3/web-frontend-service.yaml_
```
kind: Service
apiVersion: v1
metadata:
  name: web-frontend
  labels:
    app.kubernetes.io/name: web-frontend
spec:
  selector:
    app.kubernetes.io/name: web-frontend
  ports:
    - port: 80
      name: http
      targetPort: 3000

```

The top portion of the YAML should already look familiar - except for the **kind** field, it’s the same as
we saw with the Pods, ReplicaSets, Deployments.

The highlighted **selector** section is where we define the labels that Service uses to query the Pods. If
you go back to the Deployment YAML, you will notice that the Pods have this exact label set as well.

![Figure 15. Kubernetes Services and Pods](https://i.gyazo.com/7f3a938a64589154740027953dca66e0.jpg)

_Figure 15. Kubernetes Services and Pods_

Lastly, under the **ports** field, we are defining the port where the Service will be accessible on (**80**),
and with the **targetPort**, we are telling the Service on which port it can access the Pods. The
**targetPort** value matches the **containerPort** value in the Deployment YAML.

Save the YAML from above to **web-frontend-service.yaml** file and deploy it:

```
$ kubectl apply -f web-frontend-service.yaml
service/web-frontend created
```

To see the created service you can run the **get service** command:

```
$ kubectl get service
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP       PORT(S)       AGE
kubernetes    ClusterIP   10.96.0.1       <none>            443/TCP       7d
web-frontend  ClusterIP   10.100.221.29   <none>            80/TCP        24s
```

The **web-frontend** Service has an IP address that will not change (assuming you don’t delete the
Service), and you use it to access the underlying Pods reliably.

Let’s attach to the **curl** container we started before and try to access the Pods through the Service:

```
$ kubectl attach curl -c curl -i -t
If you dont see a command prompt, try pressing enter.
[ root@curl:/ ]$
```

Since we set the service port to **80**, we can curl directly to the service IP and we will get back the
same response as previously:

```
[ root@curl:/ ]$ curl 10.100.221.29
<link rel="stylesheet" type="text/css" href="css/style.css" />

<div class="container">
  Hello World!
</div>
```

Even though the Service IP address is stable and won’t change, it would be much better to use a
friendlier name to access the Service. Every Service you create in Kubernetes gets a DNS name
assigned following this format **service-name.namespace.svc.cluster-domain.sample**.

So far, we deployed everything to the **default** namespace and the cluster domain is **cluster.local**
we can access the service using **web-frontend.default.svc.cluster.local**:

```
[ root@curl:/ ]$ curl web-frontend.default.svc.cluster.local
<link rel="stylesheet" type="text/css" href="css/style.css" />

<div class="container">
  Hello World!
</div>
```

In addition to using the full name, you can also use just the service name, or service name and the
namespace name:

```
</div>[ root@curl:/ ]$ curl web-frontend
<link rel="stylesheet" type="text/css" href="css/style.css" />

<div class="container">
  Hello World!
[ root@curl:/ ]$ curl web-frontend.default
<link rel="stylesheet" type="text/css" href="css/style.css" />

<div class="container">
  Hello World!
</div>
```

I would suggest you always use the Service name and the namespace name when making requests.

### **Using the Kubernetes proxy**

Another way for accessing services that are only available inside of the cluster is through the proxy.
The **kubectl proxy** command creates a gateway between your local computer (localhost) and the
Kubernetes API server.

The proxy allows you to access the Kubernetes API as well as access the Kubernetes services. You
should __NEVER__ use this proxy to expose your service to the public. You should only use the proxy
for debugging or troubleshooting.

Let’s open a separate terminal window and run the following command to start the proxy server
that will proxy requests from **localhost:8080** to the Kubernetes API inside the cluster:

```
kubectl proxy --port=8080
```

If you open http://localhost:8080/ in your browser, you will see the list of all APIs from the
Kubernetes API proxy:

```
{
  "paths": [
  "/api",
  "/api/v1",
  "/apis",
  "/apis/",
  "/apis/admissionregistration.k8s.io",
  "/apis/admissionregistration.k8s.io/v1",
  "/apis/admissionregistration.k8s.io/v1beta1",
  ]
  ...
}
```

For example, if you want to see all Pods running in the cluster, you could navigate to http://localhost:8080/api/v1/pods or navigate to http://localhost:8080/api/v1/namespaces to see all namespaces.

Using this proxy, we can also access the **web-frontend** service we deployed. So, instead of running a
Pod inside the cluster and make cURL requests to the service or Pods, you can create the proxy
server and use the following URL to access the service:

```
http://localhost:8080/api/v1/namespaces/default/services/web-frontend:80/proxy/
```

__NOTE__:

The URL format to access a service is
[PORT](http://localhost/)**/api/v1/namespaces/[NAMESPACE]/services/[SERVICE_NAME]:[SERVICE_PORT]/proxy**.
In addition to using the service port (e.g. **80**) you can also name your ports an use
the port name instead (e.g. **http**).

Browsing to the URL above will render the simple HTML site with the **Hello World** message.

![Figure 16. Accessing the Service through Kubernetes Proxy](https://i.gyazo.com/663dece0a3e15f837f08b8e9cf18d158.jpg)

_Figure 16. Accessing the Service through Kubernetes Proxy_

To stop the proxy, you can press **Ctrl** + **C** in the terminal window.

### **Viewing Service details**

Using the **describe** command, you can describe an object in Kubernetes and look at its properties.
For example, let’s take a look at the details of the **web-frontend** Service we deployed:

```
$ kubectl describe svc web-frontend
Name:             web-frontend
Namespace:        default
Labels:           app.kubernetes.io/name=web-frontend
Annotations:      Selector: app.kubernetes.io/name=web-frontend
Type:             ClusterIP
IP:               10.100.221.29
Port:             http 80/TCP
TargetPort:       3000/TCP
Endpoints:        172.17.0.4:3000,172.17.0.5:3000,172.17.0.6:3000 + 1 more...
Session Affinity: None
Events:           <none>
```

This view gives us more information than the **get** command does - it shows the labels, selector, the
service type, the IP, and the ports. Additionally, you will also notice the **Endpoints**. These IP
addresses correspond to the IP addresses of the Pods.

You can also view endpoints using the **get endpoints** command:

```
$ kubectl get endpoints
NAME             ENDPOINTS                                                    AGE
kubernetes       192.168.64.5:8443                                            13m
web-frontend     172.17.0.4:3000,172.17.0.5:3000,172.17.0.6:3000 + 1 more...  66s
```

To see the controller that manages these endpoints in action, you can use the **--watch** flag to watch
the endpoints like this:

```
$ kubectl get endpoints --watch
```

Then, in a separate terminal window, let’s scale the deployment to one Pod:

```
$ kubectl scale deploy web-frontend --replicas=1
deployment.apps/web-frontend scaled
```

As soon as the deployment is scaled, you will notice how the endpoints get automatically updated.
For example, this is how the output looks like when the deployment is scaled:

```
$ kubectl get endpoints -w
NAME          ENDPOINTS                                                     AGE
kubernetes    192.168.64.5:8443                                             14m
web-frontend  172.17.0.4:3000,172.17.0.5:3000,172.17.0.6:3000 + 1 more...   93s
web-frontend  172.17.0.4:3000,172.17.0.5:3000,172.17.0.7:3000               95s
web-frontend  172.17.0.5:3000                                               95s
```

Notice how it went from four Pods, then down to two and finally to one. If you scale the deployment back to four Pods, you will see the endpoints list populated with new IPs.


### **Kubernetes service types**

From the service description output we saw earlier, you might have noticed this line:

```
Type: ClusterIP
```

Every Kubernetes Service has a type. If you don’t provide a service type, the __ClusterIP__ gets assigned
by default. In addition to the ClusterIP, there are three other service types in Kubernetes. These are
__NodePort__, __LoadBalancer__, and __ExternalName__.

Let’s explain the differences between these service types.

__ClusterIP__

You would use the __ClusterIP__ service type to access Pods from within the cluster through a cluster-internal IP address. In most cases, you will use this type of service for your applications running
inside the cluster. Using the ClusterIP type, you can define the port you want your service to be
listening on through the **ports** section in the YAML file.

Kubernetes assigns a cluster IP to the service. You can then access the service using the cluster IP
address and the port you specified in the YAML.

![Figure 17. Cluster IP Service](https://i.gyazo.com/4341c37b5db052084b80fb392eafff4e.png)

_Figure 17. Cluster IP Service_

__NodePort__

At some point, you will want to expose your services to the public and allow external traffic to
enter your cluster. The __NodePort__ service type opens a specific port on every worker node in your cluster. Any traffic sent to the node IP and the port number reaches the Service and your Pods.

![Figure 18. NodePort Service](https://i.gyazo.com/2e4bd618f73ddf5b04a852af9f156d9e.png)

_Figure 18. NodePort Service_

For example, if you use the node IP and the port **30000** as shown in the figure above, you will access
the Service and the Pods.

To create a NodePort Service, you need to specify the service type as shown in the listing below.

_ch3/nodeport-service.yaml_

```
kind: Service
apiVersion: v1
metadata:
  name: web-frontend
  labels:
    app.kubernetes.io/name: web-frontend
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: web-frontend
  ports:
    - port: 80
      name: http
      targetPort: 3000
```

You can set additional field called **nodePort** under the **ports** section. However, it is a best practice to
leave it out and let Kubernetes pick a port available on all nodes in the cluster. By default,
Kubernetes allocates the node port between 30000 and 32767 (this is configurable in the API
server).

You would use the NodePort type when you want to control the load balancing. You can expose
your services via NodePort and then configure the load balancer to use the node IPs and node
ports. Another scenario where you could use this is if you are migrating an existing solution to
Kubernetes, for example. In that case, you’d probably already have an existing load balancer, and
you could add the node IPs and node ports to the load balancing pool.

Let’s delete the previous **web-frontend** service and create one that uses **NodePort**. To delete the
previous service, run **kubectl delete svc web-frontend**. Then, copy the YAML contents above to the
**nodeport-service.yaml** file and run **kubectl apply -f web-frontend-nodeport.yaml**:

_ch3/nodeport-service.yaml_

```
kind: Service
apiVersion: v1
metadata:
  name: web-frontend
  labels:
    app.kubernetes.io/name: web-frontend
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: web-frontend
  ports:
    - port: 80
      name: http
      targetPort: 3000
```

Once the service is created, run **kubectl get svc** - you will notice the service type has changed and
the port number is random as well:

```
$ kubectl get svc
NAME          TYPE          CLUSTER-IP      EXTERNAL-IP     PORT(S)       AGE
kubernetes    ClusterIP     10.96.0.1       <none>          443/TCP       16m
web-frontend  NodePort      10.107.154.215  <none>          80:30417/TCP  4s
```

Now since we are using a local Kubernetes cluster (kind, Minikube, or Docker for Mac/Windows),
demonstrating the NodePorts is a bit awkward. We have a single node, and the node IPs are private
as well. When using a cloud-managed cluster you can set up the load balancer to access the same
virtual network where your nodes are. Then you can configure it to access the node IPs through the
node ports.

Let’s get the internal node IP:

```
$ kubectl describe node | grep InternalIP
  InternalIP: 192.168.64.5
```

__TIP__:

If using Minikube, you can also run **minikube ip** to get the cluster’s IP address.

Armed with this IP we can cURL to the clusters IP address and the node port (**30417**):

```
$ curl 192.168.64.5:30417
<link rel="stylesheet" type="text/css" href="css/style.css" />

<div class="container">
  Hello World!
</div>
```

If you are using Docker for Desktop, you can use **localhost** as the node address and the node port to
access the service. If you open http://localhost:30417 in your browser, you will be able to access
the **web-frontend** service.

**LoadBalancer**

The LoadBalancer service type is the way to expose Kubernetes services to external traffic. If you
are running a cloud-managed cluster and create the Service of the LoadBalancer type, the
Kubernetes cloud controller creates an actual Load Balancer in your cloud account.

![Figure 19. LoadBalancer Service](https://i.gyazo.com/504688e0af954b6cae7a1e41209717b4.png)

_Figure 19. LoadBalancer Service_

Let’s delete the previous NodePort service with **kubectl delete svc web-frontend**.

_ch3/lb-service.yaml_

```
kind: Service
apiVersion: v1
metadata:
  name: web-frontend
  labels:
    app.kubernetes.io/name: web-frontend
spec:
  type: LoadBalancer
  selector:
    app.kubernetes.io/name: web-frontend
  ports:
    - port: 80
      name: http
      targetPort: 3000
```

We can create a service that uses the LoadBalancer type using the above YAML. Run **kubectl apply -f lb-service.yaml** to create the service.

If we look at the service now, you’ll notice that the type has changed to LoadBalancer and the
external IP address is pending:

```
$ kubectl get svc
NAME          TYPE            CLUSTER-IP        EXTERNAL-IP       PORT(S)     AGE
kubernetes    ClusterIP       10.96.0.1         <none>            443/TCP     19m
web-frontend  LoadBalancer    10.106.30.168     <pending>         80:30962/TCP 4s
```

__NOTE__:

When using a cloud-managed Kubernetes cluster, the external IP would be a public,
the external IP address you could use to access the service.

If you are using Docker Desktop, you can open http://localhost or http://127.0.0.1 in your
browser to see the website running inside the cluster.

If you are using Minikube, you can run the **minikube tunnel** command from a separate terminal
window. Once the tunnel command is running, run **kubectl get svc** again to get an external IP
address of the service:

```
$ kubectl get svc
NAME          TYPE          CLUSTER-IP        EXTERNAL-IP       PORT(S)         AGE
kubernetes    ClusterIP     10.96.0.1         <none>            443/TCP         21m
web-frontend  LoadBalancer  10.106.30.168     10.106.30.168     80:30962/TCP    104s
```

You can now open http://10.106.30.168 in your browser to access the service running inside the
cluster. We will discuss how the Minikube tunnel command works in the [Exposing multiple
applications with Ingress]() section.

__ExternalName__

The __ExternalName__ service type is a particular type of a service that does not use selectors. Instead,
it uses DNS names.

Using the ExternalName, you can map a Kubernetes service to a DNS name. When you send a
request to the Service, it returns the CNAME record with the value in **externalName** field instead of
the service’s cluster IP.

Here’s an example of how ExternalName service would look like:

_ch3/external-name.yaml_
```
kind: Service
apiVersion: v1
metadata:
  name: my-database
spec:
  type: ExternalName
  externalName: mydatabase.example.com
```

You could use the ExternalName service type when migrating workloads to Kubernetes, for
example. You could keep your database running outside of the cluster and then use the **my-database**
service to access it from the workloads running inside your cluster.

## **Exposing multiple applications with Ingress**

You can use the Ingress resource to manage external access to the Services running inside your
cluster. With the Ingress resource, you can define the rules on how the services inside the cluster
can be accessed.

The Ingress resource on its own is useless. It’s a collection of rules and paths, but it needs
something to apply these rules to. That "something" is an __ingress controller__. The ingress controller
acts as a gateway and routes the traffic based on the Ingress resource rules defined.

![Figure 20. Kubernetes Ingress](https://i.gyazo.com/033e8531e42730e5b93835d49322a228.png)

_Figure 20. Kubernetes Ingress_

An ingress controller is a collection of the following items:

- Kubernetes deployment running one or more Pods with containers running a gateway/proxy
server such as NGINX, Ambassador, etc.
- Kubernetes service that exposes the ingress controller Pods
- Other supporting resources for the ingress controller (configuration maps, secrets, etc.)

__NOTE__:

How about the load balancer? The [load balancer]() is not necessarily part of the
Ingress controller. The Kubernetes service used for the ingress controller can be of
the LoadBalancer type, which triggers a load balancer’s creation if using a cloud-managed Kubernetes cluster. It is a way for the traffic to enter your cluster and,
subsequently the ingress controller that routes the traffic according to the rules.

The idea is that you can deploy the ingress controller, expose it on a public IP address, then use the
Ingress resource to create the traffic rules. Here’s how an Ingress resource would look like:

_ch3/ingress-example.yaml_

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-example
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /blog
            backend:
              serviceName: my-blog-service
              servicePort: 5000
          - path: /music
            backend:
              serviceName: my-music-service
              servicePort: 8080
```

With these rules, we are routing traffic that comes into **example.com/blog** to a Kubernetes service **my-blog-service:5000**. Similarly, any traffic coming to **example.com/music** goes to a Kubernetes service
**my-music-service:8080**.

__NOTE__

The ingress resource will also contain one or more annotations to configure the
Ingress controller. The annotations and options you can configure will depend on
the ingress controller you’re using.

Let’s say we want to run two websites in our cluster - the first one will be a simple Hello World
website, and the second one will be a Daily Dog Picture website that shows a random dog picture.

Assuming you have your cluster up and running, let’s create the deployments for these two
websites.

_ch3/helloworld-deployment.yaml_

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
  labels:
    app.kubernetes.io/name: hello-world
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: hello-world
  template:
    metadata:
      labels:
        app.kubernetes.io/name: hello-world
    spec:
      containers:
        - name: hello-world-container
          image: learncloudnative/helloworld:0.1.0
          ports:
            - containerPort: 3000
```

Run **kubectl apply -f helloworld-deployment.yaml** to create the **hello-world** deployment. Next, we
will deploy the Daily Dog Picture website.

_ch3/dogpic-deployment.yaml_

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dogpic-web
  labels:
    app.kubernetes.io/name: dogpic-web
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: dogpic-web
  template:
    metadata:
      labels:
        app.kubernetes.io/name: dogpic-web
    spec:
      containers:
        - name: dogpic-container
          image: learncloudnative/dogpic-service:0.1.0
          ports:
            - containerPort: 3000
  ```

Run **kubectl apply -f dogpic-deployment.yaml** to deploy the Daily Dog Picture website. Make sure
both pods from both deployments are up and running:

```
$ kubectl get pods
NAME                          READY         STATUS        RESTARTS          AGE
dogpic-web-559f4bb5db-dlrks   1/1           Running       0                 24m
hello-world-5fd44c56d7-d8g4j  1/1           Running       0                 29m
```

We still need to deploy the Kubernetes Services for both of these deployments. The services will be
of default, **ClusterIP** type, so there’s no need to set **type** field explicitly. You can refer to [Kubernetes
service types]() for the explanation of the **ClusterIP** service.

_ch3/services.yaml_

```
kind: Service
apiVersion: v1
metadata:
  name: dogpic-service
  labels:
  app.kubernetes.io/name: dogpic-web
spec:
  selector:
  app.kubernetes.io/name: dogpic-web
  ports:
  - port: 3000
  name: http
---
kind: Service
apiVersion: v1
metadata:
  name: hello-world
  labels:
  app.kubernetes.io/name: hello-world
spec:
  selector:
  app.kubernetes.io/name: hello-world
  ports:
  - port: 3000
  name: http
```

Save the YAML above to **service.yaml** and then create the services by running **kubectl apply -f
services.yaml**.

__TIP__

You can use **---** as a separator in YAML files to deploy multiple resources from a single
file.

### Installing the Ambassador API gateway

Before we create the Ingress resource, we need to deploy an Ingress controller. The job of an
ingress controller is to receive the incoming traffic and route it based on the rules defined in the
Ingress resource.

You have multiple options you can go with for the Ingress controller. Some of the gateways and
proxies you could use are:
- [Ambassador](https://www.getambassador.io/docs/latest/topics/running/ingress-controller/)
- [NGINX](https://www.nginx.com/products/nginx/kubernetes-ingress-controller)
- [HAProxy](https://www.haproxy.com/documentation/hapee/1-9r1/installation/kubernetes-ingress-controller/)
- [Traefik](https://docs.traefik.io/providers/kubernetes-ingress/)

You can find the list of other controllers in the [Kubernetes ingress controller documentation](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/).

In this example, I’ll be using the open-source version of the [Ambassador API gateway](https://www.getambassador.io/).

__NOTE__:

"I heard ABC/XZY/DEF is much better than GHI and JKL". Yep, that very well might
be right. My purpose is to explain what an Ingress resource is and how it works.
Some of the ingress controllers use their custom resources, instead of the default
Kubernetes Ingress resource. That way, they can support more features than the
default Ingress resource. I would encourage you to explore the available options
and pick the one that works best for you.

To deploy the Ambassador API gateway, we will start by deploying the __custom resource
definitions (CRDs)__ the gateway uses:

```
$ kubectl apply -f https://www.getambassador.io/yaml/ambassador/ambassador-crds.yaml
customresourcedefinition.apiextensions.k8s.io/authservices.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/consulresolvers.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/hosts.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/kubernetesendpointresolvers.getambassado
r.io created
customresourcedefinition.apiextensions.k8s.io/kubernetesserviceresolvers.getambassador
.io created
customresourcedefinition.apiextensions.k8s.io/logservices.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/mappings.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/modules.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/ratelimitservices.getambassador.io
created
customresourcedefinition.apiextensions.k8s.io/tcpmappings.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/tlscontexts.getambassador.io created
customresourcedefinition.apiextensions.k8s.io/tracingservices.getambassador.io created
```

**What is the difference between __create__ and __apply__?**

It’s a difference between imperative management (**create**) and declarative management (**apply**).

Using the **create** command, you are telling Kubernetes which resources to create or delete. With
**apply**, you are telling Kubernetes how you want your resources to look. You don’t define operations
to be taken as you would with **create** or **delete**. You are letting Kubernetes detect the operations for
each object. Let’s say you used the **create** command and create a deployment with image **image:123**.
If you want to change the image in the deployment to **image:999** you won’t be able to use the create
command as the deployment already exists. You’d have to delete the deployment first, then create it
again. Using the **apply** command, you don’t need to delete the deployment. The apply command will
'apply' the desired changes to an existing resource (i.e., update the image name in our case). You
can use both approaches in production. Using the declarative approach, Kubernetes determines the
changes needed for each object. The object retains any configuration changes made with the
declarative approach. If you’re using the imperative approach, the changes made previously will be
gone as you will replace it. On the other hand, the declarative approach can be harder to debug
because the resulting object is not necessarily the same as in the file you applied.

The next step is to create the Ambassador deployment (**ambassador**) and other resources needed to
run the API gateway:

```
$ kubectl apply -f https://www.getambassador.io/yaml/ambassador/ambassador-rbac.yaml
service/ambassador-admin created
clusterrole.rbac.authorization.k8s.io/ambassador created
serviceaccount/ambassador created
clusterrolebinding.rbac.authorization.k8s.io/ambassador created
deployment.apps/ambassador created
```

__NOTE__:

RBAC stands for Role-Based Access Control, and it is a way of controlling access to
resources based on the roles. For example, using RBAC, you can create roles called
**admin** and **normaluser**, and then allow admin role access to everything and
**normaluser** only access to certain namespaces or control if they can create or only
view resources. You can read more about the RBAC in [Using Role-Based Access
Control (RBAC)]()

Let’s see the resources we created when we deployed the Ambassador API gateway:

```
$ kubectl get deploy
NAME          READY       UP-TO-DATE      AVAILABLE       AGE
ambassador    3/3         3               3               30m
dogpic-web    1/1         1               1               2d
hello-world   1/1         1               1               2d

$ kubectl get svc
NAME              TYPE      CLUSTER-IP      EXTERNAL-IP         PORT(S)         AGE
ambassador-admin  NodePort  10.107.45.225   <none>              8877:31524/TCP  30m
dogpic-service    ClusterIP 10.110.213.161  <none>              3000/TCP        48m
hello-world       ClusterIP 10.109.157.27   <none>              3000/TCP        48m
kubernetes        ClusterIP 10.96.0.1       <none>              443/TCP         66d
```

The default installation creates three Ambassador Pods and the **ambassador-admin** service.

We need to separately create a LoadBalancer service that will route traffic to the **ambassador** Pods.

_ch3/ambassador-service.yaml_

```
apiVersion: v1
kind: Service
metadata:
  name: ambassador
spec:
  type: LoadBalancer
  externalTrafficPolicy: Local
  ports:
    - port: 80
      targetPort: 8080
  selector:
    service: ambassador
```

Create the load balancer service for Ambassador, by running **kubectl apply -f ambassador-service.yaml**.

If you list the services again, you will notice the **ambassador** service doesn’t have an IP address in the
EXTERNAL-IP column. This is because we are running a cluster locally. If we used a cloud-managed
cluster, this would create an actual load balancer instance in our cloud account, and we would get a
public/private IP address we could use to access the services.

```
$ kubectl get svc
NAME              TYPE          CLUSTER-IP      EXTERNAL-IP         PORT(S)         AGE
ambassador        LoadBalancer  10.109.103.63   <pending>           80:32004/TCP    10d
ambassador-admin  NodePort      10.107.45.225   <none>              8877:31524/TCP  10d
dogpic-service    ClusterIP     10.110.213.161  <none>              3000/TCP        10d
hello-world       ClusterIP     10.109.157.27   <none>              3000/TCP        10d
kubernetes        ClusterIP     10.96.0.1       <none>              443/TCP         77d
```

With Minikube, you can access the **NodePort** services using the combination of the cluster IP and the
port number (e.g., 32004 or 31524). The command **minikube ip** gives you the clusters' IP address
(**192.168.64.3** in this case). You could use that IP and the NodePort, for example, **32004** for the
**ambassador** service, and access the service.

An even better approach is to use the **minikube service** command and have Minikube open the
correct IP and port number. Try and run the following command:

```
$ minikube service ambassador
|-----------|------------|-------------|---------------------------|
| NAMESPACE |    NAME    | TARGET PORT |            URL            |
|-----------|------------|-------------|---------------------------|
| default   | ambassador |             | http://192.168.64.3:32004 |
|-----------|------------|-------------|---------------------------|
Ἰ Opening service default/ambassador in default browser...
```

The page won’t render because we haven’t created any Ingress rules yet. However, you can try and
navigate to http://192.168.64.3:32004/ambassador/v0/diag to open the Ambassador diagnostics
page.

You could open any other service that’s of type **NodePort** using the same command.

However, we want to use the LoadBalancer service type and a completely different IP address, so
we don’t have to deal with the cluster IP or the node ports. You can use the **tunnel** command to
create a route to all services deployed with the LoadBalancer type.

Since this command has to be running, open a separate terminal window and run **minikube tunnel**:

```
$ minikube tunnel
Status:
        machine: minikube
        pid: 50383
        route: 10.96.0.0/12 -> 192.168.64.3
        minikube: Running
        services: [ambassador]
    errors:
            minikube: no errors
            router: no errors
            loadbalancer emulator: no errors
```

__WARNING__:

Minikube **tunnel** command needs admin privileges and you might get
prompted for a password.

The tunnel command creates a network route on your computer to the cluster’s service CIDR
(Classless Inter-Domain Routing). The **10.96.0.0/12** CIDR includes IPs starting from **10.96.0.0** to
**10.111.255.255**. This network route uses the cluster’s IP address (**192.168.64.3**) as a gateway. You
can also get the Minikube clusters' IP address by running minikube ip command.

Let’s list the services again, and this time the **ambassador** service will get an actual IP address that
falls in the CIDR from the tunnel command:

```
$ kubectl get svc
NAME          TYPE           CLUSTER-IP        EXTERNAL-IP       PORT(S)       AGE
ambassador    LoadBalancer   10.102.244.196    10.102.244.196    80:30395/TCP  1h
ambassador-admin NodePort    10.106.191.105    <none>            8877:32561/TCP 21h
dogpic-service    ClusterIP  10.104.72.244     <none>            3000/TCP      21h
hello-world       ClusterIP  10.108.178.113    <none>            3000/TCP      21h
kubernetes        ClusterIP  10.96.0.1         <none>            443/TCP       67d
```

Since we will be using the external IP address, let’s store it in an environment variable, so we don’t
have to type it out each time:

```
$ export AMBASSADOR_LB=10.102.244.196
```
Now we can open the build-in Ambassador diagnostic web site by navigating to:
[http://AMBASSADOR_LB/ambassador/v0/diag]() (replace the AMBASSADOR_LB with the actual IP address).

![Figure 21. Ambassador API Gateway Diagnostics](https://i.gyazo.com/92c804ba913ed6e53bc68c9cdbbc20d9.png)

_Figure 21. Ambassador API Gateway Diagnostics_


The Ambassador API gateway diagnostics page gives you an overview of the gateway. You could use this if you are running into any issues or if you need to debug something. Of course, you can also turn this diagnostics page off for any production scenarios.

### **Single service Ingress**

Now that we have the ingress controller up and running, we can create an Ingress resource.

The simplest version of an Ingress resource is one without any rules. Ingress directs all traffic to the same backend service, regardless of the traffic origin.

![Figure 22. Single Service Ingress](https://i.gyazo.com/3d59d813602204b65a528018f87c69ff.png)

_Figure 22. Single Service Ingress_

Let’s create an Ingress that only defines a backend service and doesn’t have any rules.

_ch3/single-service-ing.yaml_

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: ambassador
  name: my-ingress
spec:
  defaultBackend:
    service:
      name: hello-world
      port:
        number: 3000
```

Save the YAML to **single-service-ing.yaml** and deploy the Ingress using **kubectl apply -f single-service-ing.yaml** command.

We used the following annotation **kubernetes.io/ingress/class: ambassador** in the above YAML. The
Ambassador controller uses this annotation to claim the Ingress resource, and any traffic sent to
the controller will be using the rules defined in the Ingress resource.

If you list the Ingress resources, you will see the created resource:

```
$ kubectl get ing
NAME         CLASS     HOSTS       ADDRESS     PORTS     AGE
my-ingress   <none>    *                       80        1h
```

The __*__ in the HOSTS column means that there are no hosts defined. Later, when we define per-host
rules, you will see those rules show up under the HOSTS column.

If you open the browser and navigate to the same IP as before ([http://AMBASSADOR_LB]()), the Hello
World website will show up.

### **Path based routing with Ingress**
Since we want to expose two services through the Ingress, we need to write some rules. Using a
path configuration, you can route traffic from one hostname to multiple services based on the URI
path.

In this example, we want to route traffic from [http://AMBASSADOR_LB/hello]() to the Hello World
service and traffic from [http://AMBASSADOR_LB/dog]() to Dog Pic Service.

![Figure 23. Path-based Routing with Ingress](https://i.gyazo.com/48ad35b0387f61dcd0826c35f39e4a7c.png)

_Figure 23. Path-based Routing with Ingress_


To do that, we will define two rules in the Ingress resource:

_ch3/path-ing.yaml_

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: ambassador
  name: my-ingress
spec:
    rules:
      - http:
          paths:
            - path: /hello
              pathType: Prefix
              backend:
                service:
                  name: hello-world
                  port:
                    number: 3000
            - path: /dog
              pathType: Prefix
              backend:
                service:
                  name: dogpic-service
                  port:
                    number: 3000
```

Save the YAML to **path-ing.yaml** and create the ingress by running **kubectl apply -f path-ing.yaml**.

Let’s look at the details of the created Ingress resource using the **describe** command:

```
$ kubect describe ing my-ingress
Name: my-ingress
Namespace: default
Address:
Default backend: default-http-backend:80 (<error: endpoints "default-http-backend"
not found>)
Rules:
  Host Path Backends
  ---- ---- --------
  *
  /hello hello-world:3000 (172.17.0.4:3000)
  /dog dogpic-service:3000 (172.17.0.5:3000)
Annotations: kubernetes.io/ingress.class: ambassador
Events: <none>
```

Under the rules section, you will see the two paths we defined and the backends (service names).

If you navigate to [http://AMBASSADOR_LB/hello](http://ambassador_lb/hello) the Hello World website will render, and if you navigate to `__http://AMBASSADOR_LB/dog__ you will get the Dog Pic website.

Let’s take this a step further. Wouldn’t it be nice if we could type in http://example.com/dog instead
of the IP address?

__Using a hostname instead of an IP address__

If we want to use a hostname, we will have to specify it in the Ingress resource, so the controller
knows which hosts and where to direct the traffic.

_ch3/hostname-ing.yaml_

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: ambassador
  name: my-ingress
spec:
    rules:
      - host: example.com
        http:
          paths:
            - path: /hello
              pathType: Prefix
              backend:
                service:
                  name: hello-world
                  port:
                    number: 3000
            - path: /dog
              pathType: Prefix
              backend:
                service:
                  name: dogpic-service
                  port:
                    number: 3000
```

Save the above YAML to **hostname-ing.yaml** file and run **kubectl apply -f hostname-ing.yaml** to
create the Ingress.

This time we defined a host name (**example.com**) and that will show up when you get the Ingress
details:
```
$ kubectl describe ing my-ingress
Name: my-ingress
Namespace: default
Address:
Default backend: default-http-backend:80 (<error: endpoints "default-http-backend"
not found>)
Rules:
  Host Path Backends
  ---- ---- --------
  example.com
      /hello hello-world:3000 (172.17.0.4:3000)
      /dog dogpic-service:3000 (172.17.0.5:3000)
Annotations: kubernetes.io/ingress.class: ambassador
Events: <none>
```

Notice how the Host column contains the actual host we defined.

If you try to navigate to the same Ambassador load balancer address as before
([http://AMBASSADOR_LB]()), you will get an HTTP 404 error. This error is expected because we explicitly
defined the host (**example.com**), but we haven’t defined a default backend service - this is the service
traffic gets routed to if none of the rules evaluate to true. We will see how to do that later on.

There are multiple ways you can access the IP address using a hostname.

The simplest way is to set a Host header when making a request from the terminal. For example:

```
$ curl -H "Host: example.com" http://$AMBASSADOR_LB/hello
<link rel="stylesheet" type="text/css" href="css/style.css" />

<div class="container">
  Hello World!
</div>
```

Setting the Host header works, but it would be much better if we could do the same through a
browser.

I am using a browser extension called ModHeader[https://bewisse.com/modheader]. This extension
allows you to set the same Host header in your browser.

![Figure 24. ModHeader Extension](https://i.gyazo.com/851bb2946d780f9c810a7b28c0af2135.png)

_Figure 24. ModHeader Extension_

If you navigate to [http://$AMBASSADOR_LB/hello](http://%24ambassador_lb/hello) or [http://$AMBASSADOR_LB/dog](http://%24ambassador_lb/dog) you will notice both
web pages will load. This option works well, as you can load the page in the browser. However, it
would be helpful to use the hostname e.g. **example.com/dog**, for example.

You can modify the **hosts** file on your computer that allows you to map hostnames to IP addresses.
You can map the IP address ($AMBASSADOR_LB) to **example.com**.

Open the **/etc/hosts** file (or **%SystemRoot%\System32\drivers\etc\hosts** on Windows) and add the line
mapping the hostname to an IP address. Make sure you use sudo or open the file as administrator
on Windows.

```
$ sudo vim /etc/hosts
...
10.102.244.196 example.com
...
```

Save the file and if you navigate to **example.com/hello** or **example.com/dog** you will see both pages
open. Make sure to uncheck/delete the header you have set with ModHeader.

Next, let’s see how we can set a default backend that receives the traffic if Ingress controller can’t
match any of the rules.

__Setting a default backend__

In most cases, the default backend will be set by the Ingress controller. Some Ingress controllers
automatically install a default backend service as well (NGINX, for example). Then, to configure the
default backend, you can use either annotation or one of the custom resource definitions installed
Ingress controller supports.

Since we don’t want to dig into Ingress controllers' specifics, we will set the default backend directly
in the Ingress resource. Ideally, you would be using your Ingress controller configuration and set
the default backend there. To be completely honest, you might even just use the custom resources
each Ingress controller supports, instead of the vanilla Kubernetes Ingress resource.

For this example we will set the default backend to the **hello-world** service. Here’s the updated
Ingress resource, with modified lines highlighted:

_ch3/default-backend-ing.yaml_

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: ambassador
  name: my-ingress
spec:
  defaultBackend:
    service:
      name: hello-world
      port:
       number: 3000
  rules:
    - host: example.com
      http:
        paths:
        - path: /hello
        pathType: Prefix
        backend:
          service:
           name: hello-world
           port:
             number: 3000
        - path: /dog
          pathType: Prefix
          backend:
            service:
              name: dogpic-service
              port:
                number: 3000
```

Save the above YAML to **default-backend-ing.yaml** and update the Ingress with **kubectl apply -f
default-backend-ing.yaml**. If you describe the Ingress resource using the **describe** command, you
will get a nice view of all rules and the default backend that we just set:

```
$ kubectl describe ing my-ingress
Name: my-ingress
Namespace: default
Address:
Default backend: hello-world:3000 (172.17.0.8:3000)
Rules:
  Host Path Backends
  ---- ---- --------
  example.com
        /hello hello-world:3000 (172.17.0.8:3000)
        /dog dogpic-service:3000 (172.17.0.9:3000)
Annotations: kubernetes.io/ingress.class: ambassador
Events: <none>
```

If you open http://example.com you will notice that this time the Hello World web page will load.
The **/hello** and **/dog** endpoints will still work the same way.

### **Name-based Ingress**
Sometimes you don’t want to use the fanout option with paths; instead, you want to route the traffic
based on the subdomains. For example, routing **example.com** to one service, **dogs.example.com** to
another, etc. For this example, we will try to set up the following rules:


| Host name | Kubernetes service |
|-----------|--------------------|
|example.com | hello-world:3000 |
| dog.example | dogpic-service:3000 |

To create the above rules, we need to add two **host** entries under the Ingress resource’s rules
section. We define the paths and the backend service and port name we wnat to route the traffic to
under each host entry.

_ch3/name-ing.yaml_

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: ambassador
  name: my-ingress
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /hello
            pathType: Prefix
            backend:
              service:
                name: hello-world
                port:
                  number: 3000
    - host: dog.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: dogpic-service
                port:
                  number: 3000
```

Save the above YAML to **name-ing.yaml** and deploy it using **kubectl apply -f name-ing.yaml**.
Before we can try this out, we need to add the **dog.example.com** to the **hosts** file just like we did with
the **example.com**. Open the **/etc/hosts** file (or **%SystemRoot%\System32\drivers\etc\hosts** on Windows)
and add the line mapping the **dog.example.com** hostname to the IP address. Make sure you use **sudo**
or open the file/terminal as an administrator on Windows.

```
$ sudo vim /etc/hosts
...
10.102.244.196 example.com
10.102.244.196 dog.example.com
...
```

__NOTE__:

> When using a real domain name, the entries we added to the **hosts** file would
correspond to the DNS records at your domains registrar. With an A record, you can
map a name (**example.com**) to a stable IP address. For **example.com**, you would create
an A record that points to the external IP address. Another commonly used record is
the CNAME record. You would use CNAME to map one name to another name. For
example, to map **dog.mydomain.com** to **dog.example.com**, while **dog.example.com** uses an
A record and maps to an IP. In the end, the **dog.mydomain.com** would resolve to the IP
address, same as **dog.example.com**.

Save the file, open the browser, and navigate to http://example.com. You should see the response
from the Hello World service as shown below.

![Figure 25. Hello World Website](https://i.gyazo.com/08760709a673ba49c3a9a379fe8f1f3e.png)
_Figure 25. Hello World Website_

Similarly, if you enter http://dog.example.com you will get the Dog Pic website.

![Figure 26. Dog Pic Service](https://i.gyazo.com/e60e5306ec3fea9bed17d619999ab66c.png)

_Figure 26. Dog Pic Service_

### **Cleanup**
You can delete the Service, Deployments, and Ingress using the **kubectl delete** command. For
example, to delete the **dogpic-web** deployment, run:

```
$ kubectl delete deploy dogpic-web
deployment.apps "dogpic-web" deleted
```

To delete a different resource, replace the resource name (**deploy** in the above example) with
**ingress** or **service**.

Another way of deleting the resources is to provide the YAML file you used to create the resource.
For example, if you created the **dogpic-web** from a file called **dogpic.yaml** you can delete it like this:

```
$ kubectl delete -f dogpic.yaml
```

If you get completely stuck and can’t delete something or delete too much (everyone has done that
at some point), you can always just reset your cluster. If you’re using Minikube, you can run
**minikube delete** to delete the cluster and afterward run **minikube start** to get a fresh cluster.
Similarly, you can reset the Kubernetes cluster from the Preferences menu when using Docker
Desktop.

### **Other Ingress controller responsibilities**

The job of an ingress controller or an ingress gateway is to proxy or "negotiate" the communication
between the client and the server. The client is anyone making requests, and the server is the
Kubernetes cluster or instead services running inside the cluster.

In addition to routing the incoming requests or exposing service APIs through a single endpoint, the
ingress gateways do other tasks, such as rate-limiting, SSL termination, load balancing,
authentication, circuit breaking, and more.

## Organizing applications with namespaces

Up until now, we haven’t talked about namespaces. Namespaces in Kubernetes provide a way to
scope and group different Kubernetes resources. Each namespace can contain multiple resources,
however, a single resource can only be in one namespace. The resource names need to be unique
within a namespace, but not across the namespaces.

For example, you can only have one service called **customers** inside a namespace called **production**.
However, you could create a **customers** service inside a namespace called **testing**. In thise case, the
full name of the Service would be **customers.production** and **customers.testing**.

You will create most of the Kubernetes resources inside a namespace. However some resources are
not in a namespace or are not namespaced. The example of a non-namespaced resource is the
namespace itself because namespaces cannot be nested. Other examples of non-namespaced
resources are Nodes, PersistentVolumes, CustomResourceDefinitions, and others.

__TIP__:

> You can get the full list of non-namespaced resources in your cluster by running
**kubectl api-resources --namespaced=false**. If you switch the flag value to **true**, you can
list all namespaced resources.

Typically you would use multiple namespaces in clusters with many users spread across different
projects or teams. You can deploy most of the off-the-shelf Kubernetes application using Helm
package manager into separate namespaces. Later in the book, when we talk about resource
quotas, we will show an example of how to divide the cluster resources between multiple users
using namespaces.

For the namespaced resource, you specify the namespace in the metadata section of the resource:

```
...
metadata:
  name: hello
  namespace: mynamespace
...
```

If you don’t provide the namespace, Kubernetes creates the resources in the default namespace.
The default namespace is called **default**, however, that can be changed per Kubernetes context. For
example, if you’re working with multiple clusters, you might want to set different default
namespace for your clusters.

Let’s consider the following output:

```
$ kubectl config get-contexts
CURRENT     NAME                  CLUSTER         AUTHINFO
NAMESPACE
            docker-for-desktop    docker-desktop  docker-desktop
            kind-kind             kind-kind       kind-kind
            minikube              minikube        minikube
*           peterjk8s             peterjk8s       clusterUser_mykubecluster_peterjk8s
```

Notice the namespace column is empty because I haven’t explicitly set namespaces for my contexts.
If I create a resource without specifying a namespace, it will end up in the **default** namespace.

You can create namespaces the same way you create other Kubernetes resources. You can either
define the namespace using YAML or use **kubectl**.

Let’s create a namespace called **testing** using **kubectl**:

```
$ kubectl create namespace testing
namespace/testing created

$ kubectl get namespace
NAME STATUS AGE
default Active 32d
kube-node-lease Active 32d
kube-public Active 32d
kube-system Active 32d
testing Active 3s
```

__TIP__:

> Short name for namespace is ns, so you can save typing seven characters!
The YAML representation of a namespace is straightforward, compared to some of the other
resources. You can get the YAML representation of any Kubernetes resource using the **--output yaml**
flag when running the **get** command:

```
$ kubect get ns testing --output yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2020-06-23T21:51:35Z"
  name: testing
  resourceVersion: "3750772"
  selfLink: /api/v1/namespaces/testing
  uid: 266e95ec-7de4-429c-a945-83d91a2a2296
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
```

The above output contains the field values such as **creationTimestamp**, **resourceVersion**, **selfLink**, and
others that Kubernetes adds when it creates the resource.

A cleaner way of getting the YAML representation of a resource is to add the **--dry-run** flag when
creating the resource. Any command you execute with the **--dry-run** flag will not be persisted and
won’t have any side-effects. That means nothing will get created, deleted, or modified. However,
this allows you to see how the resource would look like processed and persisted.

Let’s combine the dry run flag and the output flag and try again. Note that I am using the short
name for the output flag, -o:

```
$ kubectl create ns testing --dry-run=client -o yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: testing
spec: {}
status: {}
```

If we remove the empty and null fields, the YAML representation of the Namespace resource looks
like this:

```
apiVersion: v1
kind: Namespace
metadata:
  name: testing
```

You can also specify a namespace with each **kubectl** command. For example, if you want to get all
Pods inside the **kube-system** namespace, you can do that using the **--namespace** or -n flag:

```
$ kubectl get pods -n kube-system
NAME                            READY     STATUS    RESTARTS    AGE
coredns-66bff467f8-glmzm        1/1       Running   0           8d
coredns-66bff467f8-msvgx        1/1       Running   0           8d
etcd-minikube                   1/1       Running   0           8d
kube-apiserver-minikube         1/1       Running   0           8d
kube-controller-manager-minikube 1/1      Running   1           8d
kube-proxy-wqn8f                1/1       Running   0           8d
kube-scheduler-minikube         1/1       Running   0           8d
storage-provisioner             1/1       Running   0           8d
```

If you don’t explicitly provide a namespace, Kubernetes uses the **default** namespace. Similarly, you
can use the flag called **--all-namespaces** or **-A** to list resources across all namespaces. For example,
to list all Services in your cluster, regardless of the namespace, you would run:

```
$ kubectl get svc -A
NAMESPACE      NAME         TYPE         CLUSTER-IP      EXTERNAL-IP       PORT(S)
AGE
default        kubernetes   ClusterIP    10.96.0.1      <none>             443/TCP
8d
default        web-frontend LoadBalancer 10.106.30.168  <pending>          80:30962/TCP
8d
kube-system    kube-dns     ClusterIP    10.96.0.10     <none>
53/UDP,53/TCP,9153/TCP 8d
```

As mentioned in section [Using a Kubernetes Service]() a namespace plays a role when resolving
Services. Let’s consider the following listing of services inside a cluster:

```
$ kubectl get svc -A
NAMESPACE       NAME          TYPE        CLUSTER-IP      EXTERNAL-IP       PORT(S)
AGE
production      web-frontend  ClusterIP   10.96.0.1       <none>            80:30642/TCP
8d
testing         web-frontend  ClusterIP   10.96.0.2       <none>            80:30962/TCP
8d
```

We have two Services called **web-frontend**. One Service is in the **production** namespace and the other
in the testing namespace.

To correctly reference one or the other, you need to use the full name of the service. For example
**web-frontend.production.svc.cluster.local** for the service in the **production** namespace. If your
application that lives inside the **production** namespace sends a request to **web-frontend**, that
automatically resolves to the **web-frontend** service inside the **production** namespace. However, if you
want to reach the **web-frontend** service from the **testing** namespace, you will have to use a fully
qualified name, which is **web-frontend.testing.svc.cluster.local**. It is a good practice always to use
fully qualified names, so there’s no confusion on which service you are calling.

## Jobs and CronJobs

The type of workloads we were deploying previously were all long-running applications - things
like websites and services that keep on running continuously. If something went wrong, they get
rescheduled and start running again.

The other type of workloads you often need to run are workloads that perform a particular task,
and once the task is completed they stop running. An example of a workload like that would be
doing a backup or generating daily reports. It does not make sense to keep the reporting workload
running. It only needs to run when it’s generating the report. Once the task generates the report it
can go away. If the task fails, you can configured it to restart automatically.

Kubernetes features a resource called Job you can use to run such workloads. The Job resource can
create one or more Pods and track the number of successful completions. The Job resource ensures
that Pods are run to completion. You could achieve a similar behavior only with Pods, but then
you’d have to be the manage the Pods' lifecycle in case it fails or gets rescheduled.

Let’s run a Job that doesn’t nothing but sleep for a minute:

_ch3/sleep-job.yaml_

```
apiVersion: batch/v1
kind: Job
metadata:
  name: sleep-on-the-job
spec:
  template:
    metadata:
      labels:
        app.kubernetes.io/name: sleep-on-the-job
      spec:
        restartPolicy: Never
        containers:
          - name: sleep-container
            image: busybox
            args:
              - sleep
              - "60"
```

Save the above YAML in **sleep-job.yaml** and run **kubectl apply -f sleep-job.yaml** to create the Job.

__NOTE__:

> We are explicitly setting the **restartPolicy** to Never. The default value for
**restartPolicy** is Always, however, the Job resource does not support that restart
policy. The two supported values are Never and OnFailure. Setting one of these
values prevents the container from being restarted when it finishes running.

You can list the Job the same way as any other resource. Notice how the output also shows the
number of completions of the Job and duration of the Job.

```
$ kubectl get job
NAME              COMPLETIONS     DURATION    AGE
sleep-on-the-job  0/1             4s          4s
```

If you describe the Job you will notice in the Events section that controller creates a Pod to run the
Job:

```
$ kubectl describe job sleep-on-the-job
...
Events:
  Type    Reason            Age   From            Message
  ----    ------           ----   ----            -------
  Normal SuccessfulCreate   18s   job-controller  Created pod: sleep-on-the-job-f9ht8
...
```

Let’s look at the Pod that was created by this Job. We will use the **--labels** flag to get all Pods with
the label **app.kubernetes.io/name=sleep-on-the-job:**

```
$ kubectl get pods -l=app.kubernetes.io/name=sleep-on-the-job
NAME                      READY     STATUS    RESTARTS    AGE
sleep-on-the-job-f9ht8    1/1       Running   0           48s
```

After a minute, the Pod will stop running, however, it won’t be deleted. If you run the same
command as above, you will notice that the Pod is still around. Kubernetes keeps the Pod around so
you can look at the logs, for example.

```
$ kubectl get pods -l=app.kubernetes.io/name=sleep-on-the-job
NAME                    READY     STATUS      RESTARTS      AGE
sleep-on-the-job-f9ht8  0/1       Completed   0             28m
```

Similarly happens with the Job resource. The resource stays around until you explicitly delete it.
Deleting the Job also deletes the Pod. Notice how the number of completions now shows 1/1, which
means that the Job was completed successfully one time.

```
$ kubectl get job
NAME              COMPLETIONS       DURATION      AGE
sleep-on-the-job  1/1               63s           12m
```

If you’re wondering if a job can be completed and executed multiple times, the answer is yes, it can.
The Job can track the number of successful completions, and you can use that number as to control
when the Job completes.

By default, Kubernetes sets the number of completions to 1, and you can change that by setting a
different value to the **completions** field. Let’s create a new Job called **three-sleeps-on-the-job** where we set the completions to **3**. Setting it to **3** causes the Job’s Pod to run three times sequentially:

_ch3/three-sleeps.yaml_

```
apiVersion: batch/v1
kind: Job
metadata:
  name: three-sleeps-on-the-job
spec:
  completions: 3
  template:
    metadata:
      labels:
        app.kubernetes.io/name: three-sleeps-on-the-job
    spec:
      restartPolicy: Never
      containers:
        - name: sleep-container
          image: busybox
          args:
            - sleep
            - "60"
```

Save the above YAML in **three-sleeps.yaml** and deploy it with **kubectl apply -f three-sleeps.yaml**.

If you look at Jobs now, you will notice the **COMPLETIONS** column for the latest Job shows **0/3**:

```
$ k get job
NAME                      COMPLETIONS     DURATION     AGE
sleep-on-the-job          1/1             63s          40m
three-sleeps-on-the-job   0/3             10s          10s
```

As soon as the first job finishes (60 seconds later), the column will be updated and will show **1/3** in
the **COMPLETIONS** column. The Job executes Pods one after another, and Kubernetes marks the Job
completed when there are three successful completions (i.e., three pods ran without any failures).

If you need to execute Pods in parallel, you can use the **parallelism** setting. The parallelism setting
defines how many Pods you can run in parallel.

Let’s take the previous example and set the **parallelism** value to two:

_ch3/three-sleeps-parallel.yaml_

```
apiVersion: batch/v1
kind: Job
metadata:
  name: three-sleeps-on-the-job-parallelism
spec:
  completions: 3
  parallelism: 2
  template:
  metadata:
  labels:
  app.kubernetes.io/name: three-sleeps-on-the-job-parallelism
  spec:
  restartPolicy: Never
  containers:
  - name: sleep-container
  image: busybox
  args:
  - sleep
  - "60"
```

Save the above YAML in **three-sleeps-parallel.yaml** and deploy it with **kubectl apply -f three-sleeps-parallel.yaml**.

If you list the Pods now, you will notice two Pods running at the same time:

```
$ kubectl get pods
NAME                                        READY     STATUS      RESTARTS      AGE
three-sleeps-on-the-job-parallelism-b8ckz   1/1       Running     0             8s
three-sleeps-on-the-job-parallelism-zcrzq   1/1       Running     0             8s
```

What happens if the Pods keep failing? Well, the Job will keep creating them and retrying based on
the **backoffLimit** setting. The back-off limit is a setting on the spec that specifies the number of
retries before Kubernetes considers the Job failed. Kubernetes sets the default value to **6** and recreates them with an exponential back-off delay. The back-off delay means if the first Pod fails, the
controller will wait for 10 seconds before recreating it. Then if it fails again, it will wait for 20
seconds and so on up until a total of six minutes of delay. Kubernetes resets the back-off delay
either when you delete the Pod or when the Pod completes successfully.

In addition to the **backoffLimit**, you can also terminate a job using the **activeDeadlineSeconds**. The
**activeDeadlineSeconds** represents how long a Job can run, regardless of how many Pods it creates. If
we consider the previous example and set the **activeDeadlineSeconds** to 10, the Job will fail after 10
seconds. Kubernetes will terminate the Pods and set the Jobs' status to **DeadlineExceeded**.

Let’s look at this using an example:

_ch3/failing-job.yaml_

```
apiVersion: batch/v1
kind: Job
metadata:
  name: failing-job
spec:
  completions: 3
  activeDeadlineSeconds: 20
  template:
    metadata:
  l   abels:
        app.kubernetes.io/name: failing-job
  spec:
      restartPolicy: Never
      containers:
        - name: sleep-container
          image: busybox
          args:
            - sleep
            - "60"
```

Save the above YAML in **failing-job.yaml** and deploy it with **kubectl apply -f failing-job.yaml**.

The Job will create a Pod, but after 20 seconds, the Pod will get terminated, and the Job will fail with
the **DeadlineExceeded** reason:

```
$ kubectl describe job failing-job
...
Events:
  Type    Reason            Age     From            Message
  ----    ------           ----     ----            -------
  Normal  SuccessfulCreate  86s     job-controller  Created pod: failing-job-rq9ht
  Normal  SuccessfulDelete  66s     job-controller  Deleted pod: failing-job-rq9ht
  Warning DeadlineExceeded  66s     job-controller  Job was active longer than
specified deadline
```

If you ran all these examples, you have probably noticed that Kubernetes does not automatically
remove the Jobs and completed Pods. You can clean up the completed (or failed) Jobs by setting the
**ttlSecondsAfterFinished** value. After a Job completes (or fails) the controller waits for the duration
specified in the **ttlSecondsAfterFinished** field and then deletes the Job and Pods.

_ch3/delete-job.yaml_

```
apiVersion: batch/v1
kind: Job
metadata:
  name: delete-job
spec:
  ttlSecondsAfterFinished: 30
  template:
    metadata:
      labels:
        app.kubernetes.io/name: delete-job
    spec:
      restartPolicy: Never
      containers:
        - name: sleep-container
          image: busybox
          args:
            - sleep
            - "10"
```

Save the above YAML in **delete-job.yaml** and deploy it with **kubectl apply -f delete-job.yaml**.

The above Job will complete in 10 seconds. After that, the controller will wait for an extra 30
seconds before deleting the Job and the Pod. If you want to delete the Job right after it finishes, you
can set the **ttlSecondsAfterFinished** to 0.

Note that this feature is currently in alpha state. To use it, you need to enable alpha features in your
cluster.

__CronJobs__:

A CronJob is a type of a Job you can run at specified times and dates. The regular Jobs start running
when you create them. There are essential settings that allow you to control how Kubernetes runs
the Job. However, you cannot schedule the Job to run at a particular times or intervals. Running
Jobs at particular times or intervals is what the CronJob allows you to do.

The CronJob resource uses a well-known cron format, made out of five fields that represent the
time to execute the command.

![Figure 27. cron format](https://i.gyazo.com/4277d22a3f0c342901e29fc13cafde33.png)

_Figure 27. cron format_

The cron format is out of scope for this book, but here are a couple of examples:

| Example | Description |
|---------|-------------|
| * * * * * | Runs every minute, every hour, day, month and day of the week) |
| */10 * * * * | Runs every 10 minutes |
| 00 9,21 * * * | Runs at 9 AM and 9 PM, every day, month, and day of the week |
| 00 7-17 * * * | Run at the top of every hour, from 7 AM and 5 PM, every day, month, and day of the week |
| 00 7-17 * * 1-5 | Run at the top of every hour, from 7 AM and 5 PM, every day, month, but only during weekdays. |
| 0,15,30,45 * * * * | Runs every 15 minutes of every hour, day, month, and day of the week |

Let’s create a CronJob that runs every minute, so that we can see it in action. You can set the cron
format with the **schedule** field:

_ch3/minute-cron.yaml_

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: minute-cron
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app.kubernetes.io/name: minute-cron
        spec:
          restartPolicy: Never
          containers:
            - name: sleep-container
              image: busybox
              args:
                - sleep
                - "10"
```
Save the above YAML in **minute-cron.yaml** and deploy it with **kubectl apply -f minute-cron.yaml**.

Let’s look at the CronJob by running **kubectl get cronjob** or using the short name **kubectl get cj**:

```
$ kubectl get cj
NAME          SCHEDULE      SUSPEND     ACTIVE    LAST    SCHEDULE   AGE
minute-cron   */1 * * * *   False       0         <none>             3s
```

If you wait for a couple of minutes, you will see the CronJob automatically create the Pods every
minute.

To suspend the CronJob you can edit the resource (e.g. **kubectl edit minute-cron**) and change the
**suspend** field from **false** to **true**.

In addition to the CronJob schedule, you can also configure how CronJob deals with concurrent
executions. Let’s say you configured the CronJob to run every 5 minutes. The Pod starts, and for
some reason, it runs for more than 5 minutes. What should the CronJob controller do in this case?
Does it create the second Pod as per schedule or do nothing, since the previous Job is still
executing?

You can control this behavior using the concurrencyPolicy setting. This setting has three possible
values - Forbid, Allow, and Replace. The default value is Allow, and if the previous iteration hasn’t
completed when the new one is supposed to start, the allow setting will allow the second instance
to run.

Setting the value to **Forbid** will do the opposite - the Pod that was supposed to start per schedule will not start.

Finally, the **Replace** will stop the currently running Job and start a new one.

Another configuration value we need to mention here is the **startingDeadlineSeconds**. You would set
this in cases where you don’t want to start the job over the scheduled time. If the Job doesn’t start at
the scheduled time + the deadline, it will be marked as Failed.

Consider the following example:

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: ten-minute-cron
spec:
  schedule: "*/10 * * * *"
  startingDeadlineSeconds: 30
  ...
```

The Job needs to run every ten minutes. Let’s say the first Job is supposed to start at 2 PM (2 hours, 0
minutes, and 0 seconds). The **startingDeadlineSeconds** says that if the Job doesn’t begin by 2:00:30
the controller should mark it as failed.

# Configuration

One of the factors from the [Twelve-Factor App manifesto](https://12factor.net/) talks about application configuration,
specifically about storing configuration in the environment, instead of hardcoding it inside your
services.

In most cases, the application configuration will vary between different deployment environments.
For example, when you’re running your application in a testing or staging environment, it is highly
likely that you want your application to use databases or other backing services from the same
environment. You don’t want the application in a testing environment to talk to your production
database - that would be bad.

Another reason for separating code from your configuration is that you don’t want to rebuild your application each time you change a configuration value. That would be a massive waste of time and resources. Instead, you should always deploy the same application or binary, but augment its behavior using a configuration specific to the environment application is running in.

![Figure 28. Configuration per Environment](https://i.gyazo.com/1ee433389c38a8f593b672ba98fbaf08.png)

_Figure 28. Configuration per Environment_:

Typically configuration is anything that your application needs to be able to start and run. Here are
a couple of standard configuration settings:

- Connection strings to databases, queues, or other connection strings
- Credentials (any usernames, passwords, keys, certificates)
- Port numbers, dependent service names, and addresses

We can separate these configuration settings into two groups at a high-level: __non-sensitive__
configuration values and __sensitive__ configuration values.

The former contains anything that you don’t consider to be sensitive information - things like port
numbers, service names, or perhaps even connection strings, assuming they don’t contain any
usernames or passwords. The latter group includes sensitive information or secrets. Things like passwords, API keys, application secrets, credentials, certificates, etc. Pretty much anything that can
severely compromise your application or your system if this information is leaked.

## Configuring application through arguments

The simplest way to configure the application running inside Kubernetes is to set the command and
pass arguments to it - we have already done that in the previous chapter, where we talked about
Pods. Here’s an example we used:

_ch4/hello-pod.yaml_
```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
  labels:
    app.kubernetes.io/name: hello
spec:
  containers:
    - name: hello-container
      image: busybox
      command: ["sh", "-c", "echo Hello from my container!"]
```

With the above YAML, we are invoking sh and then echoing the "Hello from my container!"
message. Another way you can do this is to specify "echo" as the command and pass-in the message
as an argument, just like in the YAML below:

_ch4/hello-pod-args.yaml_

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
  labels:
    app.kubernetes.io/name: hello
spec:
  containers:
    - name: hello-container
      image: busybox
      command: ["echo"]
      args: ["Hello from my container!"]
```

In some cases (depending on how you defined your Dockerfile), you could even omit the command
and specify the arguments. Here’s a snippet from the Traefik deployment YAML and how the
arguments are passed to the Traefik Docker image:

```
...
  spec:
    containers:
      - name: router
        image: traefik:v2.3
        args:
          - --entrypoints.web.Address=:8000
          - --entrypoints.websecure.Address=:4443
          - --providers.kubernetescrd
          - --accesslog=true
          - --accesslog.filepath=/var/log/traefik.log
          - --certificatesresolvers.myresolver.acme.tlschallenge
...
```

In the above case, the Dockerfile for the **traefik** image is using **ENTRYPOINT** command to point to the
**traefik** binary inside the container. Therefore, you don’t need to set the command name explicitly.
Here’s a snippet from the Traefiks' Dockerfile:

```
...
EXPOSE 80
VOLUME ["/tmp"]
ENTRYPOINT ["/traefik"]
```

Let’s quickly look at what the difference between the CMD and ENTRYPOINT is.

### **Difference between CMD and ENTRYPOINT in Dockerfiles**

With the **ENTRYPOINT** instruction, you can specify a command that Docker executes when the
container starts. On the other hand, with **CMD** you can specify arguments that get passed to the
**ENTRYPOINT** instruction. Let’s consider the following Dockerfile:

_ch4/Dockerfile_

```
FROM alpine
RUN apk add curl

ENTRYPOINT ["/usr/bin/curl"]
CMD ["google.com"]
```

If you build this image (let’s call it **curlalpine**) and then run it without any arguments, you will get
back the response from google.com

```
$ docker run -it curlalpine
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```

If you run the same image with an argument **example.com**, the container will curl to the **example.com**:

```
$ docker run -it curlalpine example.com
<!doctype html>
<html>
<head>
  <title>Example Domain</title>
...
```

Now let’s say we modified the Dockerfile, removed the ENTRYPOINT and invoked the curl
**google.com** within the CMD instruction:

_ch4/Dockerfile.cmd_

```
FROM alpine
RUN apk add curl

CMD ["/usr/bin/curl", "google.com"]
```

Running the container without any arguments will invoke "curl google.com", just like before.
However, if you pass in an argument (example.com, like we did before), you will get an error:

```
$ docker run -it curlcmd example.com
docker: Error response from daemon: OCI runtime create failed: container_linux.go:349:
starting container process caused "exec: \"example.com\": executable file not found in
$PATH": unknown.
ERRO[0000] error waiting for container: context canceled
```

The container is trying to run the argument, which in our case, doesn’t make any sense (unless you
have a binary called **example.com** in the container). If you replace **example.com** with a command that
exists inside the container (let’s say **ls**), the container will execute that command:

```
$ docker run -it curlcmd ls
bin     etc     lib     mnt     proc    run     srv     tmp     var
dev     home    media   opt     root    sbin    sys     usr
```

In addition to passing arguments to containers, Kubernetes also has dedicated resources to store
the configuration. A resource for storing configuration settings is called a ConfigMap, and the
resource for storing secret configuration values is named Secret. Let’s look at the ConfigMaps first.

## Creating and using ConfigMaps

A ConfigMap resource can store configuration values (key-value pairs). These values can be
consumed or 'mounted' to containers inside a Pod either as environment variables or files. Here’s a
YAML representation of a ConfigMap that stores single values ("8080", "Ricky") as well as values that
look like fragments of a configuration file (e.g. "someVariable" and "anotherName" under the
**settings.env** key):

_ch4/simple-config.yaml_

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: simple-config
  namespace: default
data:
  portNumber: "8080"
  name: Ricky
  settings.env: |
    someVariable=5
    anotherName=blah
```

Save the above YAML into a file called **simple-config.yaml** and create it by running **kubectl apply -f
simple-config.yaml**.

To see the details of the ConfigMap you create, you can use the **describe** command:

```
$ kubectl describe cm simple-config
Name: simple-config
Namespace: default
Labels: <none>
Annotations:
Data
====
name:
\----
Ricky
portNumber:
\----
8080
settings.env:
\----
someVariable=5
anotherName=blah

Events: <none>
```
For Pod to use the values from the ConfigMap, both Pod and ConfigMap need to reside in the same
namespace. You can mount a ConfigMap to your Pod and use the values in one of the following
ways:
- As command-line arguments
- As environment variables
- As a file in a read-only-volume
- Through Kubernetes API that reads the ConfigMap

Let’s create a Pod and show how to consume the values from a ConfigMap.

_ch4/config-pod-1.yaml_

```
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
  labels:
    app.kubernetes.io/name: config-pod
spec:
  containers:
    - name: config
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      env:
        - name: FIRST_NAME
          valueFrom:
            configMapKeyRef:
              name: simple-config
              key: name
        - name: PORT_NUMBER
          valueFrom:
            configMapKeyRef:
              name: simple-config
              key: portNumber
```

We are using the **valueFrom** and **configMapKeyRef** to specify where the value for the environment
variable is coming from. The environment variable name (e.g. **FIRST_NAME**) is and can be different
from the key name stored in the ConfigMap (e.g. **name** or **portNumber**).

Save the above YAML in **config-pod-1.yaml** and create the Pod with **kubectl apply -f config-pod-1.yaml**. Once the Pod is running, let’s get the terminal inside the container and look at the
environment variables:

```
$ kubectl exec -it config-pod -- /bin/sh
/ # env
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT=443
HOSTNAME=config-pod
SHLVL=1
HOME=/root
PORT_NUMBER=8080
TERM=xterm
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
FIRST_NAME=Ricky
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
```

Notice the **PORT_NUMBER** and **FIRST_NAME** environment variables values are coming from the
ConfigMap.

How about the values under the **settings.env** key? Let’s mount that as a file inside the Pod. Here’s
the updated YAML for the **config-pod**:

_ch4/config-pod-2.yaml_

```
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
  labels:
    app.kubernetes.io/name: config-pod
spec:
  containers:
    - name: config
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      env:
        - name: FIRST_NAME
          valueFrom:
            configMapKeyRef:
              name: simple-config
              key: name
    - name: PORT_NUMBER
      valueFrom:
        configMapKeyRef:
          name: simple-config
          key: portNumber
  volumeMounts:
    - name: config
      mountPath: "/config"
      readOnly: true
volumes:
  - name: config
    configMap:
      name: simple-config
      items:
        - key: "settings.env"
          path: "local.env"
  ```

To bring in the **settings.env** from the ConfigMap, we create a Volume called **config** from the
ConfigMap and mount the items (or, in our case, just one item called **settings.env**) to the path called
**local.env**. Mounting creates a file **local.env** file containing the key-value pairs defined under the
key **settings.env** from the ConfigMap. Using a **volumeMount** field, we mount the **config** volume under
the **/config** folder inside the container.

Before deploying the above YAML, make sure you delete the previous Pod first, by running **kubectl
delete pod config-pod**. Then, save the above YAML in **config-pod-2.yaml** and apply it with **kubectl
apply -f config-pod-2.yaml**.

Once the Pod is deployed, let’s use the **exec** command to get a shell inside the container and look in
the **/config** folder:

```
$ kubectl exec -it config-pod -- /bin/sh
/ # cat config/local.env
someVariable=5
anotherName=blah
/ #
```

In the previous two examples, we were setting two configuration settings as environment variables
and mounting the settings.env as a file from a volume. Instead of mounting configuration as
environment variables, we can mount the whole ConfigMap as a Volume. In this case, each
configuration setting name (e.g., portNumber, name) will be created as a file containing the respective
value.

Once you’ve delete the previous pod (kubectl delete po config-pod), deploy the following YAML:

_ch4/config-pod-3.yaml_

```
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
  labels:
    app.kubernetes.io/name: config-pod
spec:
  containers:
    - name: config
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: config
          mountPath: "/config"
          readOnly: true
  volumes:
    - name: config
      configMap:
        name: simple-config
```
The above YAML looks similar to what you’ve seen before. The only difference is that we aren’t
explicitly calling out any configuration settings - we are merely creating a volume (**config**) from a
ConfigMap and then mounting that volume under **/config** folder inside the Pod.

If you look at the contents of the **/config** folder inside the container, you will notice that there’s a
separate file created for every configuration setting from the ConfigMap:

```
$ kubectl exec -it config-pod -- ls /config
name          portNumber      settings.env
```

Each of the files contains the value(s) set in the ConfigMap:

```
$ kubectl exec -it config-pod -- cat /config/name
Ricky
```

The mounted ConfigMaps are updated automatically. Let’s edit the **simple-config** ConfigMap and
change the **portNumber** from **8080** to **5000**. Run **kubectl edit cm simple-config** to launch the editor
from the terminal and change the value of **portNumber** to **5000**. Save the changes and exit the editor.

Kubelet periodically checks if the ConfigMap was updated and sets the new value. If you check the
value of the **/config/portNumber** file, you will notice that the value is updated to **5000**:

```
$ kubectl exec -it config-pod -- cat /config/portNumber
5000
```

Similarly, you can use **envFrom** field to automatically set all configuration settings from a ConfigMap
as environment variables. Here’s how the Pod YAML looks like in this case:
_ch4/pod-env-prefix.yaml_

```
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
  labels:
    app.kubernetes.io/name: config-pod
spec:
  containers:
    - name: config
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      envFrom:
        - prefix: SIMPLE_CONFIG_
          configMapRef:
            name: simple-config
```

Instead of using the **env** field, you use the **envFrom**, specify the prefix you want to set to each entry,
and reference the ConfigMap. If you don’t set the prefix, the variable names will look like this:

```
portNumber
name
settings.env
```

With the prefix set to **SIMPLE_CONFIG_**, Kubernetes names the variables like this:

```
SIMPLE_CONFIG_portNumber
SIMPLE_CONFIG_name
SIMPLE_CONFIG_settings.env
```

The prefix is optional, and you don’t have to set it. However, it might make sense to select it,
especially if you have values coming from different ConfigMaps, and want to group them.

If you deploy the above YAML and run the env command in the container, you will see the
environment variables being set with the prefix:

```
$ kubectl exec -it config-pod -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=config-pod
TERM=xterm
SIMPLE_CONFIG_name=Ricky
SIMPLE_CONFIG_portNumber=8080
SIMPLE_CONFIG_settings.env=someVariable=5
anotherName=blah
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
HOME=/root
```

Up until now, you’ve been creating ConfigMaps through YAML. In some cases, though, that might
not be practical. Think about having an environment variable file with 10+ key-value pairs - it
doesn’t make sense to re-type all of them to create the YAML for the ConfigMap. For this purpose,
you can also use Kubernetes CLI to create the ConfigMaps.

### **Creating ConfigMaps using Kubernetes CLI**

The Kubernetes CLI supports creating ConfigMaps in three different ways:

- Creating key-value pairs using **--from-literal** setting
- Creating key-value pairs from an environment file (list of **key=value** pairs) using **--from-env-file**
setting
- Creating key-value pairs from the file name and file contents using --from-file setting

Let’s look at the first option where you can create key-value pairs using the **--from-literal** setting:

```
$ kubectl create cm literal-config --from-literal=portNumber=8080 --from-literal
=firstName=Ricky
configmap/literal-config created

$ kubectl describe cm literal-config
Name:         literal-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
firstName:
\----
Ricky
portNumber:
\----
8080
Events: <none>
```

You should use the **--from-env-file** setting if you are using **.env** files in your projects. Let’s consider
the following **local.env** file:

```
PORT=8080
SERVICE_URL=http://myservice
FIRST_NAME=Ricky
RETRIES=10
```

You could use **--from-literal** for each of the settings, but it’s much easier to use **--from-env-file**
command like this:

```
$ kubectl create cm env-config --from-env-file=local.env
configmap/env-config created

$ kubectl describe cm env-config
Name:         env-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
FIRST_NAME:
\----
Ricky
PORT:
\----
8080
RETRIES:
\----
10
SERVICE_URL:
\----
http://myservice
Events: <none>
```

Finally, let’s consider a scenario where you’re storing configuration settings in a **config.json** file
that looks like this:

```
{
  "portNumber": "5000",
  "firstName": "Ricky",
  "serviceUrl": "http://myservice"
}
```

It is fairly safe to assume that your applications consume the JSON configuration as is - so it
wouldn’t make sense to split the file into regular key-value pairs and mount them as environment
variables or something like that. You need a single configuration item that has the JSON contents as
its value. To do this, you can use the **--from-file** setting:

```
$ kubectl create cm fromfile-cm --from-file=config.json
configmap/fromfile-cm created

$ kubectl describe cm fromfile-cm
Name:         fromfile-cm
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
config.json:
\----
{
  "portNumber": "5000",
  "firstName": "Ricky",
  "serviceUrl": "http://myservice"
}

Events: <none>
```

The **--from-file** setting will use the file name (**config.json**) as the configuration key, and the value
will be the contents of the file. If you have multiple configuration files, you can also pass in the
folder name to the **--from-file** setting. The CLI will iterate through the files in the folder and create
the configuration items, just like a single file.

You can combine one of the file options with the literal option in a single command and create a
ConfigMap from multiple sources. For example:

```
$ kubectl create cm combined-cm --from-file=config.json --from-literal=HELLO=world
```

Note that the command fails if you use more than one option that reads a file (e.g. **--from-file** and
**--from-env-file**) together in the same command.

## Storing secrets in Kubernetes

The Secret resource in Kubernetes is similar to ConfigMaps. Just like with ConfigMaps, you can use
Secrets to store key-value pairs. You can mount them into your containers either as files in a
volume or use them as environment variables.

The choice between using a ConfigMaps or a Secret is straightforward: if the value you’re storing
contains sensitive information (keys, passwords, credentials, etc.) store it in a Secret. Otherwise,
store the values in a ConfigMap. The Secret resource values are stored encrypted in the etcd (etcd is
the key-value store used by Kubernetes to store the objects and their state).

There are three types of Secrets you can create as shown in the 
table below.

_Table 2. Secrets_

| Secret type | Description | Example |
|-------------|-------------|---------|
| generic | Secret that can be created from a file, directory or a literal value | kubectl create secret generic service-auth --from -literal=username=ricky --from -file=password=password -file.txt |
| docker-registry | Secret for use with Docker registry. You can pass in .dockercfg file or manually specify values needed to authenticate with the Docker registry (docker-server, docker-username, docker-password, docker-email) |kubectl create secret docker-registry my-docker-registry --docker -server=https://index.docker.i o/v1/ --docker -username=username --docker -password=ILOVEPIZZA2 |
| tls | Secret that holds the public/private key pair. | kubectlcreate tls my-tls-secret --cert=my/certs/tls.cert --key=/my/certs/tls.key |


Let’s create a generic secret using kubectl:

```
$ kubectl create secret generic service-auth --from-literal=username=ricky --from
-literal=password=ILOVEPIZZA2
secret/service-auth created
```

You can now use the describe command to describe the Secret:

```
$ kubectl describe secret service-auth
Name:         service-auth
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type: Opaque

Data
====
password: 11 bytes
username: 5 bytes
```

If you remember from earlier when you describe a ConfigMap, you could see the actual key values.
Let’s create a ConfigMap with the same two values and describe it:

```
$ kubectl create cm service-auth-cm --from-literal=username=ricky --from-literal
=password=ILOVEPIZZA2
configmap/service-auth-cm created

$ kubetl describe cm service-auth-cm
Name: service-auth-cm
Namespace: default
Labels: <none>
Annotations: <none>

Data
====
password:
\----
ILOVEPIZZA2
username:
\----
ricky
Events: <none>
```

When describing the ConfigMap, you can see the actual key values (the username and password),
while in the Secret you only know the size (11 and 5 bytes).

Let’s also look at the YAML representation of the Secret to see how the values are stored:

```
$ kubectl get secret service-auth -o yaml
apiVersion: v1
data:
  password: SUxPVkVQSVpaQTI=
  username: cmlja3k=
....
```
.>You can add the **-o yaml** flag to the **get** command to show the YAML representation of any resource
in Kubernetes:

Kubernetes stores the values as a Base64-encoded string. Encoding the values like that allows you to
store not just plain text, but also binary data. If you are creating a secret through YAML, you will
have to Base64-encode all binary values. However, if you have any secrets that don’t need to be
Base64-encoded (i.e., non-binary values), you can provide them in plain text using the **stringData**
field.

Here’s the same Secret as before, but in this case, we are providing the **username** in plain text, while
the password is still provided as Base64-encoded string:

_ch4/svc-auth-2.yaml_

```
apiVersion: v1
kind: Secret
metadata:
  name: service-auth-2
  namespace: default
stringData:
  username: ricky
data:
  password: SUxPVkVQSVpaQTI=
```

Save the above YAML in **svc-auth-2.yaml** file and create it using this command: **kubectl apply -f svc-auth-2.yaml**.

If you get the YAML representation, you will see that both values still end up being Base64-encoded
and the field **stringData** is omitted (Kubernetes uses it to create the Base64-encode entries under
the **data** field):

```
$ kubectl get secret service-auth-2 -o yaml
apiVersion: v1
data:
  password: SUxPVkVQSVpaQTI=
  username: cmlja3k=
kind: Secret
metadata:
....
```

Just like with ConfigMaps, Pods can consume Secrets through environment variables and volumes.
When you use the Secret is a Pod, the values are stored as plain text. You don’t have to Base64-
decode the values in your containers.

### **Consuming Secrets as environment variables**

Let’s look at how you can consume the values from a Secret we created earlier as environment
variables:

_ch4/pod-secret.yaml_

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
  labels:
    app.kubernetes.io/name: secret-pod
spec:
  containers:
    - name: secret
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      env:
        - name: USERNAME
          valueFrom:
            secretKeyRef:
              name: service-auth
              key: username
        - name: PASSWORD
          valueFrom:
            secretKeyRef:
              name: service-auth
              key: password
```

Save the above YAML in **pod-secret.yaml** and deploy it using **kubectl apply -f pod-secret.yaml**.

If you compare the above to the example we did in the ConfigMaps section you won’t see many
differences. The difference is in the field name. When referencing a Secret, you use **secretKeyRef**,
instead of **configMapKeyRef** when you reference a ConfigMap.

Once the Pod is running, let’s look at the environment variables set in the container:

```
$ kubectl exec -it secret-pod -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=secret-pod
TERM=xterm
USERNAME=ricky
PASSWORD=ILOVEPIZZA2
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
HOME=/root
```

Note that we didn’t have to do any Base64-encoding, and the Secret values show up encoded. To
delete this Pod, run **kubectl delete pod secret-pod**.

Even though you can use secrets as environment variables, it is __not recommended__ as it is much
easier to expose them unintentionally. For example, you might be writing all environment variables
to log files when your application crashes or when the application starts up. To be safe, you should
instead use volumes to mount secrets into Pods.

### **Consuming Secrets from Volumes**

Let’s look at an example of how to mount the Secret values through files. Mounting Secrets as files
is the preferred and more secure way of consuming secrets than using them through environment
variables.

_ch4/secret-pod-volume.yaml_

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
  labels:
    app.kubernetes.io/name: secret-pod
spec:
  containers:
    - name: secret
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: auth-secret
          mountPath: "/var/secrets"
          readOnly: true
  volumes:
    - name: auth-secret
      secret:
        secretName: service-auth
```

We define the volume holding the Secret similarly as we did the ConfigMap. We are using the **secret** field and then providing the **secretName**. Finally, we are mounting the volume using the **volumeMounts** field and specifying the volume name and the container’s location where we want to store the secrets.

Save the above YAML in **secret-pod-volume.yaml** and create the Pod using **kubectl apply -f secret-pod-volume.yaml**.

If you run the ls command inside the container, you will notice two files: password and username.
These two files contain the Secret values in plain-text.

```
$ kubectl exec -it secret-pod -- ls /var/secrets
password username

$ kubectl exec -it secret-pod -- cat /var/secrets/username
ricky

$ kubectl exec -it secret-pod -- cat /var/secrets/password
ILOVEPIZZA2
```

# Stateful Workloads

Running stateful workloads inside Kubernetes is different from running stateless services. The
reason being is that the containers and Pods can get created and destroyed at any time. If any of the
cluster nodes go down or a new node appears, Kubernetes needs to reschedule the Pods.

If you ran a stateful workload or a database in the same way you are running a stateless service, all
of your data would be gone the first time your Pod restarts.

Therefore we need to store the data outside of the container. Storing the data outside ensures that
nothing happens to it when the container restarts.

## What are Volumes?

The Volume abstraction in Kubernetes solves this problem. The Volume lives as long as the Pod
lives. If any of the containers within the Pod get restarted, Volume preserves the data. However,
once you delete the Pod, the Volume gets deleted as well.

![Figure 29. Volumes in a Pod](https://i.gyazo.com/576020dcfabff34a5f9d06fd6ba53b67.png)

_Figure 29. Volumes in a Pod_

The Volume is just a folder that may or may not have any data in it. The folder is accessible to all
containers in a pod. How this folder gets created and the backing storage is determined by the
volume type.

The most basic volume type is an empty directory (**emptyDir**). When you create a Volume with the **emptyDir** type, Kubernetes creates it when it assigns a Pod to a node. The Volume exists for as long
as the Pod is running. As the name suggests, it is initially empty, but the containers can write and
read from the Volume. Once you delete the Pod, Kubernetes deletes the Volume as well.

There are two parts to using the Volumes. The first one is the Volume definition. You can define the
volumes in the Pod spec by specifying the volume name and the type (**emptyDir** in our case). The
second part is mounting the Volume inside of the containers using the **volumeMounts** key. In each Pod
you can use multiple different Volumes at the same time.

Inside the volume mount we refer to the Volume by name (pod-storage) and specifying which path
we want to mount the Volume under (**/data/**).

_ch5/empty-dir-pod.yaml_

```
apiVersion: v1
kind: Pod
metadata:
  name: empty-dir-pod
spec:
  containers:
    - name: alpine
      image: alpine
      args:
        - sleep
        - "120"
      volumeMounts:
        - name: pod-storage
          mountPath: /data/
  volumes:
    - name: pod-storage
      emptyDir: {}
```

Save the above YAML in **empty-dir-pod.yaml** and run **kubectl apply -f empty-dir.pod.yaml** to create
the Pod.

Next, we are going to use the **kubectl exec** command to get a terminal inside the container:

```
$ kubectl exec -it empty-dir-pod -- /bin/sh
/ # ls
bin   dev   home  media   opt   root  sbin  sys   usr
data  etc   lib   mnt     proc  run   srv   tmp   var
```

If you run **ls** inside the container, you will notice the **data** folder. The **data** folder is mounted from
the **pod-storage** Volume defined in the YAML.

Let’s create a dummy file inside the **data** folder and wait for the container to restart (after 2
minutes) to prove that the data inside the **data** folder stays around.

From inside the container create a **hello.txt** file under the **data** folder:

```
echo "hello" >> data/hello.txt
```

You can type exit to exit the container. If you wait for 2 minutes, the container will automatically
restart. To watch the container restart, run the **kubectl get po -w** command from a separate terminal window.

Once container restarts, you can check that the file **data/hello.txt** is still in the container:

```
$ kubectl exec -it empty-dir-pod -- /bin/sh
/ # ls data/hello.txt
data/hello.txt
/ # cat data/hello.txt
hello
/ #
```

Kubernetes stores the data on the host under the **/var/lib/kubelet/pods** folder. That folder contains a list of pod IDs, and inside each of those folders is the **volumes**. For example, here’s how you can get
the pod ID:

```
$ kubectl get po empty-dir-pod -o yaml | grep uid
  uid: 683533c0-34e1-4888-9b5f-4745bb6edced
```

Armed with the Pod ID, you can run **minikube ssh** to get a terminal inside the host Minikube uses to
run Kubernetes. Once inside the host, you can find the **hello.txt** in the following folder:

```
$ sudo cat /var/lib/kubelet/pods/683533c0-34e1-4888-9b5f-4745bb6edced/volumes/kubernetes.io~empty-dir/pod-storage/hello.txt
hello
```

If you are using Docker Desktop, you can run a privileged container and using nsenter run a shell
inside all namespace of the process with id 1:

```
$ docker run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
/ #
```

Once you get the terminal, the process is the same - navigate to the **/var/lib/kubelet/pods** folder
and find the **hello.txt** just like you would if you’re using Minikube.

Kubernetes supports a large variety of other volume types. Some of the types are generic, such as
**emtpyDir** or **hostPath** (used for mounting folders from the nodes' filesystem). Other types are either
used for __cloud-provider storage__ (such as **azureFile**, **awsElasticBlockStore**, or **gcePersistentDisk**),
__network storage__ (**cephfs**, **cinder**, **csi**, **flocker**, …), or for mounting Kubernetes resources into the
Pods (**configMap**, **secret**).

Lastly, another particular type of Volumes are Persistent Volumes and Persistent Volume Claims we
discuss in [Persisting data with Persistent Volumes and Persistent Volume Claims]().

The lack of the word "persistent" when talking about other volumes can be misleading. If you are
using any cloud-provider storage volume types (**azureFile** or **awsElasticBlockStore**), the data will
still be persisted. The persistent volume and persistent volume claims are just a way to abstract
how Kubernetes provisions the storage.

For the full and up-to-date list of all volume types, check the [Kubernetes Docs](https://kubernetes.io/docs/contepcts/storage/volumes)


## Persisting data with Persistent Volumes and Persistent Volume Claims

A persistent volume (**PersistentVolume** or PV) is another volume type you can use to mount a persistent volume into a Pod. 

Using a PV and persistent volume claim (**PersistentVolumeClaim** or PVC), you can claim a portion of
durable storage (such as persistent disks from cloud providers) without knowing any details about
the cloud environment and how that storage was provisioned or created.

If you think about this as a user or a developer, you only want to get a piece of durable storage to
store your apps' data, but you don’t necessarily care about the data’s location.

You can create PVs in two different ways:
1. Provisioned by cluster administrators
2. Dynamically provisioned using Storage Classes

Once the persistent volume is created (either by the administrator or dynamically using a storage
class), you need a way to request it or claim it. You use the persistent volume claim
(**PersistentVolumeClaim** or PVC) to request or claim the volume. Inside the PVC, you can request a
volume of a specific size and different access modes. For example, you might have multiple
persistent volumes available - one optimized for heavy reads, other optimized for writes, etc.

When you create a PVC, a Kubernetes controller will try to match the PVC with a requested PV and
bind them together. In case the matching PV does not exist, the claim remains unbound. As soon as
the PV is available again, the PVC will get bound to it.

### **Storage classes**

A storage class (**StorageClass**) is a resource that describes a type of storage. A cluster administrator
can create multiple storage classes with different provisioners and properties. Here’s an example of
a StorageClass that uses an Azure Disk as its provisioner:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azure-disk-slow
provisioner: kubernetes.io/azure-disk
parameters:
  skuName: Standard_LRS
  location: westus
  storageAccount: [storage-account-name]
```
If **azure-disk-slow** storage class is requested as part of the persistent volume claim, Kubernetes will
use the storage class information to either use the existing storage account or create a new one.

The parameters for each type of storage class depend on the provisioner. For Azure Disk
provisioner, we set the **skuName**, **location**, and **storageAccount**. The Glusterfs provisioner might
require us to set parameters such as URL cluster ID, secret name, etc.

A Kubernetes cluster running with Docker Desktop uses a **hostpath** storage class provisioner. You
can get a list of all storage classes in your cluster by running the following command:

```
# Docker Desktop
$ kubectl get storageclass
NAME                PROVISIONER         AGE
hostpath (default)  docker.io/hostpath  44h
```

Similarly, a cluster running with Minikube has the following storage class:

```
# Minikube
$ kubectl get storageclass
NAME                  PROVISIONER               RECLAIMPOLICY       VOLUMEBINDINGMODE
ALLOWVOLUMEEXPANSION AGE
standard (default)    k8s.io/minikube-hostpath  Delete              Immediate
false                2m49s
```

You will also notice the word __default__ next to the **hostpath** storage class name. You can mark a
storage class as default, so when, for example, you request some storage using a PVC without
specifying the storage class, Kubernetes uses the default storage class.

### **Creating Persistent Volumes**

Let’s start by creating a persistent volume using the hostPath volume plugin. In the resource, we
define the storage capacity (5 gibibytes), the access mode (**ReadWriteOnce**), and a host path of
**/mnt/data**. This means that we will allocate 5Gi on the node in the folder **/mnt/data**, and the Volume
can only be mounted as read-write by a single node.

There are three different access modes we can choose from:

_Table 3. Access Modes_

| Access mode | Short name | Description |
|-------------|------------|-------------|
| ReadWriteOnce | RWO | The volume can be mounted as read-write by a single node only |
| ReadOnlyMany | ROX | The volume can be mounted read-only by many nodes |
| ReadWriteMany | RWX | The volume can be mounted as read-write by many nodes |

Note that not all volume plugins support all access modes. For example, the **hostPath** plugin only
supports the **ReadWriteOnce** access mode, and so does the Azure Disk plugin. However, Azure File,
Glusterfs, and a couple of other plugins support all three access modes.

Let’s save the YAML below in **local-pv.yaml** and run **kubectl apply -f local-pv.yaml** to create the
PersistentVolume.

_ch5/local-pv.yaml_

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv-volume
spec:
  storageClassName: manual
  capacity:
  storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

You can view the created persistent volume by running the following command:

```
$ kubectl get pv
NAME            CAPACITY    ACCESS MODES    RECLAIM POLICY    STATUS    CLAIM
STORAGECLASS    REASON    AGE
local-pv-volume 5Gi         RWO             Retain            Available
manual                    16s
```

The reclaim policy for the volume (**Retain** in the above case) specifies that Kubernetes retains the
data, and it’s up to the user/administrator to reclaim the space manually. The other two options are
**Recycle** where volume contents are deleted, and **Delete** that applies to cloud-provider backed
storage where the storage resource is deleted.

Now that we have the persistent volume, we can create a persistent volume claim to request
storage. Let’s look at the YAML first and then explain the different fields:

_ch5/local-pvc.yaml_

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pv-claim
spec:
  storageClassName: manual
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Using the above claim we request 1Gb of storage using the **manual** storage class name and RWO
access mode. Surprise, surprise, this exactly matches the persistent volume we create before.

Save the above YAML in **local-pvc.yaml** and run **kubectl apply -f local-pvc.yaml** to create the
PersistentVolumeClaim.

If we look at the status of the persistent volume and run the **kubectl get pv** command, you will
notice the **STATUS** column shows the **Bound** status and the **CLAIM** column shows the name of the claim:

```
$ kubectl get pv
NAME              CAPACITY      ACCESS MODES      RECLAIM POLICY    STATUS    CLAIM
STORAGECLASS REASON AGE
local-pv-volume   5Gi           RWO               Retain            Bound     default/local-pv-claim manual     14m
```

You can also see the Volume name and status by looking at the PVC:

```
$ kubectl get pvc
NAME             STATUS    VOLUME           CAPACITY     ACCESS MODES    STORAGECLASS
AGE
local-pv-claim   Bound     local-pv-volume  5Gi          RWO             manual
19s
```

Before we create a Pod that consumes this Volume, let’s create a file in the **/mnt/data** folder on the
host.

If you’re using Docker Desktop, you can use the command below to get into the virtual machine
(host):

```
docker run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```

If you’re using Minikube, the equivalent command is **minikube ssh**.

Once inside the host, let’s start by creating the **/mnt/data** folder. If you’re using Docker Desktop,
create the folder under the **/containers/services/docker/rootfs** folder. If using Minikube, just
create the **/mnt/data** folder.

The data folder will have an **index.html** file inside. In the Pod, we will run an Nginx container that
will show that **index.html** file.

```
# create the folder (Docker Desktop)
mkdir -p /containers/services/docker/rootfs/mnt/data

# create the folder (Minikube)
sudo mkdir -p /mnt/data

# create index.html (Docker Desktop)
echo "Hello from Storage!" >> /containers/services/docker/rootfs/mnt/data/index.html

# create index.html (Minikube)
sudo sh -c "echo 'Hello from Storage!' > /mnt/data/index.html"

# exit the shell
exit
```

With the volume populated, let’s create a Pod that consumes the volume:

_ch5/pv-pod.yaml_

```
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - name: http
          containerPort: 80
      volumeMounts:
        - name: pv-storage
          mountPath: "/usr/share/nginx/html"
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: local-pv-claim
```

We refer to the persistent volume claim under **volumes**, not to the underlying volume. Inside the
**containers** field, we are then referencing the volume (**pv-storage**) and mounting it under
**/usr/share/nginx/html** (this is the folder Nginx uses by default).

Save the above YAML in **pv-pod.yaml** file and create it by running **kubectl apply -f pv-pod.yaml**.

To check if the volume mount worked correctly, we are going to use the **port-forward** command to forward port **5000** on the local machine to port **80** on the container. Open a separate terminal window and run:

```
$ kubectl port-forward po/pv-pod 5000:80
Forwarding from 127.0.0.1:5000 -> 80
Forwarding from [::1]:5000 -> 80
```

You can either open a browser and navigate to http://localhost:5000 or run **curl**
http://localhost:5000 from another terminal window. You should get back the contents of the
**index.html** file we created earlier:

```
$ curl http://localhost:5000
Hello from Storage!
```

You can clean up everything by deleting the Pod, PVC, and then the PV:

```
$ kubectl delete po pv-pod
$ kubectl delete pv local-pv-volume
$ kubectl delete pvc local-pv-claim
```

Finally, you can also delete the contents of the **/mnt/data** folder inside the host VM.


### **Using Storage Class for dynamic provisioning**

In the previous examples, we manually created a PersistentVolume first (using **manual** storage class),
before we could claim the storage and then use it.

Both Minikube and Docker Desktop have a default storage class set - in both cases, they use the
**hostPath** provisioner.

Let’s try and create a persistent volume claim, but this time we won’t specify the storage class field
at all:

_ch5/default-pvc.yaml_

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: default-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Save the above YAML in **default-pvc.yaml** and run **kubectl apply -f default-pvc.yaml** to create the
persistent volume claim that uses the default storage class.

If you list the PVC, you will notice that it was bound to a volume called **pvc-bf222497-024e-4ff1-
a29f-1f4d3a441607** using **hostpath** storage class (this is the default storage class name if using Docker
Desktop).

```
$ kubectl get pvc
NAME              STATUS    VOLUME                                    CAPACITY
ACCESS MODES STORAGECLASS AGE
default-pv-claim  Bound     pvc-d1cb8b49-bb9b-499b-baca-3631b40a44bf  1Gi       RWO
hostpath                  3s
```

You can also list the PV to see the persistent volume that was created dynamically by the storage
class:

```
$ kubectl get pv
NAME                                      CAPACITY      ACCESS MODES      RECLAIM POLICY
STATUS       CLAIM            STORAGECLASS        REASON        AGE
pvc-d1cb8b49-bb9b-499b-baca-3631b40a44bf  1Gi           RWO               Delete
Bound        default/default-pv-claim hostpath                  90s
```

On Docker Desktop the volume gets created under **/var/lib/k8s-pvs/[pvc-name]/[pv-name]** folder on
the host VM, where the PVC name in our case is **default-pv-claim** and the PV name is that randomly
generate volume name (**pvc-d1cb8b49-bb9b-499b-baca-3631b40a44bf**).

So, let’s get a shell inside the VM and create an index.html file, just like we did before:

```
# Get a shell inside host VM
docker run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh

# Create index.html file
echo "Hello Dynamic Volume!" >> /var/lib/k8s-pvs/default-pv-claim/pvc-d1cb8b49-bb9b499b-baca-3631b40a44bf/index.html

# exit the VM
exit
```

Finally, let’s create the Pod - the resource looks exactly the same as before, the only difference is the
PVC name:

_ch5/default-pv-pod.yaml_

```
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - name: http
          containerPort: 80
      volumeMounts:
        - name: pv-storage
          mountPath: "/usr/share/nginx/html"
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: default-pv-claim
```

Save the above YAML in **default-pv-pod.yaml** and run **kubectl apply -f default-pv-pod.yaml** to
create the Pod.

When the Pod starts running, use **port-forward** to forward local port **5000** to the port **80** on the
container:

```
$ kubectl port-forward po/pv-pod 5000:80
Forwarding from 127.0.0.1:5000 -> 80
Forwarding from [::1]:5000 -> 80
```

Just like before, you can open http://localhost:5000 in the browser or run the curl command to get
back the response, contents of the **index.html** file:

```
$ curl http://localhost:5000
Hello Dynamic Volume!
```

If you delete the Pod and then the PVC, you will notice that the Persistent Volume gets deleted
automatically. The automatic deletion is due to the reclaim policy on the default storage class. The
value is set to **Delete**, and Kubernetes controller automatically deletes the volume once the claim is
gone.


## Running stateful workloads with StatefulSets

Using PersistentVolumes and PersistentVolumeClaims, you could run multiple Pod replicas using a
ReplicaSet. These replicas have different names and IP addresses, but other than that, they are the
same.

Let’s consider the following snippet from one of the previous examples of a Pod and a PVC:

```
...
spec:
  containers:
    ...
    volumeMounts:
      - name: pv-storage
        mountPath: "/usr/share/nginx/html"
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: local-pv-claim
....
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
The Volume and the persistent volume claim reference are both defined in the Pod spec, the
template ReplicaSet uses to create the new Pods. If you have a single Pod, it will reference a single
PVC and a single PV.


![Figure 30. Single Pod with a PVC and PV](https://i.gyazo.com/18a28b9cde20b76d5da08fdd2f69edc2.png)
_Figure 30. Single Pod with a PVC and PV_

When you scale the Pods, each newly created Pod gets a different name and an IP address. Since
you include PV and PVC in the Pod template, all replicas use the same PVC and the same PV as
shown in the figure below.

![Figure 31. Multiple Pods with a Single PVC and PV](https://i.gyazo.com/69455f3e056e362387422cdb6cd9b116.png)
_Figure 31. Multiple Pods with a Single PVC and PV_

Because you defined the reference to the PVC in the Pod template, you can’t make each replica use
its persistent volume claim. For that reason, you can’t use a single ReplicaSet to run a distributed
data store where each Pod has its storage.

There are a couple of workarounds, such as creating the Pods manually and have them use
separate PVC in that way, however, you don’t want to manually manage the Pods' lifecycle
(remember that any manually created Pod will not get automatically restarted when or if it
crashes). Another workaround would be to use multiple ReplicaSets - one for each Pod. This
solution could work; however, it is painful to maintain.

In addition to have a separate storage per Pod, you also need to ensure the Pods have a stable and
persistent identity. Stable identity means if the Pod is restarted, it comes back with the same
identifier (the same name and the same IP address). The reason for stable identity is that you often
need to address a specific replicate, especially when running a storage system. That is complicated
to do if you don’t have stable identifiers.

A resource called a __StatefulSet__ can help. You can use StatefulSets for applications that require a
stable name and state. The main difference between a StatefulSet and a ReplicaSet is that when
Pods created by a ReplicaSet are rescheduled, they get a new name and a new IP address. On the
other hand, the StatefulSet ensures that Pod keeps the same name and the same IP address even
when it gets rescheduled.

StatefulSets also allow you to run multiple Pod replicas. These replicas are not the same - the can
have their own set of volume claims or, more specifically volume claim templates. The replicas
don’t have random names. Instead, each Pod gets assigned an index.

We have mentioned earlier that in some scenarios, you want to be able to address a specific replica
from the StatefulSet. You can do that by creating a headless Kubernetes service.

In the next example, we will show how to run MongoDB using a StatefulSet. As the first step, we
will create a headless service called **mongodb**. We need to create this first because we will reference
the service name in the StatefulSet.

_ch5/mongodb-svc.yaml_

```
apiVersion: v1
kind: Service
metadata:
  name: mongodb
  labels:
    app: mongodb
spec:
  clusterIP: None
  selector:
    app: mongodb
  ports:
    - port: 27017
      targetPort: 27017
```

Save the above YAML in **mongodb-svc.yaml** and create the Service by running **kubectl apply -f
mongodb-svc.yaml**. If you list the Services, you will notice the **mongodb** Service does not have an IP
address set:

```
$ kubectl get svc
NAME        TYPE      CLUSTER-IP    EXTERNAL-IP     PORT(S)     AGE
kubernetes  ClusterIP 10.96.0.1     <none>          443/TCP     47h
mongodb     ClusterIP None          <none>          27017/TCP   1s
```

Next, we will create the following StatefulSet:

_ch5/mongodb-statefulset.yaml_

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: mongodb
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
        selector: mongodb
    spec:
      containers:
        - name: mongodb
          image: mongo:4.0.17
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: pvc
              mountPath: /data/db
  volumeClaimTemplates:
    - metadata:
        name: pvc
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
```

Looking at the YAML above, you will notice that it looks very similar to the ReplicaSet. The critical
part is the **volumeClaimTemplates** section. That’s the section where we defined the
PersistentVolumeClaim. When the StatefulSet needs to create a new Pod replica, it uses that
template also to create a PersistentVolume and a PersistentVolumeClaim resource for each Pod, as
shown in the figure below.


![Figure 32. Pods Created by StatefulSet](https://i.gyazo.com/fa80d350f0885da624f84f0d6719e4f1.png)
_Figure 32. Pods Created by StatefulSet_

Save the above YAML in **mongodb-statefulset.yaml** and create the StatefulSet by running **kubectl apply -f mongodb-statefulset.yaml**.

If you list the Pods, you will notice you created a single Pod named **mongodb-0**:

```
$ kubectl get pods
NAME        READY       STATUS      RESTARTS       AGE
mongodb-0   1/1         Running     0              29m
```

Similarly, let’s list the PersistentVolumes and PersistentVolumeClaims:

```
$ kubectl get pv,pvc
NAME                                                        CAPACITY      ACCESS MODES 
RECLAIM POLICY      STATUS    CLAIM       STORAGECLASS REASON AGE
persistentvolume/pvc-275f7731-0f10-4b33-b99a-3cc95d0be30a 1Gi RWO
Delete              Bound default/pvc-mongodb-0 standard 31m

NAME STATUS VOLUME
CAPACITY ACCESS MODES STORAGECLASS AGE
persistentvolumeclaim/pvc-mongodb-0 Bound pvc-275f7731-0f10-4b33-b99a-3cc95d0be30a 1Gi RWO standard 31m
```
Notice the naming of both resources - the PVC uses the name (**pvc**) we provided under the
**volumeClaimTemplates**, and it appends the name of the Pod (**mongodb-0**) to it. Kubernetes prefixes the
PersistentVolume name with the PVC name (**pvc**). The rest of the name is a unique identifier.

Let’s see what happens if we scale the StatefulSet to 3 Pods:

```
$ kubectl scale statefulset mongodb --replicas=3
statefulset.apps/mongodb scaled
```

If you watch the Pods as Kubernetes creates them (**kubectl get pods -w**) you will notice they are
created in order - the replica named **mongodb-1** is created first, and after that replica **mongodb-2** is
created.

A StatefulSet stores the name of the pod in label called **statefulset.kubernetes.io/pod-name**. You can
see these labels if you run **kubectl get po --show-labels**:

```
$ kubectl get po --show-labels
NAME        READY     STATUS    RESTARTS    AGE     LABELS
mongodb-0   1/1       Running   0           44m     statefulset.kubernetes.io/pod-name
=mongodb-0
mongodb-1   1/1       Running   0           2m34s   statefulset.kubernetes.io/pod-name
=mongodb-1
mongodb-2   1/1       Running   0           2m30s   statefulset.kubernetes.io/pod-name
=mongodb-2
```

These Pods have other labels (**selector**, **app**, **controller-revision-hash**). I removed them from the
above output for better readability.

Using this label, you could create a service to target a specific replica. Remember when we created
a headless service? Let’s see how we can use it to access a particular replica.

We have 3 Pods running, and they are named **mongodb-0**, **mongodb-1**, and **mongodb-2**. We also have a
headless service called **mongodb**. To access each instance, we use the following format
**[podName].[serviceName]**. For example, to access **mongodb-1** we can use **mongodb-1.mongodb** or ***mongodb-1.mongodb.default.svc.cluster.local**, where **default** is the namespace where the Pods (and Service)
reside, and cluster.local is the local cluster domain.

Let’s try accessing these instances. We will run a **mongo** container inside the cluster and use that
container to access the **mongodb-0**,**mongodb-1**, and **mongodb-2** instances.
Run the following command to get create a Pod called **mongo-shell** and run a **/bin/bash** inside it:

```
$ kubectl run -it mongo-shell --image=mongo:4.0.17 --rm -- /bin/bash
If you don\'t see a command prompt, try pressing enter.
root@mongo-shell: # /
```

The container has the **mongo** shell installed, and we can use this binary to connect to the MongoDB
instances. Let’s try and connect to the **mongodb-0** instance.

To do that, we will use the **mongodb-0.mongodb** name - remember that this only works because we are
running the **mongo-shell** container inside of the cluster.

```
root@mongo-shell:/# mongo mongodb-0.mongodb
MongoDB shell version v4.0.17
connecting to: mongodb://mongodb-0.mongodb:27017/test?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("7972f9c4-f04e-42fe-aa15-e63adfaea2eb") }
MongoDB server version: 4.0.17
Welcome to the MongoDB shell.
....
...
>
```

The shell will automatically connect us to the collection called test. Let’s insert a dummy entry into
that collection so that we can prove the MongoDB instances are each storing the data in a different
volume.

Run the command below to insert a single entry with the name field set to **first MongoDB instance**:

```
> db.test.insert({ name: "first MongoDB instance" })
WriteResult({ "nInserted" : 1 })
```

You can also run **db.test.find()** to list all documents from the **test** collection (there will only be one
in there):
```
> db.test.find()
{ "_id" : ObjectId("5f540f288a4810beb9d9237a"), "name" : "first MongoDB instance" }
```
Let’s connect to the second instance now. You can type exit to **exit** the first instance, and get back to
the **mongo-shell** container. Just like before, we will use the **mongo** binary to connect to a different
instance:

```
$ root@mongo-shell:/# mongo mongodb-1.mongodb
```

Once connected, you can look at the contents of the **test** collection and, as expected, there will be 0
documents in the collection:

```
> db.test.find()
>
```

We can create a different document here:

```
> db.test.insert({ name: "second MongoDB instance" })
WriteResult({ "nInserted" : 1 })
```

Let’s see if the data is persistent when we delete the **mongodb-1** Pod. Open a separate terminal
window and delete the **mongodb-1** Pod:

```
$ kubectl delete po mongodb-1
pod "mongodb-1" deleted
```

As soon as the Pod is deleted, StatefulSet will re-create it again - it will use the same name, PVC and
PV. If you run the **db.test.find()** from the mongo shell in the first terminal window you will notice
that it contains the same data we inserted earlier right after it reconnects to the Pod:

```
> db.test.find()
2020-09-05T22:29:43.690+0000 I NETWORK [js] trying reconnect to mongodb-1.mongodb:27017 failed
2020-09-05T22:29:43.693+0000 I NETWORK [js] reconnect mongodb-1.mongodb:27017 ok
{ "_id" : ObjectId("5f5411034901f8a94bfdcc7c"), "name" : "second MongoDB instance" }
```

# Organizing Containers

## Init containers

Init containers allow you to separate your application from the initialization logic and provide a
way to run the initialization tasks such as setting up permissions, database schemas, or seeding
data for the main application, etc. The init containers may also include any tools or binaries that
you don’t want to have in your primary container image due to security reasons.

The init containers are executed in a sequence before your primary or application containers start.
On the other hand, any application containers have a non-deterministic startup order, so you can’t
use them for the initialization type of work.

The figure below shows the execution flow of the init containers and the application containers.

![Figure 33. Init Containers](https://i.gyazo.com/a01e12b7cc687837025cc667f184d9e9.png)

_Figure 33. Init Containers_

The application containers will wait for the init containers to complete successfully before starting.
If the init containers fail, the Pod is restarted (assuming we didn’t set the restart policy to **RestartNever**), which causes the init containers to run again. When designing your init containers,
make sure they are idempotent, to run multiple times without issues. For example, if you’re seeding
the database, check if it already contains the records before re-inserting them again.

Since init containers are part of the same Pod, they share the volumes, network, security settings,
and resource limits, just like any other container in the Pod.

Let’s look at an example where we use an init container to clone a GitHub repository to a shared
volume between all containers. The Github repo contains a single **index.html**. Once the repo is
cloned and the init container has executed, the primary container running the Nginx server can use
**index.html** from the shared volume and serve it.

You define the init containers under the **spec** using the **initContainers** field, while you define the
application containers under the **containers** field. We define an **emptyDir** volume and mount it into
both the init and application container. When the init container starts, it will run the **git clone**
command and clone the repository into the **/usr/share/nginx/html** folder. This folder is the default
folder Nginx serves the HTML pages from, so when the application container starts, we will be able
to access the HTML page we cloned.


_ch6/init-container.yaml_

```
apiVersion: v1
kind: Pod
metadata:
  name: website
spec:
  initContainers:
    - name: clone-repo
      image: alpine/git
      command:
        - git
        - clone
        - --progress
        - https://github.com/peterj/simple-http-page.git
        - /usr/share/nginx/html
      volumeMounts:
        - name: web
          mountPath: "/usr/share/nginx/html"
  containers:
    - name: nginx
      image: nginx
      ports:
        - name: http
          containerPort: 80
      volumeMounts:
        - name: web
          mountPath: "/usr/share/nginx/html"
  volumes:
    - name: web
      emptyDir: {}
```

Save the above YAML to **init-container.yaml** and create the Pod using **kubectl apply -f init-container.yaml**.

If you run **kubectl get pods** right after the above command, you should see the status of the init
container:

```
$ kubectl get po
NAME     READY     STATUS    RESTARTS     AGE
website  0/1       Init:0/1  0            1s
```

The number **0/1** indicates a total of 1 init containers, and 0 containers have been completed so far.
In case the init container fails, the status changes to **Init:Error** or **Init:CrashLoopBackOff** if the
container fails repeatedly.

You can also look at the events using the **describe** command to see what happened:

```
Normal    Scheduled   19s   default-scheduler   Successfully assigned   default/website to
minikube
Normal    Pulling     18s   kubelet, minikube   Pulling image "alpine/git"
Normal    Pulled      17s   kubelet, minikube   Successfully pulled image "alpine/git"
Normal    Created     17s   kubelet, minikube   Created container clone-repo
Normal    Started     16s   kubelet, minikube   Started container clone-repo
Normal    Pulling     15s   kubelet, minikube   Pulling image "nginx"
Normal    Pulled      13s   kubelet, minikube   Successfully pulled image "nginx"
Normal    Created     13s   kubelet, minikube   Created container nginx
Normal    Started     13s   kubelet, minikube   Started container nginx
```

You will notice as soon as Kubernetes schedules the Pod, the first Docker image is pulled (
**alpine/git**), and the init container (**clone-repo**) is created and started. Once that’s completed (the
container cloned the repo) the main application container (**nginx**) starts.

Additionally, you can also use the **logs** command to get the logs from the init container by
specifying the container name using the **-c** flag:

```
$ kubectl logs website -c clone-repo
Cloning into '/usr/share/nginx/html'...
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (6/6), done.
```

Finally, to actually see the static HTML page can use **port-forward** to forward the local port to the
port **80** on the container:

```
$ kubectl port-forward pod/website 8000:80
Forwarding from 127.0.0.1:8000 -> 80
Forwarding from [::1]:8000 -> 80
```

You can now open your browser at http://localhost:8000 to open the static page as shown in figure
below.

![Figure 34. Static HTML from Github repo](https://i.gyazo.com/1e9d1078219d9000edddcc54e5a00b8a.png)

_Figure 34. Static HTML from Github repo_

Lastly, delete the Pod by running **kubectl delete po website**.

## Sidecar container pattern

The sidecar container aims to add or augment an existing container’s functionality without
changing the container. In comparison to the init container, we discussed previously, the sidecar
container starts and runs simultaneously as your application container. The sidecar is just a second
container you have in your container list, and the startup order is not guaranteed.

Probably one of the most popular implementations of the sidecar container is in Istio service mesh.
The sidecar container (an Envoy proxy) is running next to the application container and
intercepting inbound and outbound requests. In this scenario, the sidecar adds the functionality to
the existing container and allows the operator to do traffic routing, failure injection, and other
features.

![Figure 35. Sidecar Pattern](https://i.gyazo.com/efcee7b243db2ee3a1012d9dd2d6fe41.png)

_Figure 35. Sidecar Pattern_

A simpler idea might be having a sidecar container (log-collector) that collects and stores
application container’s logs. That way, as an application developer, you don’t need to worry about
collecting and storing logs. You only need to write logs to a location (a volume, shared between the
containers) where the sidecar container can collect them and send them to further processing or
archive them.

If we continue with the example we used for the init container; we could create a sidecar container
that periodically updates runs git pull and updates the repository. For this to work, we will keep
the init container to do the initial clone, and a sidecar container that will periodically (every 60
seconds for example) check and pull the repository changes.

To try this out, make sure you fork the original repository (https://github.com/peterj/simple-http-page.git) and use your fork in the YAML below.

_ch6/sidecar-container.yaml_
```
apiVersion: v1
kind: Pod
metadata:
  name: website
spec:
  initContainers:
    - name: clone-repo
      image: alpine/git
      command:
        - git
        - clone
        - --progress
        - https://github.com/peterj/simple-http-page.git
        - /usr/share/nginx/html
      volumeMounts:
        - name: web
          mountPath: "/usr/share/nginx/html"
  containers:
    - name: nginx
      image: nginx
      ports:
        - name: http
          containerPort: 80
      volumeMounts:
        - name: web
          mountPath: "/usr/share/nginx/html"
    - name: refresh
      image: alpine/git
      command:
        - sh
        - -c
        - watch -n 60 git pull
      workingDir: /usr/share/nginx/html
      volumeMounts:
        - name: web
          mountPath: "/usr/share/nginx/html"
  volumes:
    - name: web
      emptyDir: {}
```
We added a container called **refresh** to the YAML above. It uses the **alpine/git** image, the same
image as the init container, and runs the **watch -n 60 git pull** command.

__NOTE__:

>The **watch** command periodically executes a command. In our case, it executes **git pull** command and updates the local repository every 60 seconds.

Another field we haven’t mentioned before is **workingDir**. This field will set the working directory
for the container. We are setting it to **/usr/share/nginx/html** as that’s where we originally cloned the repo to using the init container.

Save the above YAML to **sidecar-container.yaml** and create the Pod using **kubectl apply -f sidecar-container.yaml**.

If you run **kubectl get pods** once the init container has executed, you will notice the **READY** column
now shows **2/2**. These numbers tell you right away that this Pod has a total of two containers, and
both of them are ready:

```
$ kubectl get po
NAME      READY    STATUS    RESTARTS     AGE
website   2/2      Running   0            3m39s
```

If you set up the port forward to the Pod using **kubectl port-forward pod/website 8000:80** command
and open the browser to http://localhost:8000, you will see the same webpage as before.

We can open a separate terminal window and watch the logs from the refresh container inside the
**website** Pod:

```
$ kubectl logs website -c refresh -f

Every 60.0s: git pull
Already up to date.
```

The **watch** command is running, and the response from the last **git pull** command was Already up
to date. Let’s make a change to the index.html in the repository you forked.

I added < div> element and here’s how the updated **index.html** file looks like:

_ch6/index.html_
```
<html>
  <head>
    <title>Hello from Simple-http-page</title>
  </head>
  <body>
    <h1>Welcome to simple-http-page</h1>
    <div>Hello!</div>
  </body>
</html>
```
Next, you need to stage this and commit it to the **master** branch. The easiest way to do that is from
the Github’s webpage. Open the **index.html** on Github (I am opening https://github.com/peterj/
simple-http-page/blob/master/index.html, but you should replace my username **peterj** with your
username or the organization you forked the repo to) and click the pencil icon to edit the file (see
the figure below).

![Figure 36. Editing index.html on Github](https://i.gyazo.com/9ab9de042cae56ced7964bc7a310c960.png)

_Figure 36. Editing index.html on Github_

Make the change to the **index.html** file and click the __Commit changes__ button to commit them to the
branch. Next, watch the output from the refresh container, and you should see the output like this:

```
Every 60.0s: git pull

From https://github.com/peterj/simple-http-page
  f804d4c..ad75286 master -> origin/master
Updating f804d4c..ad75286
Fast-forward
 index.html | 1 +
 1 file changed, 1 insertion(+)
```

The above output indicates changes to the repository. Git pulls the updated file to the shared
volume. Finally, refresh your browser where you have http://localhost:8000 opened, and you will
notice the changes on the page:

![Figure 37. Updated index.html page](https://i.gyazo.com/bb59db5b601fefc4306e1a55f74e330a.png)

_Figure 37. Updated index.html page_

You can make more changes, and each time, the page will get updated within 60 seconds. You can
delete the Pod by running **kubectl delete po website**.

## Ambassador container pattern

The ambassador container pattern aims to hide the primary container’s complexity and provide a
unified interface through which the primary container can access services outside of the Pod.


![Figure 38. Ambassador Pattern](https://i.gyazo.com/c59c4c12ff99528b06b1794c0894eb66.png)

_Figure 38. Ambassador Pattern_

These outside or external services might present different interfaces and have other APIs. Instead
of writing the code inside the main container that can deal with these external services' multiple
interfaces and APIs, you implement it in the ambassador container. The ambassador container
knows how to talk to and interpret responses from different endpoints and pass them to the main
container. The main container only needs to know how to talk to the ambassador container. You
can then re-use the ambassador container with any other container that needs to talk to these
services while maintaining the same internal interface.

Another example would be where your main containers need to make calls to a protected API. You
could design your ambassador container to handle the authentication with the protected API. Your
main container will make calls to the ambassador container. The ambassador will attach any
needed authentication information to the request and make an authenticated request to the
external service.

![Figure 39. Calls Through Ambassador](https://i.gyazo.com/b4e7d2ca7b5a2184b5d8336a5e4cf364.png)

_Figure 39. Calls Through Ambassador_

To demonstrate how the ambassador pattern works, we will use The Movie DB
(TMBD)[https://www.themoviedb.org/]. Head over to the website and register (it’s free) to get an API
key.

The Movie DB website offers a REST API where you can get information about the movies. We have
implemented an ambassador container that listens on path **/movies**, and whenever it receives a
request, it will make an authenticated request to the API of The Movie DB.

Here’s the snippet from the code of the ambassador container:

```
func TheMovieDBServer(w http.ResponseWriter, r *http.Request) {
  apiKey := os.Getenv("API_KEY")
  resp, err := http.Get(fmt.Sprintf
("https://api.themoviedb.org/3/discover/movie?api_key=%s", apiKey))
  // ...
  // Return the response
}
```
We will read the API_KEY environment variable and then make a GET request to the URL. Note if you
try to request to URL without the API key, you’ll get the following error:

```
$ curl https://api.themoviedb.org/3/discover/movie
{"status_code":7,"status_message":"Invalid API key: You must be granted a valid key."
,"success":false}
```

I have pushed the ambassador’s Docker image to **startkubernetes/ambassador:0.1.0**.

Just like with the sidecar container, the ambassador container is just another container that’s
running in the Pod. We will test the ambassador container by calling curl from the main container.

Here’s how the YAML file looks like:

_ch6/ambassador-container.yaml_
```
apiVersion: v1
kind: Pod
metadata:
  name: themoviedb
spec:
  containers:
    - name: main
      image: radial/busyboxplus:curl
      args:
        - sleep
        - "600"
    - name: ambassador
      image: startkubernetes/ambassador:0.1.0
      env:
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: themoviedb
              key: apikey
      ports:
        - name: http
          containerPort: 8080
```
Before we can create the Pod, we need to create a Secret with the API key. Let’s do that first:

```
$ kubectl create secret generic themoviedb --from-literal=apikey=<INSERT YOUR API KEY
HERE>
secret/themoviedb created
```

You can now store the Pod YAML in **ambassador-container.yaml** file and create it with **kubectl apply -f ambassador-container.yaml**.

When Kubernetes creates the Pod (you can use **kubectl get po** to see the status), you can use the
**exec** command to run the **curl** command inside the **main** container:

```
$ kubectl exec -it themoviedb -c main -- curl localhost:8080/movies

{"page":1,"total_results":10000,"total_pages":500,"results":[{"popularity":2068.491,"v
ote_count":
...
```

Since containers within the same Pod share the network, we can make a request against
**localhost:8080**, which corresponds to the port on the ambassador container.

You could imagine running an application or a web server in the main container, and instead of
making requests to the **api.themoviedb.org** directly, you are making requests to the ambassador
container.

Similarly, if you had any other service that needed access to the **api.themoviedb.org** you could add
the ambassador container to the Pod and solve access like that.

## Adapter container pattern

For the ambassador container pattern, we said that it hides outside services' complexity and
provides a unified interface to the main container. The adapter container pattern does the opposite.
It provides a unified interface to the external services.

![Figure 40. Adapter Pattern](https://i.gyazo.com/f7e07a3604e09232ede74094ae091053.png)

_Figure 40. Adapter Pattern_

Using the adapter pattern, you use common interfaces across multiple containers. An excellent
example of this pattern are adapters that ensure all containers have the same monitoring interface.
For example, adapter for exposing the application or container metrics on **/metrics** and port **9090**.

Your application might write the logs and metrics to a shared volume, and the adapter reads the
data and serves it on a common endpoint and port. Using this approach, you can add the adapter
container to each Pod to expose the metrics.

Another use of this pattern is for logging. The idea is similar as the metrics - your applications can
use an internal logging format. Simultaneously, the adapter takes that format, cleans it up, adds
additional information, and then serves it to the centralized log aggregator.

## Lifecycle Hooks

The concept of hooks is well-known in the tech world. Events usually trigger hooks, and they allow
developers to react to those events and run some custom code. Let’s take a simple user interface
with a button and a text box. There might be multiple events that developers might be interested in
handling (i.e., running some code whenever the event happens). One of these events could be the
**onClick** event. You could write an onClick handler that gets called whenever a user clicks a button.

Another popular example of hooks is webhooks. For example, your e-commerce website can define
webhooks that can send you a JSON payload with the purchase information to a URL you specified
whenever a sale occurs. You write a handler (in this case, it could be a serverless function) and set
your serverless function as a handler for an event. This allows you to loosely couple the
functionality and handle events that happen on a different system.

![Figure 41. Simple Webhook](https://i.gyazo.com/092d8e6dde60e95c34e423c3ea4e0846.jpg)

_Figure 41. Simple Webhook_

Similarly, Kubernetes provides so-called __container hooks__. The container hooks allow you to react
to container lifecycle events. There are two hooks you can use, the **PostStart** and **PreStop**.

Kubernetes executes the **PostStart** hook as soon as the container is created. However, there’s no
guarantee that the hook runs __before__ the containers' ENTRYPOINT command is called (they fire
asynchronously). Note that if the hook handler hangs, it will prevent the container from reaching a
running state.

Kubernetes calls the **PreStop** hook before a container gets terminated. For the container to stop, the
hook needs to complete executing. If the code in the handler hangs, your Pod will remain in the
Terminating state until it gets killed.

If either of the hook handlers fails, the container will get killed. If you decide on using these hooks,
try to make your code as lightweight as possible, so your containers can start/stop quickly.

As for the handlers, you can use a command that gets executed inside the container (e.g. **myscript.sh**) or send an HTTP request to a specific endpoint on the container (e.g. /**shutdown**).

The most common scenarios you’d use the hooks for are performing some cleanup or saving the
state before the container is terminated (PreStop) or configure application startup once the
container starts (PostStart).

We’ve talked about init containers, and there are differences between the two:

- Init containers have their image while lifecycle hooks are executed inside the parent containers
- Init containers are defined at the Pod level, while lifecycle hooks are defined per each container
- Init containers are guaranteed to execute before the application containers start, while the
PreStart hook might not execute before the ENTRYPOINT is called

![Figure 42. Lifecycle Hooks](https://i.gyazo.com/c18ec3008819e04aa54b96b76c1c875d.png)

_Figure 42. Lifecycle Hooks_

Let’s look at an example to see how these lifecycle handlers work.