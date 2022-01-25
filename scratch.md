How many files are contained in the following commit:

```sh
$ mkdir my-repo
$ cd my-repo
$ cp ~/path/to/prewritten/text.txt README.txt
$ git init
$ git add .
$ git commit -m "Lorem ipsum"
$ echo "*.txt" > .gitignore
```

One
Two
None
===


How many files are contained in the following commit:

```sh
$ mkdir my-repo
$ cd my-repo
$ cp ~/path/to/prewritten/text.txt README.txt
$ git init
$ touch .gitignore
$ git add .
$ git commit -m "Lorem ipsum"
```

Two
One
None
===




Consider the life of a project that proceeded roughly as follows:

```sh
$ mkdir cool-project
$ cd cool-project
$ git init
Initialized empty Git repository in /some/path/to/cool-project/.git/
$ scp buddysbox:/remote/path/to/friends/code/ .
$ git add .
$ git commit -m "Sed ut perspiciatis unde omnis"
[main (root-commit) 9f6312b] Sed ut perspiciatis unde omnis
...
$ git checkout -b buddys-idea
Switched to a new branch 'buddys-idea'
$ cp ~/my/own/ideas .
$ cp ~/templates/.gitignore .
$ git add .
$ git commit -m "Lorem ipsum".
[buddys-idea efef8e0] Lorem ipsum.
$ git tag -a my-starting-point -m "Initial starting point"
$ git checkout -b my-direction
Switched to a new branch 'my-direction'
# do a bunch of hardening
$ git add .
$ git commit -m "Quis autem vel eum iure"
[my-direction 0029a6d] Quis autem vel eum iure
$ git checkout buddys-idea
Switched to branch 'buddys-idea'
$ scp buddysbox:/another/path/to/more/friends/code/ .
$ git add .
$ git commit -m "Excepteur sint occaecat cupidatat"
[buddys-idea de00c69] Excepteur sint occaecat cupidatat
$ scp buddysbox:/another/path/to/more/friends/doc/*.txt .
$ git add .
$ git commit -m "Quod erat demonstrandum"
[buddys-idea 098e15b] Quod erat demonstrandum
```

How many commits are there in this repo altogether?

Five
Four
Three
Two


How many branches are there in this repo altogether?

Three
Two
One


How many commits are there in your own branch (my-direction)?

Three
One 
Two
Four
Five

How many commits are there in your buddy's branch (buddys-idea)?

Four
One 
Two
Three
Five



===
Q3 working

The local Docker cache:
A1: holds images requested by the user 
A2: may be side-stepped via the --no-cache option
A3: Fetches 
F1:
F2:


What happens when you attempt to run a Docker image that has not been previously referenced on your machine?
A: Docker will pull the image from the container registry specified by the image specified.
F: Docker will return a 408: image not found message
F: Docker will pull the image from GitLab
F: Dockre will pull the image from Google

Instructions on how to build a Docker image is written in a file named:
A: Dockerfile
Makefile
Dockerfile.yaml

If your laptop is running low on disk space, what can you do to free up some space:
A1. Use `docker image ls` to determine your largest image and stop that
F2. Use `docker container ls -a` to determine your largest container and delete that
F3. Use `docker ps` to determine your largest container and delete that

If your laptop is running low on memory, what can you do to determine if Docker may be responsible?
A1. Use `docker container ls -a` to check on your running containers
F2. Use `docker image ls` to check on the sizes of your images


Consider the following output:

```bash
$ docker image ls
REPOSITORY                                   TAG               IMAGE ID       CREATED         SIZE
overcoil/trino-base                          <none>            b6ba5e89f538   4 weeks ago     1.98GB
<none>                                       <none>            a1a814dc6ce3   4 weeks ago     1.62GB
<none>                                       <none>            6081ffd05652   4 weeks ago     839MB
<none>                                       <none>            a99109e98354   4 weeks ago     1.62GB
<none>                                       <none>            b6ba5e89f538   4 weeks ago     1.98GB
openjdk                                      11-slim           ecb15898eaf2   4 weeks ago     429MB
ghcr.io/scp756-221/c756-tool                 v1.0beta4         a7c0e566fa1a   5 weeks ago     2.72GB
cassandra                                    4.0               22b9db032997   13 days ago     341MB
cassandra                                    4.0.0             8f0b5142d8ed   4 months ago    338MB
cassandra                                    4.0.1             b082d7bbd165   2 months ago    341MB
gcr.io/k8s-minikube/kicbase                  v0.0.28           e2a6c047bedd   3 months ago    1.08GB
thrift                                       0.9.3             8e12c5473f80   2 years ago     141MB
justincormack/nsenter1                       latest            c81481184b1b   4 years ago     101kB
```

How much disk space would be freed up, approximately, if the image `overcoil/trino-base` was deleted?
A1: 1.98GB
F2: 2 x 1.98GB = 3.96GB
F3: Insufficient informationi to determine



Assume that your local Docker environment is as above. Consider the following:

```bash
$ docker run --rm -d cassandra
5f36041be23e481feb209582e68e5571859370db3989d308a810bad2d7f3e749
$ docker container ls
NAMES              CONTAINER ID   STATUS          IMAGE                                 COMMAND                  SIZE
infallible_jones   5f36041be23e   Up 11 minutes   cassandra                             "docker-entrypoint.s…"   0B
```

