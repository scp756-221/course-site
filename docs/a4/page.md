# Assignment 4&mdash;Kubernetes & Container Orchestration

## Introduction

This assignment will revisit the music service you used in Assignments&nbsp;1 through 3. We will expand our viewpoint from a single service, the music service, to a larger microserviced-based application:

![AWS Image](System-diagram.png?raw=true)

 The application incorporates four services, three of which are implemented by us:
<table>
<caption>The microservice design</caption>
<thead>
<th>Service</th><th>Short name</th><th>Purpose</th><th>Location</th>
</thead>
<tbody>
<tr>
<td>Users</td><td>S1</td><td>List of users</td><td>Your Kubernetes cluster on AWS</td>
</tr>
<tr>
<td>Music</td><td>S2</td><td>Lists of songs and their artist</td><td>Your Kubernetes cluster on AWS</td>
</tr>
<tr>
<td>Database</td><td>DB</td><td>Interface to key-value store</td><td>Your Kubernetes cluster on AWS</td>
</tr>
<tr>
<td>DynamoDB</td><td>(None)</td><td>Key-value store</td><td>Service managed by Amazon</td>
</tr>
</tbody>
</table>

Once again, you will find and fix the "bug" in the music service, using the tools of Kubernetes to locate its logs and redeploy the fixed version of the service.

### Kubernetes

Manually running a single service on a remote instance (Assignments&nbsp;2 and 3) took a lot of effort. Running three services is probably manageable but not a good use of time. You will instead offload the work to Kubernetes. 

Some definitions of terms:

* **Kubernetes** is an open-source tool that manages the containers on a *cluster*.
* **k8s** is a common abbreviation for "Kubernetes" (because there are 8 letters between the "K" and "s")
* **Cluster** is a group of machines managed by Kubernetes and designed for its exclusive use.
* **EKS** (Elastic Kubernetes Service) is Amazon's managed Kubernetes service. It simplifies the use of Kubernetes by taking care of the installation and operation of Kubernetes on a group of machine.

Managing containers is often called "orchestrating" the containers.  This entails:

* Assigning containers to the machines (called "nodes") on which they will run.
* Starting the containers.
* Restarting them when they crash or fail health checks.
* Establishing the network connections between services, both
  internal (inaccessible outside the cluster) and external
  (accessible outside the cluster).

