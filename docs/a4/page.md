## Introduction

**Please refer to the [Exercise 4 FAQ](https://docs.google.com/document/d/1xXuSCJCHU_l0moMviQu_DlPgcj2qFJN4z6a7-MJt5dY/edit?usp=sharing) for up-to-date answers on common problems. I will update them continually as new information rolls in.**

This exercise will revisit the music service you used in Assignments&nbsp;1 through 3. We will expand our viewpoint from a single service, the music service, to a complete microservice design incorporating four services:
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

Manually running a single service on a remote instance (Assignments&nbsp;2 and 3) took a lot of effort.  Running three services that in turn send requests to a fourth (DynamoDB) would be prohibitive. You will instead offload much of the management to Kubernetes. 

Some definitions of terms:

* **Kubernetes** is an open-source tool that manages the containers on a *cluster*.
* **Cluster** is a group of machines managed by Kubernetes and designed for its exclusive use.
* **k8s** is a common abbreviation for "Kubernetes" (because there are 8 letters between the "K" and "s")
* **EKS** is Amazon's Kubernetes service. It stands for "Elastic Kubernetes Service".

Managing containers is often called "orchestrating" the containers.  It includes:

* Assigning containers to the machines (called "nodes") on which they will run.
* Starting the containers.
* Restarting them when they crash or fail health checks.
* Establishing the network connections between services, both
  internal (inaccessible outside the cluster) and external
  (accessible outside the cluster).

You will run this exercise using AWS as a cloud provider.  We also provide files that you can optionally use to run clusters on Microsoft&nbsp;Azure and Google&nbsp;GCP. If you wish to run those, you may need to install their respective command-line interfaces.  We also provide a file that you can optionally use to run a cluster on your own computer, using [Minikube](https://minikube.sigs.k8s.io/docs/start/). However, there are technical difficulties accessing Minikube from inside the tools container, so it will require considerable effort to install it and all required tools in your host OS.

**A final word before I launch you into this exercise,** this exercise is involved with many details. From prior cohort and TA's experiences, it requires careful reading, attention to detail, follow-through and some initiative on your part. I suggest to read through the exercise *at least once* before starting anything. I have included a copious amount of links to fill in the background; follow and read them as required. Doing so will help you to orient to how the entire exercise fits together. Hopefully, Exercise 3 has prepared you by previewing some of the techniques and tools. Even though this is an individual exercise (submission and grading is per individual), you can definitely collaborate and discuss with your class mates as you work through it. Do not procrastinate as there is no substitute for time to read and absorb the material here.


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
|gatling/ | Gatling load tester---for Exercise 5 onward |
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

**Note that the exercise is organized as a collection of templates (files with ``-tpl`` suffix) that require instantiation. Do not fill in/use the ``-tpl`` files directly since you run the risk of leaking sensitive information should they be pushed (inadvertently or otherwise) to Github etc. (The ``.gitignore`` are setup to exclude your instantiated files.)**

## 2. Creating and controlling clusters

You create a cluster with a command specific to the vendor (AWS, Azure, or Google for clusters in the cloud or Minikube for clusters on your own machine).
Once the cluster is created, you use common tools to control all the clusters: ``kubectl``, `k9s`, and `make`.

In the same fashion, ``istioctl`` operates on the service mesh for a cluster, irrespective of how the cluster came to be.

### 2.1 Commands for controlling a cluster

You will use `kubeconfig`, `k9s`, and `make` to control your clusters.

#### 2.1.1 kubeconfig

``kubectl`` operates from a so-called kubeconfig file which contains the definitions of clusters and contexts. There is no file with such a name. Rather, the file is ``~/.kube/config`` which holds the clusters and contexts set up.

Examine your kubeconfig file now.

One obvious but subtle point about kubeconfig is that its content (each entry being a context comprising a cluster and a set of credential) exists independently of the resources that it refers to. The makefiles for the course's set of exercises tries to keep the two in sync: when a new cluster is created, a corresponding entry is added to the kubeconfig. And similarly when you delete a cluster (its kubeconfig entry is removed). But it is possible for the two to diverge. For example, if you create a cluster via the makefile/command-line and use the cloud provider's web portal to delete this same cluster.

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

#### 2.1.2 k9s

The tool `k9s` is an alternative to `kubectl`.  They offer equivalent functionality with different interfaces:  `kubetcl` runs a single command and returns text output whereas `k9s` provides a [text-based user interface (TUI)](https://en.wikipedia.org/wiki/Text-based_user_interface). `k9s` takes a bit of time to learn but supports much faster interaction. We will not go into `k9s` in detail in this exercise but encourage you to practice using it, as it can save considerable time.

#### 2.1.3 make & makefile

The exercises for 756 are organized around a set of makefiles to simplify and to introduce the various tools. If you aren't familiar with makefiles or the make tool, refer [here](https://www.gnu.org/software/make/manual/html_node/Introduction.html).

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

It may seem dubious value to introduce this extra machinery when a single call to `eksctl` is a direct way to start, stop and check on clusters. But one benefit of using make is to wrap a _series_ of commands and associate it with a pseudo-target (e.g., the `start`, `stop`, or `ls` in the above) for automation. (The ``ls`` pseudo-target is an example that will display all deployments, pods and services inside your cluster.) More crucially, adopting a set of consistent pseudo-targets across a number of makefiles allows us to homogenize the creation of clusters in multiple contexts--across the a set of public cloud providers:

| Command | Purpose |
| -----| ----------------- |
| ``make -f mk.mak start`` | create a Minikube cluster |
| ``make -f az.mak start`` | create an AKS cluster |
| ``make -f eks.mak start`` | create an AWS EKS cluster |
| ``make -f gcp.mak start`` | create a GCP GKE cluster |
| ``make -f allclouds.mak`` | a convenient `ls -a` across all three clouds |

### 2.2 Start up clusters

**To help with learning k8s, this exercise is built around a set of makefiles that wrap many long and verbose commands into convenient short snippets. Please study the makefiles (`*.mak`) and examine the commands therein to work out what's happening. You will need to understand these commands to effectively use Kubernetes here and in the term project.**

#### 2.2.2 Cloud cluster

Start up an Amazon EKS cluster as follows:

```bash
/home/k8s# make -f eks.mak start
```

This is a slow operation, often taking 10--15 minutes. If you review ``eks.mak``, this is the command that was used (some parameters may vary):

```
eksctl create cluster --name aws756 --version 1.18 --region us-west-2 --nodegroup-name worker-nodes --node-type t3.medium --nodes 2 --nodes-min 2 --nodes-max 2 --managed
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

The makefiles included with this exercise set the following context names for each cloud vendor:

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

From this point on, you can use the shorter:
```bash
# Choose one of eks/az/gcp
/home/k8s#  make -f VENDOR.mak cd
```
to switch between contexts/clusters.

Confirm that your kubeconfig has c756ns as the default namespace for all contexts.

### 2.3 Stop the clusters

For the cloud vendors, the following will stop the cluster:

```bash
/home/k8s#  make -f VENDOR.mak stop
```

In the public cloud, stopping a machine is tantamount to deleting a machine since your hold on the resource ends when you stop using (and paying for) it.

As with startup, this command will take some time to complete. This command will complete at this time because your cluster was freshly initialized with no resources (beyond the barebone nodes). In general, cloud vendors will not allow you to delete a cluster unless all resources within it have been cleaned up appropriately. More on this later.

**With your cloud cluster, practice at least stopping it once. Once you are comfortable, restart the cluster. Beware that one stop-and-start cycle is easily 15-30 minutes so manage your time. When you are comfortable, leave your cloud cluster up to continue with the exercise. (But we will stop the cluster at the end of this exercise.)**

#### 2.3.1 Managing cloud costs

Two additional sub-commands have been added to ``eks.mak`` to manage costs within your cloud.

As our goal is to limit your AWS expenses (<= $100), you will want to pay attention to your spending. There are two costs to watch out for: NAT gateway (which allow your cluster to communicate with the public Internet) are charged at $0.045/h while the t3.small EC2 instances (which provides the compute resources for your containers) are charged at $0.0208/h. If you leave these up continuously, you will raise your charges prematurely. ($100/($0.045+$0.0208)=1519h or about 9 weeks. And there are other costs beyond these 2 pieces.)

To avoid burning precious money, remember to dispose of the compute/node group whenever you do not need them (overnight or just away for an extended period (2h+)).

**Deleting your node group will not harm the cluster as that exists independently of the resources within it. There just would be no resources to serve your application.**

To delete the node-group of your cloud cluster:
```bash
/home/k8s#  make -f VENDOR.mak down
```

To recreate the node-group of your cloud cluster:
```bash
/home/k8s#  make -f VENDOR.mak up
```

Either of these command will take a relatively long time (10+ min) to complete. In fact, the time of the initial creation of your cluster is largely composed of the time for creating the node group.

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

### 2.4 istio

``istio`` is a service mesh that was conceived concurrently with k8s. But for various reasons, it was ultimately pulled out of k8s and developed as an independent project.

``istio`` uses the side-car pattern to manage network traffic for an application within a specified namespace.  Refer [here](https://istio.io/latest/docs/setup/additional-setup/sidecar-injection/) for details.)

``istio`` is installed into each cluster only once but it will only intervene for specific namespaces that you choose within the cluster. A k8s namespace is a cluster-level construct that organizes the resources within your cluster. You can liken it to the way a filesystem have folders though namespaces are only one-level deep. (You can't nest namespaces as you can with file system directories.)

To use ``istio`` with an application, you create a namespace for your application, install the components of your application into this namespace and mark the namespace for ``istio`` to 'inject' itself.

To install Istio and mark the `c756ns` namespace:

```bash
# switch to the EKS context
/home/k8s# kubectl config use-context aws756
/home/k8s# istioctl install -y --set profile=demo --set hub=gcr.io/istio-release
/home/k8s# kubectl label namespace c756ns istio-injection=enabled
```

#### 2.4.1 Tunneling into your cluster in the cloud

The steps above will trigger istio to set up an AWS Elastic Load Balancer (ELB) to act as an ingress gateway, opening the cluster to requests from outside machines. (The comparable operation on Azure or GCP will do the same.)

Now, use `kubectl get svc --all-namespaces` (or the `lsa` pseudo-target in `k8s.mak`) to verify that you have 3 new services under a new `istio-system` namespace: `istio-egressgateway`, `istio-ingressgateway`, & `istiod`.

**Tip:** Some commands produce output with long lines, which may wrap around if your window is not wide enough.  To clip the lines, pipe the output through the `cut -c -N` command, where `N` is the maximum line width you want. For example, to clip `kubectl` output to a 140-character length:

~~~bash
/home/k8s# kubectl get svc --all-namespaces | cut -c -140
~~~

Finally, you will need to determine how to access your cluster.
You can get the required external IP address using `kubectl`:

```bash
/home/k8s# kubectl -n istio-system get service istio-ingressgateway | cut -c -140
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP                                                              PORT(S)      
istio-ingressgateway   LoadBalancer   10.100.255.11   a844a1e4bb85d49c4901affa0b677773-127853909.us-west-2.elb.amazonaws.com   15021:32744/T
```

The `EXTERNAL-IP` is the entry point to your cluster. The example above (for EKS) shows the rather ungainly DNS name (`a844a1e4bb85d49c4901affa0b677773-127853909.us-west-2.elb.amazonaws.com`).

**Whenever there is no way into your cluster, you will have a `<pending>` as your EXTERNAL-IP.**  Check that Istio is running, using the `kubectl get svc --all-namespaces` command described above.

## 3. Deploying your Services

There are several steps required to deploy and run these three services under Kubernetes.

### 3.1 Building your images

As in Assignment 3, you will need to build the containers and push them to the container registry. This time, you will be building three services (S2, the music service you have used so far, along with S1 and DB).

Build your images with (`cri` short for container registry images):

```
/home/k8s# make -f k8s.mak cri
```

Finally, there is one manual step left before the system can come up auto-magically: to open up access of your container repositories to allow public access. This is reasonable within the context of this exercise because the work involved to set up authentication from the cloud provider (AWS, Azure or GCP) back to `ghcr.io` is more than I can bear at this time.

Refer to GitHub's documentation to [set public access on your container repositories](https://docs.github.com/en/packages/guides/configuring-access-control-and-visibility-for-container-images#configuring-visibility-of-container-images-for-your-personal-account)

### 3.2 Kubernetes operation

Kubernetes operates by way of a declaration specified via a manifest file. There are two formats supported for a manifest file--JSON & YAML--with YAML the preferred format. See [here](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/). A manifest file contains one or more declarations of resource for k8s to set up.

This exercise contain 3 services: s1, s2 and db. s1 and db are comparable in that they are both unversioned. (There is only one version of either.) In contrast, s2 is provided with 2 versions. Note that the YAML files in `cluster` are similarly templatized (have a `-tpl` suffix) as the makefiles. As before, only use the files without the `-tpl` suffix. You will find all manifests in the ``cluster`` directory.

Our usage of ``istio`` necessitates a manifest (``cluster/service-gateway.yaml``) to declare the ingress gateway to allow traffic into the cluster.

Each resource in a manifest file starts off with 4 common element: `apiVersion`, `kind`, `metadata` and `spec`. The first three are standardized while the last (`spec`) varies according to the resource. Refer to the documentation on [k8s objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/).


We will use a combination of a `Deployment` resource to declare the desired resources to power your container and a `Service` resource to expose the containers' capabilities to the world. (Refer to the documentation for [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) and [Service](https://kubernetes.io/docs/concepts/services-networking/service/))

Refer to ``cluster/s2-dpl-v1.yaml``, which specifies the `Deployment` for the Music service (S2) and whose `spec` section looks similar to:
```  
    spec:
      serviceAccountName: svc-s2
      containers:
      - name: cmpt756s2
        image: ghcr.io/<YOUR GITHUB USERID>/cmpt756s2:v1
        imagePullPolicy: Always
        ports:
        - containerPort: 30001

  ```

Note that your manifest contains your Github id in place of  `<YOUR GITHUB USERID>`. (This was performed during the instantiation process at the start of this exercise.)

You will now run your container in your cluster by ``APPLY``ing this manifest using
the pseudo-target `s2`. (You are using a pseudo-target instead of just applying the single file `s2-dpl-v1.yaml` because there is more machinery involved that I won't get into presently.)

```bash
/home/k8s# make -f k8s.mak s2
```

Verify that the service is operating with:
```bash
/home/k8s# make -f k8s.mak ls
```

Shutdown the service and the containers deployed to provide it:

```bash
/home/k8s# kubectl -n c756ns delete svc cmpt756s2
/home/k8s# kubectl -n c756ns delete deployment cmpt756s2-v1
```

### 3.3 Querying the music service

This version of the music service  (S2) is not standalone.  It relies upon several other resources:

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

The Kubernetes command is awkward and you have to repeat it every time you want the logs.  The k9s tool provides a faster, interactive way to get logs.

1. In a separate terminal window, start the tools container.
2. Start k9s:

   ~~~bash
   /home/k8s# k9s
   ~~~

3. K9s will open up its "pods" display. You should see two, one each for the database and music services. Locate the one for the music service (it will begin with `cmpt756s2-v1`), move the cursor down to select it, then press `Return`.
4. The pod for the music service includes three containers.  Two of them were injected by Istio, while one of them is the container for our music code. That one will be named `cmpt756s2`.  Move the cursor to select it, then press `Return`.
5. You will see the most recent lines of the music service log. In fact, the music service will show requests every five seconds or so. These are health checks, requests from Kubernetes to determine if the music service meets minimal standards of functionality.
6. To see the beginning lines of the log, press `1`.  You will return to this when you want to locate the unique code for the "bug".
7. Leave the log view by pressing `Escape`.
8. Leave the S2 pod by pressing `Escape` (again).
9. Scroll to the database pod, whose name starts with `cmpt756db`, and use the same steps as above to view its log.
10. Leave the database log scrolling for now.
11. When you want to leave k9s, press `:` to enter command mode and in the command line enter `q`  followed by `Return`.

To complete setting up the music service, we need to start DynamoDB and load it with initial data:

~~~bash
/home/k8s# make -f k8s.mak loader
~~~

Watch the k9s log to see the lines where the loader calls the database to load the values.  The lines  will look like:

~~~
127.0.0.6 - - [15/Dec/2021 17:56:32] "POST /api/v1/datastore/load HTTP/1.1" 200 -
~~~

where `datastore` is the service and `load` is the operation, in distinction to `health` and `readiness`, the automated checks from Kubernetes.

To test that all these pieces are now working, we need to start the music client. In a separate (third) window, run the client,
where `EXTERNAL-IP` is the long DNS name you found in the section, "Tunneling into your cluster in the cloud", such as
`a844a1e4bb85d49c4901affa0b677773-127853909.us-west-2.elb.amazonaws.com`:

~~~bash
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

### 3.4 Fixing the "bug" in the music service

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

### 3.5 Reflection and look ahead

That was a lot of material.  At this point, it may well seem like using Kubernetes is far more work than running containers by hand. Do we really benefit from all this infrastructure?  Yes, we do:

1. Most of the work in this lesson has been setting up the cluster, installing Istio, defining a namespace, and so forth.  You typically will not be doing this often.  A cluster is created once, then services run upon it.
2. The Kubernetes control algorithms can be managed by automated tools.  For example, in future assignments, you will set up the entire cluster, including some extra services that we didn't introduce here, by the pair of commands:

   ~~~bash
   # Choose your cloud provider
   /home/k8s# make -f eks.mak start OR make -f az.mak start OR make -f gcp.mak start
   # Now configure it --- same command, regardless of cloud provider
   /home/k8s# make -f k8s.mak provision
   ~~~

  You entered all the commands explicitly in this assignment just to gain some familiarity with the underlying mechanisms.  Going forward, you can use the simpler, automated operations.
3. Starting a service was much simpler:  Just send a description of the service to Kubernetes and it selects a machine, pulls the container image, sets up its ports and files, runs the container on the chosen machine, and performs ongoing health checks.
4. Revising a service was as simple as starting it, with the rollout algorithm ensuring minimal disruption as the revision is deployed.

Actual applications will comprise far more services than the three simple ones (Music, Users, and Database) used in this course. The effort to manage large numbers of services by hand would quickly outstrip human capacity. Kubernetes manages the containers and the underlying cluster, allowing developers to focus on their applications.

### 3.6 Getting into your cluster

BLERG I recommend deleting this part ...

Review section "Tunneling into your cluster in the cloud" to look up the EXTERNAL-IP for your cluster in turn. (This exercise is setup to use port 80.)

Fill in the IP/DNS-port combo into ``api.mak`` as the IGW variable.

Study the following pseudo-targets inside ``api.mak`` which will call the API. This exercise only uses `curl`. Pay attention to the variables used to pass the various parameters to `curl`.

| `api.mak` psuedo-target | operation |
| - | - |
| cuser | create a user |
| apilogin | login as a specified user |
| apilogoff | logoff as a specified user |
| uuser | update a user |
| duser | delete a user |
| cmusic | create of a song |
| dmusic | delete of a song |

Fill in the variables inside `api.mak` as appropriate.

For each cluster, perform the following:

* 1. Create 2 users.
* 2. Delete one of the 2 users.
* 3. Update the remaining user.
* 4. Login as the remaining user.
* 3. Create 2 songs as the user that you've log in as.
* 6. Delete one of the 2 songs.
* 7. Logoff from the previously log-in user.

Save the output of each operation. With the creation of user and music, add a serial no to distinguish the various runs (`mv logs/cuser.out logs/cuser1.out` etc). You will have 9 files in total.


## 4. Cleaning up your cluster

Remember to shutdown your cluster after this exercise.
You need to be particularly careful about your cloud-side clusters as they are expensive to keep over longer periods of time (overnight). Remember that you can use `allclouds.mak` to help with this.


## 5. Submission

BLERG
Work through section 3.3 above for Minikube and one cloud cluster (one of AWS, Azure or GCP). For clarity of understanding, you may choose to run thru 2 or more cloud clusters. However, this exercise submission requires the submission of exactly two clusters: Minikube and one cloud cluster of your choosing.


### Collect the Material

Here's a checklist of the items to collect:

| output file | Note |
| - | - |
| logs/mk-start.log | output file for creation of Minikube cluster |
| logs/mk-stop.log | output file for deletion of Minikube cluster |
| {eks|aks|gks}-start.log | output file for creation of {EKS|AKS|GKE} cluster |
| {eks|aks|gks}-stop.log | output file for deletion of {EKS|AKS|GKE} cluster |

For each cluster:

| output file | Note |
| - | - |
| logs/cuser1.out | output file for creation of user 1 |
| logs/cuser2.out | output file for creation of user 2 |
| logs/duser.out | output file for deletion of a user |
| logs/uuser.out | output file for update of a user |
| logs/apilogin.out | output file for login as a user |
| logs/cmusic1.out | output file for creation of song 1 |
| logs/cmusic2.out | output file for creation of song 2 |
| logs/dmusic.out | output file for deletion of a song |
| logs/apilogoff.out | output file for logoff as a user |


Paste the content of each file above into the document below.


### Create a PDF

Open the [submission template](https://docs.google.com/document/d/1PqP4dIvSvXvzSIri_D9i4lvABIQC3IU6VdUaQu_UPwI/edit?usp=sharing)(GDoc format).

Fill in:

a. The header box at the top of the document.

b. Content from the steps above.

c. Answer the additional questions.

When complete, generate a PDF.

You must name your PDF according to the pattern: **SFU-student-no**`-e4-submission.pdf` where **SFU-student-no** is the  numeric id (typicall 30...) assigned to you upon entering SFU. Unfortunately, you will be penalized for incorrect filename because of cascading dependencies for a large class.

**A penalty will be assessed for failure to name your submission appropriately.**

### Canvas submission

Navigate to this exercise and upload your PDF.



### Reference

[Introduction to Kubernetes](https://www.youtube.com/watch?v=VnvRFRk_51k)
A tight 15 min introduction to Kubernetes.

[TechWorld with Nana](https://www.youtube.com/channel/UCdngmbVKX1Tgre699-XLlUA)
YouTube Channel with tutorials for many distributed systems topics including [Docker](https://www.youtube.com/watch?v=3c-iBn73dDE) and [Kubernetes](https://www.youtube.com/watch?v=X48VuDVv0do).