Which version of Cassandra is actually running?
A1: Insufficient information to determine
F2: 4.0.1, as that is the most recent
F3: 4.0, as it has the lowest image id


To determine the version of Cassandra that is running, which command would you use?
A1: $ docker inspect container infallible_jones
F2: $ docker image ls cassandra



Consider your environment looks as follow:

```bash
$ ls
Makefile           system1.mak        system2.mak        system3.mak        variants.mak
``

$ make -f system1.mak foo

====

Makefile & Kubernetes (kubectl & k9s)





Kubernetes is a framework for operating some software:


Select all required components within a working Kubernetes cluster:
A1. kubelet
A2. etcd
A3. API server
A4. controller
A5. scheduler
F6. container registry
F7. container repository


The core concept/unit within Kubernetes is that of a "resource". Select all such resources in the list below:

A1. pod
A2. replicaset
A3. deployment
A4. service
A5. secret
A6. configmap
F7. container
A8. node
A9. namespace

Describe the difference between `kubectl` and `k9s` in your own words.


Kubernetes offers the core function of `scheduling` containers to run within the cluster. What is the scheduler component responsible for?

A1. Choosing the node for the container to run on
F2. Signaling the assigned node to run the scheduled container
F3. Monitoring the operation of the pod that hosts the scheduled container


Each node within a Kubernetes cluster contains a `kubelet`. What is the kubelet responsible for?

A1. Watching for an assigned container 
A2. Signaling the assigned node to run
A1. Choosing the node for the container to run on


Which Kubernetes resources abstract the computational aspect of your application:

A1: pod
A2: replicaset
A3: deployment
A4: statefulset
F5: service
F6: gateway


One or more ______ runs within a Kubernetes pod.
A1: containers
F2: nodes
F3: service

A Kubernetes manifest is:
A1: a YAML file delaring one or more resources
F2: a k
Selector

Label

Namespace

Which 

W
Cluster


EXPLAIN

WITH
  item AS (SELECT * FROM deltas3."$path$"."s3://tpc-datasets/tpcds_1000_dat_delta/item" ),
  store_sales AS (SELECT * FROM deltas3."$path$"."s3://tpc-datasets/tpcds_1000_dat_delta/store_sales" )
SELECT * 
FROM store_sales, item 
WHERE ss_item_sk = i_item_sk AND i_item_sk = 1969;

===
There are some addendeum to a3 based on late changes.

There are 2 files to work with:
-~/.aws-a
-~/.ec2.mak

The goal is to have a fast macro for working with EC2 instances. You were previously driving the web console earlier on. We will 
switch to the AWS CLI now.

Run (or use a package)
erun/epkg
-instance id
-public IP
-tag

Login
esshn
-use tag

Check on your status
eps
-see everything

Terminate:
ekill
ekn
-
===

GPU article

1. Sources of Acceleration
  Data Specialization
  Parallelism
  Local and optimized memory
  Reduced Overhead

2. Codesign is Needed

3. Memory Dominates Accelerators

4. Balancing Specialzation & Generality
  Special instructions vs special engines
  Domain-specific parallel computers

===
5. Total Cost of Ownership

6. Accelerator Design
  Programming accelerators
  Creating accelerators with programs

Conclusion

A GPU is to DSA as:
-a subclass is to a superclass
-a superclass is to a subclass
-a singleton is to a class

What techniques do GPU exploit to delivery extraordinary levels of acceleration?
A1. Data specialization
A2. Parallelism
A3. Local & optimized memory
A4. Reduced Overhead

What is the key "shift of tradeoff" between using general purpose processor and a DSA/GPU?
1. Complex algorithm with small memory toward simpler algorithm with large memory.
2. Use of simple data types towards use of matrices
3. Use of local servers towards cloud-based servers 

What is the key idea behind data specialization?
A1. The DSA is designed to executed a limited set of operations on specially organized data/structures
F2. The DSA is designed to executed low power operations
F3. The DSA is designed for sparse data structures

What is the key idea behind a DSA's massive parallelism advantage?
A1. The DSA executes, in parallel, operations that require little to no synchronization/communication
F2. The DSA uses double buffering to maintain a high execution unit utilization

What is the key idea behind the local and optimized memory approach?
A1. Optimize the use of and the traffice into/out of the DSA's memory
F2. Compress all DSA memory structures

What is the key idea behind reducing the overhead when using a DSA?
A1. Amortize the cost of managing instructions over large blocks of data
F2. Eliminate the use of speculative execution


How does a CUDA thread compare to a CPU thread?
A1. A CUDA thread is light-weight with no context switching costs
A2. A CPU thread is full featured as it runs on a general purpose processor
F3. A CUDA thread is longer lived
F4. A CPU thread is not supported byt the CUDA library

How are CUDA threads organized?
A1. CUDA threads are grouped into thread block each of which can share memory and synchronize its operation  
F2. CUDA threads are grouped into a single grid with no possibility of sharing or synchronization

A GPU kernel is:
A1. A program executed by each GPU thread  
F2. A program that manages/schedules the GPU's threads
F3. A program that performs memory transfers between the host and the GPU device

A GPU device contains:
A1. Multiple multiprocessors each of which executes a thread block
F2. Multiple threads which execute in parallel
F3. Multiple blocks of global memory 