You will complete this assignment using AWS EKS as your Kubernetes platform.  (We have included _optional_ material you may use  with Microsoft&nbsp;Azure, Google&nbsp;GCP, and [Minikube](https://minikube.sigs.k8s.io/docs/start/). For Azure and GCP, you will need to install their respective command-line interfaces. Minikube is a local Kubernetes learning environment. It is a self-contained environment but requires substantial resources (16GB and more ideally more). More critically, Minikube installation need to be installed into your host OS environment which also entails all related k8s tools to be similarly installed.

**A final word before I launch you into this assignment,** this assignment is involved. From prior cohort and TAs' experiences, it requires careful reading, attention to detail, follow-through and some initiative on your part. I suggest to read through the assignment *at least once* before starting anything. I have included copious amount of links to fill in the background; follow and read them as required. Doing so will help you to orient to how the entire assignment fits together. Hopefully, Assignment 3 has prepared you by previewing some of the techniques and tools. Even though this is an individual assignment (submission and grading is per individual), you can definitely collaborate and discuss with your class mates as you work through it. Do not procrastinate as there is no substitute for time to read and absorb the material here.


## 1. Preparing Your Service

In the first three assignments, we worked exclusively in the `c756-exer/s2/standalone` directory. In this and future assignments, we will work with the full directory structure under `c756-exer`:

| Directory/file | Note |
|-|-|
|s1/ | the User service |
|s2/ | the Music service |
|db/ | the database writer service |
|postman/ | the API definition collection |
|cluster/ | material for working with a Kubernetes cluster. Also location of all files that would pose security risks if made public|
|logs/ | all output files are collected here |
|tools/ | misc tools to prepare repo content (makefiles) for use |
|gatling/ | Gatling load tester---for Assignment 5 onward |
|profiles/ | files defining any personal bash or Git command aliases you use |
|gcloud/ | tools for running the Google cloud command-line |
| --- | --- |
|{cmusic.sh, dmusic.sh, duser.sh} | scripts to drive the API |
| --- | --- |
|k8s.mak | instantiated makefile for operating a Kubernetes cluster |
|api.mak | instantiated makefile to operate the API |
|obs.mak | makefile to manage the observability components (Grafana, Prometheus, Kiali) |
|allclouds.mak | instantiated makefile to examine all cloud providers |
|eks.mak | instantiated makefile for setup of an AWS EKS cluster |
| --- | --- |
|az.mak | (optional) instantiated makefile for setup of an Azure AKS cluster |
|gcp.mak | (optional) instantiated makefile for setup of a GCP GKE cluster |
|mk.mak | (optional) instantiated makefile for setup of a Minikube cluster. Will not work in tools container |

**Note that the assignment is organized as a collection of templates (files with ``-tpl`` suffix) that require instantiation. Do not fill in/use the ``-tpl`` files directly since you run the risk of leaking sensitive information should they be pushed (inadvertently or otherwise) to Github etc. (The ``.gitignore`` are setup to exclude your instantiated files.)**

## 2. Creating and Controlling your Cluster

You create a cluster with a command specific to the vendor (AWS, Azure, or Google for clusters in the cloud or Minikube for clusters on your own machine).
Once a cluster is created, you use operate the cluster using the Kubernetes-API-based tools: ``kubectl``, `k9s`, and `make`.

In the same fashion, ``istioctl`` operates on the service mesh for a cluster, irrespective of how the cluster came to be.

### 2.1 The Kubernetes toolset

You will use `kubeconfig`, `k9s`, and `make` to control your clusters.

#### 2.1.1 `kubectl` and kubeconfig

``kubectl`` operates from a so-called kubeconfig file which contains the definitions of clusters and contexts. There is no file with such a name. Rather, the file is ``~/.kube/config`` which holds the clusters and contexts set up.

Examine your kubeconfig file now.

One obvious but subtle point about kubeconfig is that its content (each entry being a context comprising a cluster and a set of credential) exists independently of the resources that it refers to. The makefiles for the course's set of assignments tries to keep the two in sync: when a new cluster is created, a corresponding entry is added to the kubeconfig. And similarly when you delete a cluster (its kubeconfig entry is removed). But it is possible for the two to diverge. For example, if you create a cluster via the makefile/command-line and use the cloud provider's web portal to delete this same cluster.

There is an included `allclouds.mak` to fetch the status across all your environments:

```
/home/k8s# make -f allclouds.mak
kubectl config get-contexts
CURRENT   NAME     CLUSTER                      AUTHINFO                          NAMESPACE
*         aws756   aws756.us-west-2.eksctl.io   AWSID@aws756.us-west-2.eksctl.io  c756ns

AWS (eks.mak):
eksctl get cluster --region us-west-2 -v 0
NAME	REGION		EKSCTL CREATED
aws756	us-west-2	True

Azure (az.mak):
az aks list -o table || true


GCP (gcp.mak):
Command 'gcloud' not found.  Skipping this step.

DynamoDB tables, read units, and write units
Music-GHID         5       5
User-GHID          5       5

Background Gatling jobs running

Run "kill -9 PROCESS-ID-LIST" to terminate the Gatling jobs
```

where every occurrence of `AWSID` will be your AWS userid and every occurrernce of `GHID` will be your GitHub userid.

#### 2.1.2 `k9s`

The tool `k9s` is a complement to `kubectl`.  They both offer equivalent functionality with different interfaces:  `kubetcl` runs a single command and returns text output whereas `k9s` provides a [text-based user interface (TUI)](https://en.wikipedia.org/wiki/Text-based_user_interface). `k9s` takes a bit of time to learn but supports much faster interaction. We will not go into `k9s` in detail in this assignment but it is sufficiently intuitive that you can pick it up easily. `k9s` is much more convenient and will save a lot of time so don't avoid it.

#### 2.1.3 `make` & makefile

The assignments for 756 are organized around a set of makefiles to simplify and to introduce the various tools. Recall [last week's reading](https://scp756-221.github.io/course-site//#/r3) included an article on it. If you wish to dig further, refer [here](https://www.gnu.org/software/make/manual/html_node/Introduction.html).

There are four files that you will interact with frequently and three others that you might want to explore on your own:

| File | Purpose |
| -----| ----------------- |
| k8s.mak | instantiated makefile for operating k8s |
| api.mak | instantiated makefile to operate the API |
| allclouds.mak | instantiated makefile to examine all cloud providers |
| eks.mak | instantiated makefile for setup of an AWS EKS cluster |
| --- | --- |
| az.mak | (Optional) instantiated makefile for setup of an Azure AKS cluster |
| gcp.mak | (Optional) instantiated makefile for setup of a GCP GKE cluster |
| mk.mak | (Optional) instantiated makefile for setup of a Minikube cluster. Will not work in tools container |


The core idea of a makefile is to automate the execution of commands to reproduce artifacts in a chained fashion. It is typically used to compile program source into executables but it is applicable anytime you have outputs that are dependent on a chain of upstream inputs.

For example, ``eks.mak`` has been setup to start, stop, and check on the status of your Amazon EKS  cluster as follows:

| Command | Purpose |
| -----| ----------------- |
| ``make -f eks.mak start`` | create your EKS cluster |
| ``make -f eks.mak stop`` | delete your EKS cluster |
| ``make -f eks.mak down`` | delete an EKS cluster's nodegroup |
| ``make -f eks.mak up`` | create a nodegroup for an EKS cluster whose group was previously deleted |
| ``make -f eks.mak status`` | check on the status of your EKS cluster |
| ``make -f eks.mak ls`` | list all EKS clusters and their node groups |
| ``make -f eks.mak cd`` | make the EKS cluster your current cluster (when runnning clusters from multiple vendors) |

It may seem dubious value to introduce this extra machinery when a single call to `eksctl` is a direct way to start, stop and check on clusters. But one benefit of using make is to wrap a _series_ of commands and associate it with a pseudo-target (e.g., the `start`, `stop`, or `ls` in the above) for automation. As well, the  parameter definitions (e.g.,  `REGION`/`$(REGION)`, `KVER`/`$(KVER)`) and comments serves to document the tools, options and process for achieving a given objective. In this course, I have adopted a set of consistent pseudo-targets across the cloud vendors to homogenize the creation of clusters in multiple contexts--across the a set of public cloud providers:

| Command | Purpose |
| -----| ----------------- |
| ``make -f mk.mak start`` | create a Minikube cluster |
| ``make -f az.mak start`` | create an AKS cluster |
| ``make -f eks.mak start`` | create an AWS EKS cluster |
| ``make -f gcp.mak start`` | create a GCP GKE cluster |
| ``make -f allclouds.mak`` | a convenient `ls -a` across all three clouds |

### 2.2 Creating a Cluster

**To help with learning k8s, this assignment is built around a set of makefiles that wrap many long and verbose commands into convenient short snippets. Please study the makefiles (`*.mak`) and examine the commands therein to work out what's happening. You will need to understand these commands to effectively use Kubernetes here and in the term project.**

#### 2.2.2 Cloud cluster

Start up an Amazon EKS cluster as follows:

```bash
/home/k8s# make -f eks.mak start
```

This is a slow operation, often taking 10--15 minutes. If you review ``eks.mak``, this is the command that was used (some parameters may vary):

```
eksctl create cluster --name aws756 --version 1.21 --region us-west-2 --nodegroup-name worker-nodes --node-type t3.medium --nodes 2 --nodes-min 2 --nodes-max 2 --managed
```

You can see where parameters have been used to allow for easy tailoring. At the completion of this command (typically 10-15 minutes), you will have a barebone k8s cluster comprising some master nodes and 2 worker nodes (each of which is an EC2 instance). (The nodes are specified by the tail ``--node-type t3.medium --nodes 2 --nodes-min 2 --nodes-max 2``.)

#### 2.2.1 kubeconfig

Upon completion of the creation of your cluster, you can see a summary of your current environment (also known as kubeconfig) by:

```bash
/home/k8s#  kubectl config get-contexts
CURRENT   NAME     CLUSTER                      AUTHINFO                         NAMESPACE
*         aws756   aws756.us-west-2.eksctl.io   AWSID@aws756.us-west-2.eksctl.io   
```

where `AWSID` will be your AWS userid.

Read up on k8s [kubeconfig contexts](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/). This is a vital concept for working and managing k8s clusters.

If you create clusters from other vendors via the corresponding VENDOR.mak, your kubeconfig gains an additional entry indicating that there is an additional cluster available to you. As you work with ``kubectl``, you choose which cluster to operate on. Strictly speaking, each entry returned by `kubectl get-contexts` is a context which comprise a cluster with a specific set of credentials (the `AUTHINFO`). For convenience, there is a 'current context' (indicated by the asterisk * above) which will be the default if you do not specify a context.

The makefiles included with this assignment set the following context names for each cloud vendor:

| Cloud | Context Name |
| -----| ----------------- |
| AWS EKS | aws756 |
| Azure AKS | az756 |
| GCP GKE | gcp756 |

While you're here organizing your context, create a `c756ns` namespace inside each cluster and set each context to use this:

```bash
# for AWS EKS; similar for az/gcp
/home/k8s#  kubectl config use-context aws756
/home/k8s#  kubectl create ns c756ns
/home/k8s#  kubectl config set-context aws756 --namespace=c756ns
```

There is a `cd` target that you can also use:
```bash
# Choose one of eks/az/gcp
/home/k8s#  make -f VENDOR.mak cd
```
to switch between contexts/clusters.

### 2.3 Deleting a Cluster

For the cloud vendors, the following will stop the cluster:

```bash
/home/k8s#  make -f VENDOR.mak stop
```

As with startup, this command will take some time to complete. This command will complete at this time because your cluster was freshly initialized with no resources (beyond the barebone nodes). I

**With your cloud cluster, practice at least stopping it once. Once you are comfortable, restart the cluster. Beware that one stop-and-start cycle is easily 15-30 minutes so manage your time. When you are comfortable, leave your cloud cluster up to continue with the assignment. (But we will stop the cluster at the end of this assignment.) You can use the AWS aliases from the last assignment to observe these machines:

```bash
/home/k8s# eps
eps
i-02fab22446f1e81d3 t3.medium running 52.36.87.109 eks-worker-nodes-e4bf1b12-a552-56b2-e782-be1264163155
i-02e7726afdeeb1b8e t3.medium running 34.223.52.204 fleet-f4e324e3-ff77-c29b-0e3a-2b029f0cebe2
```
**

#### 2.3.1 Managing cloud costs

Two additional pseudo-targets are included to manage costs within your cloud.

In AWS EKS, there are two costs: the NAT gateway (which allow your cluster to communicate with the public Internet) which costs $0.045/h and the t3.small EC2 instances (which provides the compute resources for your containers) which costs $0.0208/h. If you leave a cluster up continuously, you can rack up charges fairly quickly: USD 0.045 + 3 x 0.0208 = USD 0.1074 = CAD 0.134/h.

To reduce cost, you can either delete the cluster entirely (`make -f eks.mak stop`) or delete the nodegroup.

**Deleting the nodegroup does not harm the cluster.** If anything, the design of Kubernetes supports and anticipates the removal of computing resources.

To delete the nodegroup of your cloud cluster:
```bash
/home/k8s#  make -f VENDOR.mak down
```

To recreate the node-group of your cloud cluster:
```bash
/home/k8s#  make -f VENDOR.mak up
```

Either of these command will again take a relatively long time (10+ min) to complete. The time of the initial creation of your cluster is largely composed of the time for creating the node group. A more common reason for creating/deleting your nodegroup is to change the size of machines in your cluster.

This ``down`` command will again complete readily at this time because there are no services running on them.

The ``ls`` command will determine what services are running on your cluster (the nodes). You will need to stop any service that exposes an external IP before you can delete the cluster.

```
/home/k8s#  make -f eks.mak ls
kubectl config get-contexts
CURRENT   NAME     CLUSTER                      AUTHINFO                         NAMESPACE
*         aws756   aws756.us-west-2.eksctl.io   AWSID@aws756.us-west-2.eksctl.io c756ns
eksctl get cluster --region us-west-2 -v 0
NAME	REGION		EKSCTL CREATED
aws756	us-west-2	True
eksctl get nodegroup --cluster aws756 --region us-west-2
2021-12-14 15:48:11 [ℹ]  eksctl version 0.73.0
2021-12-14 15:48:11 [ℹ]  using region us-west-2
CLUSTER	NODEGROUP	STATUS	CREATED			MIN SIZE	MAX SIZE	DESIRED CAPACITY	INSTANCE TYPE IMAGE ID   ASG NAME
aws756	worker-nodes	ACTIVE	2021-12-14T23:32:51Z	2		2		2	              t3.medium     AL2_x86_64 eks-a6bedda5-4e39-6364-d81f-225c5f385b6f
```

### 2.4 Installing the service mesh `istio`

``istio`` is a service mesh that was conceived concurrently with k8s. But for various reasons, it was ultimately pulled out of k8s and developed as an independent project.

``istio`` is installed into each cluster only once but it will only operate within specific namespaces that you choose. A k8s namespace is a cluster-level construct that organizes the resources within your cluster. You can liken it to the folders in a filesystem (though namespaces are only one-level deep).

To use ``istio`` with an application, you create a namespace for your application, label the namespace for ``istio``, and install your application into this namespace.

To install Istio and label the `c756ns` namespace:

```bash
# switch to the EKS context
/home/k8s# kubectl config use-context aws756
/home/k8s# istioctl install -y --set profile=demo --set hub=gcr.io/istio-release
/home/k8s# kubectl label namespace c756ns istio-injection=enabled

# if you wish to remove the label
/home/k8s# kubectl label namespace c756ns istio-injection-
```

#### 2.4.1 Tunneling into your cluster in the cloud

The steps above will trigger istio to set up an AWS Elastic Load Balancer (ELB) to act as an ingress gateway, opening the cluster to requests from outside machines. (The comparable operation on Azure or GCP will do the same.)

Now, use `kubectl get svc --all-namespaces` (or the `lsa` pseudo-target in `k8s.mak`) to verify that you have 3 new services under a new `istio-system` namespace: `istio-egressgateway`, `istio-ingressgateway`, & `istiod`.

**Tip:** Some commands produce output with long lines, which may wrap around if your window is not wide enough.  To clip the lines, pipe the output through the `cut -c -N` command, where `N` is the maximum line width you want. For example, to clip `kubectl` output to a 140-character length:

~~~bash
/home/k8s# kubectl get svc --all-namespaces | cut -c -140
~~~

Finally, you will need to determine how to access your cluster.
You can fetch the required external IP address using `kubectl`:

```bash
/home/k8s# kubectl -n istio-system get service istio-ingressgateway | cut -c -140
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP                                                              PORT(S)      
istio-ingressgateway   LoadBalancer   10.100.255.11   a844a1e4bb85d49c4901affa0b677773-127853909.us-west-2.elb.amazonaws.com   15021:32744/T
```

The `EXTERNAL-IP` is the entry point to your cluster. The example above (for EKS) shows the rather ungainly DNS name (`a844a1e4bb85d49c4901affa0b677773-127853909.us-west-2.elb.amazonaws.com`).

**Whenever there is no way into your cluster, you will have a `<pending>` as your EXTERNAL-IP.**  Check that Istio is running, using the `kubectl get svc --all-namespaces` command described above.

## 3. Deploying your Application

There are several steps required to deploy and run these our music application in Kubernetes.

### 3.1 Building your images

As in Assignment 3, you will need to build the containers and push them to the container registry. This time, you will build three services (S2, the music service you have used so far, along with S1 and DB).

Build your images with (`cri` short for container registry images):

```
/home/k8s# make -f k8s.mak cri
```

Finally, there is one manual step left before the system can come up auto-magically: to switch your container repositories to public access. (This is a simplification for the purpose of this course.)

Refer to GitHub's documentation to [set public access on your container repositories](https://docs.github.com/en/packages/guides/configuring-access-control-and-visibility-for-container-images#configuring-visibility-of-container-images-for-your-personal-account)


### 3.2 Deploying the music service

This final version of the music service (S2) in the application is not standalone; it relies upon several other resources:

* **DB:** The database service, providing persistent storage to the two higher-level services, S1 and S2.
* **DynamoDB:** An Amazon service, called by DB to actually store the values.
* **Gateway:** A link between S1 and S2 and the external world, such as your machine.

To run this, we first need to start the gateway, database, and music services.  We can do that with a single call:

~~~bash
/home/k8s# make -f k8s.mak gw db s2
~~~

In Assignments&nbsp;1--3, you examined the logs of the music service. When running under Docker in Assignment&nbsp;3, you used `docker container logs` to get these logs. Kubernetes offers a similar command:

~~~bash
/home/k8s# kubectl logs --selector app=cmpt756s2 --container cmpt756s2 --tail=-1
~~~

The `--selector` parameter specifies the pod name and the `--container` parameter specifies the container name (both of which are `cmpt756s2`), while `--tail=-1` requests that the entire log be returned, no matter how long.

The `kubectl` command is verbose and inconvenient to use if you wish to return to the log frequently. Even though automating this via the makefile is possible, `k9s` provides a better, faster, interactive way to get logs.

1. In a separate terminal window, start the tools container as before.

2. Start k9s:

   ~~~bash
   /home/k8s# k9s
   ~~~

3. K9s will open up its "pods" display. You should see two pods, one each for the database and music services. Locate the one for the music service (it will begin with `cmpt756s2-v1`), move the cursor down to select it, then press `Return`.
4. The pod for the music service includes three containers.  Two of them were injected by Istio, while one of them is the container for our music code. That one will be named `cmpt756s2`.  Move the cursor to select it, then press `Return`.
5. You will see the most recent lines of the music service log. In fact, the music service will show requests every five seconds or so. These are health checks, requests from Kubernetes to determine if the music service meets minimal standards of functionality.
6. To see the beginning lines of the log, press `1`.  You will return to this when you want to locate the unique code for the "bug".
7. Leave the log view by pressing `Escape`.
8. Leave the S2 pod by pressing `Escape` (again).
9. Scroll to the database pod, whose name starts with `cmpt756db`, and use the same steps as above to view its log.
10. Leave the database log scrolling for now.
11. When you want to leave k9s, press `:` to enter command mode and in the command line enter `q`  followed by `Return`.

To complete setting up the music service, we need to initialize DynamoDB and load it with initial data:

~~~bash
/home/k8s# make -f k8s.mak loader
~~~

This step builds and pushes another image (`cmpt756loader`) to `ghcr.io`. Return to your GitHub Packages tab and set the access for this new image to public as before.

Watch the k9s log to see the lines where the loader calls the database to load the values.  The lines  will look like:

~~~
127.0.0.6 - - [15/Dec/2021 17:56:32] "POST /api/v1/datastore/load HTTP/1.1" 200 -
~~~

where `datastore` is the service and `load` is the operation, in distinction to `health` and `readiness`, the automated checks from Kubernetes.

To test that all these pieces are now working, we need to start the music client. In a separate (third) terminal window, run the client,
where `EXTERNAL-IP` is the long DNS name you found in the section, "Tunneling into your cluster in the cloud", such as
`a844a1e4bb85d49c4901affa0b677773-127853909.us-west-2.elb.amazonaws.com`:

~~~bash
/home/k8s# cd mcli
/home/k8s/mcli# make PORT=80 SERVER=EXTERNAL-IP run-mcli
~~~

This version of the music service will not list all entries in its database (`read` with no argument will return a zero-length list) but we can
read specific songs using their UUID:

~~~bash
mql: read 6ecfafd0-8a35-4af6-a9e2-cbd79b3abeea
1 items returned
6ecfafd0-8a35-4af6-a9e2-cbd79b3abeea  Taylor Swift         The Last Great American Dynasty
~~~

If it works, congratulations! You have just set up a small microservice-based system on
a Kubernetes cluster running in the cloud.

### 3.3 Fixing the "bug" in the music service

For one final time, I ask you to find and fix the music servce "bug".  To demonstrate the bug, issue the `test` command from the music client.  You will get the familiar error code `500` and the stack trace in the S2 logs. (The stack trace will not be visible in the k9s logs because it is black text displayed on a black background.  But if you select the blank space where the text should be, you will see the trace.)

So this version of the music service has our "bug". Here are the familiar steps for fixing it, with the details of performing them in the Kubernetes environment:

1. **Read the logs:**  Get the unique code from the S2 log (using either k9s or kubectl).
2. **Fix the "bug":** Update `s2/v1/app.py` on your local machine (note that this time, the name is simply `app`, without a suffix).
3. **Push the fix:** This is accomplished by the single command:

   ~~~bash
   /home/k8s# make -f k8s.mak rollout-s2
   ~~~

   This command:

   1. Builds a new S2 container image that incorporates the revised code.
   2. Pushes the image to GHCR.
   3. Initiates a Kubernetes *rollout* of the revision.

The Kubernetes rollout algorithm minimizes service disruption. It first sets up the revised version of the service, performs modest health and readiness checks to ensure that the revision works, and only then terminates the original version, routing all traffic to the revision. This process is too tricky and tedious to do by hand but is readily handled by the Kubernetes algorithms.  This is our first example of the benefits of orchestrating containers by software rather than by hand.

### 3.4 Reflection and look ahead

That assignment was a lot of material.  At this point, it may well seem like using Kubernetes is far more work than running containers by hand. Do we really benefit from all this machinery?  Yes, we do:

1. Most of the work in this lesson has been setting up the cluster, installing some dependencies (istio), defining a namespace, and so forth. These setup work is tedious because it was manual; in the real, most of this will be automated via an infrastructure-as-code framework (e.g. Terraform).
2. The Kubernetes control algorithms can be managed by automated tools.  For example, in future assignments, you will set up the entire cluster, including some extra services that we didn't introduce here, by the pair of commands:

   ~~~bash
   # Choose your cloud provider
   /home/k8s# make -f eks.mak start OR make -f az.mak start OR make -f gcp.mak start
   # Now configure it --- same command, regardless of cloud provider
   /home/k8s# make -f k8s.mak provision
   ~~~

  You entered all the commands explicitly in this assignment to gain some familiarity with the underlying mechanisms.  Going forward, you can use the simpler, automated operations.
3. Starting a service was much simpler:  Just send a description of the service to Kubernetes and it selects a machine, pulls the container image, sets up its ports and files, runs the container on the chosen machine, and performs ongoing health checks. More importantly, if anything unexpected... say an instance fails, 
4. Revising a service was as simple as starting it, with the rollout algorithm ensuring minimal disruption as the revision is deployed.

Actual applications will comprise far more services than the three simple ones (Music, Users, and Database) used in this course. The effort to manage large numbers of services by hand would quickly outstrip human capacity. Kubernetes manages the containers and the underlying cluster, allowing developers and system operators to focus on the applications.

## 4. Cleaning up your cluster

Remember to shutdown your cluster after this assignment.
You need to be particularly careful about your cloud-side clusters as they are expensive to keep over longer periods of time (overnight). Remember that you can use `allclouds.mak` to help with this.

## Submission

Your submission this week is divided between entries entered into CourSys directly and as a PDF. Submit these via [Assignment 4](https://coursys.sfu.ca/2022sp-cmpt-756-g1/+a4/).


CourSys entries:

1. URL of the line of code with the fix in your Github repo's copy of `c756-exer/s2/standalone/app-a3.py`. Navigate to your repo inside Github and locate the file/line. Click on the line number and select "Copy permalink". 
![AWS Image](https://github.com/scp756-221/course-site/blob/main/docs/a1/github-permalink.png?raw=true)

2. The URLs of the 4 images that were built/pushed in this assignment:
  a. `cmpt756s1`
  b. `cmpt756s2`
  c. `cmpt756db`
  d. `cmpt756loader`


In your PDF provide the following:

1. Screen-capture of a terminal session with the git commit that correct the problem. You can use `git log` to retrieve the history. 

2. Compare and contrast Kubernetes with Hadoop and Spark. What similarities do you see between them? What differences are there between them? **Limit your answer to 500 words.** (4 marks)

3. Assume that you wish to run a Java/Python/C/C++/R program of your choice on a Kubernetes cluster. Describe in words the steps/materials required to have your program run on a Kubernetes cluster. (You do not need to do this... think through and describe what's required.) **Limit your answer to 500 words.** (3 marks)
