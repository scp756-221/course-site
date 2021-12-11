## Introduction

In Part&nbsp;1 of this assignment, you will once again run the music service on an EC2 instance but with an important difference: Instead of tediously transferring source files  to the remote instance, building the service there, and then running it, you will simply pull down a previously-built container and run it on the instance. This will simplify the build process but make it a bit harder to get the server logs.

In Part&nbsp;2, you will explore further features of containers and the `docker` command.

**This assignment has several prerequistes:**

**1. You must have obtained a GHCR access token and saved it in `cluster/ghcr.io-token.txt`.  See Assignment&nbsp;0 for details.**

**2. You must have completed the Docker exercises previously assigned.**

## Part&nbsp;1: Running a container on a remote EC2 instance

Rather than building the music service on the remote instance, in this exercise we will pull a previously-built container image from the GitHub Container Registry (GHCR).  But first, the image must be built and pushed to GHCR.

Because this assignment is about using the `docker` command, we will be using the command directly, rather than calling it inside a predefined shell script or Makefile. This will give you some familiarity with the many options the command requires. To avoid typing these long commands in, risking errors due to mis-typing, copy the commands from this file and paste them into the appropriate terminal window.

However, when you paste a command into the terminal window, **you must edit the personalized arguments before pressing `Return`:**

* `REGID` must be replaced with your GitHub userid.
* `KEY-FILE` must be replaced with the name of the file in your `~/.ssh` directory containing the AWS key used to sign on to EC2 instances.
* `EC2-DNS-NAME` must be replaced with the Public IPv4 DNS name of the EC2 instance.  See Assignment&nbsp;2 for how to locate that name and copy it into your clipboard.

### Building and pushing the container image to GHCR

Begin by switching to the correct directory:

~~~bash
/home/k8s# cd s2/standalone
~~~

Build the Assignment&nbsp;3 version of the music service with the following command:

~~~bash
/home/k8s/s2/standalone# docker image build --build-arg ASSIGN=a3 -t s2-standalone:v0.75 .
~~~

Check that the image was created by listing all images named `s2-standalone` on your machine:

~~~bash
/home/k8s/s2/standalone# docker image ls s2-standalone
REPOSITORY      TAG       IMAGE ID       CREATED         SIZE
s2-standalone   v0.75     ...            ...             230MB
.... [perhaps others] ...
~~~

Pushing the image to GHCR is a three-step process. First, you have to log in:

~~~bash
/home/k8s/s2/standalone# cat ../../cluster/ghcr.io-token.txt | docker login ghcr.io -u REGID --password-stdin
~~~

You only need to do this occasionally---a login lasts for days until it times out.

Second, you need to tag the image with the prefix for GHCR and your GitHub userid:

~~~bash
/home/k8s/s2/standalone# docker image tag s2-standalone:v0.75 ghcr.io/REGID/s2-standalone:v0.75
~~~

Now that the image is tagged with GHCR as its destination registry, you can push it there:

~~~bash
/home/k8s/s2/standalone# docker image push ghcr.io/REGID/s2-standalone:v0.75
... Messages for twelve or more layers ...
v0.75: digest: sha256:... size: ....
~~~

The container image is now on GitHub's servers, available to be pulled to any machine in the world by anyone who has a GHCR personal access token authorized for that image.

### Pulling the container image to an EC2 instance

As in Assignment&nbsp;2, start up an EC2 instance.  Use the same AMI (Page&nbsp;1), instance type (Page&nbsp;2), and security group (Page&nbsp;6).

Once it is created, transfer the required files. Although you do not have to transfer the source files to the instance, you still have to transfer the access token required to log in to GHCR and the data file used by the service. Run the following:

~~~bash
/home/k8s/s2/standalone# ./transfer-for-run.sh ~/.ssh/KEY-FILE ec2-user EC2-DNS-NAME
~~~

Once the files are transferred to the instance, sign on to it using `signon.sh`.

On the EC2 instance, you will also need to log in to GHCR. The command you use is almost the same as the one you ran locally, only differing in the path to the GHCR access token:

~~~bash
[ec2-user@ip-172-31-12-116 ~]$ cat ghcr.io-token.txt | docker login ghcr.io -u REGID --password-stdin
~~~

After signing on, a single command pulls the container image, packaging all the executable code for the music service, and executes it (the `$PWD` in the following command is a shell variable---you do not have to replace it):

~~~bash
[ec2-user@ip-172-31-12-116 ~]$ docker container run -d --rm -p 30001:30001 -v $PWD:/data --name s2 ghcr.io/REGID/s2-standalone:v0.75
Unable to find image 'ghcr.io/.../s2-standalone:v0.75' locally
v0.75: Pulling from .../s2-standalone
... Layers pulled and expanded ...
Status: Downloaded newer image for ghcr.io/tedkirkpatrick/s2-standalone:v0.75
....
[ec2-user@ip-172-31-12-116 ~]$ 
~~~

Note that we don't see the output from the service as it starts.  We ran the container in "detached" mode (`-d` option), in the background with no connection to our terminal.
We can use another command to check that the container is running:

~~~bash
ec2-user@ip-172-31-12-116 ~]$ docker container ls
CONTAINER ID   IMAGE                                        COMMAND                 CREATED         STATUS         PORTS
be4c48e6b9ed   ghcr.io/.../s2-standalone:v0.75              "python app.py 30001"   4 minutes ago   Up 4 minutes   8000/
~~~

If the container's status reads, `Up [period of time]`, it has started successfully and is still running.

That's it!  The music service is now running on EC2.  Run `mcli` on your own machine, passing it the address of the remote instance (see Assignment&nbsp;2 for details), and verify that the service is running.

### ... But how will we get the logs?

Running a remote container in detached mode with no terminal output is clean and is amenable to automated execution but it does make one part of our job harder:  How will we get the logs when we need to debug the server code?

In `mcli` on your own machine, enter the `test` command:

~~~bash
mql: test
Non-successful status code: 500
~~~

Our familiar "bug" is present in this server but to fix it we will need to see the logs and get our unique code. How do we read them?

Docker provides the `logs` command to get the logs from a detached container.  On the EC2 instance, enter:

~~~bash
[ec2-user@ip-172-31-12-116 ~]$ docker container logs s2
[2021-12-10 01:51:58,036] ERROR in app: Unique code: ...code...
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
...
~~~

where `s2` is the name we assigned the container when we created it via the `--name` parameter of `docker container run`. This is the log we saw in the previous exercises, with the unique code on the first line.

**Background:** With some work, you could configure the `docker` client on your local machine to pull the logs directly from the remote EC2 instance. But that is tricky---more security permissions---so we'll just use the client directly on the EC2 instance. When we move to Kubernetes in the next assignment however, running the client directly on the remote instance will not be an option.

### Fixing the bug

After the previous two assignments, the fix-rebuild-test run-commit cycle should be getting familiar.  The difference this time is that you will rebuild the service on your own machine, push it to the GitHub container repository, pull it to the remote instance, and run it there.

1. *On your local machine*, edit `app-a3.py`.  This is the same sequence you did in previous exercises, just using the unique code for this exercise.
2. *On your local machine*, build the container image using the same command you used above.
3. *On your local machine*, tag and push the rebuilt image to `ghcr.io` using the same command you used above. However, you won't need to log in to `ghcr.io` again, as your prior login will still apply.
4. *On the remote machine*, pull and run the rebuilt image.  This time, you will have to force docker to pull the image.  The remote machine already  has a local copy of the image `ghcr.io/REGID/s2-standalone:v0.75` and by default assumes that is up to date.  We add the `--pull always` argument to force docker to pull the rebuilt image from GHCR:

   ~~~bash
   [ec2-user@ip-172-31-12-116 ~]$ docker container run --pull always -d --rm -p 30001:30001 -v $PWD:/data --name s2 ghcr.io/REGID/s2-standalone:v0.75
   ~~~

Test the revised service by issuing `test` from your local copy of `mcli`. 

Check the logs from the rebuilt service:

~~~bash
[ec2-user@ip-172-31-12-116 ~]$ docker container logs s2
...
208.78.41.77 - - [10/Dec/2021 18:34:47] "GET /api/v1/music/test HTTP/1.1" 200 -
...
~~~

This time there should be no stack trace and the call to `/api/v1/music/test HTTP/1.1` should return a `200` status code, indicating success.

**Tip:** If you're still getting the error code, you may have forgotten to issue the `docker image tag` command after rebuilding the image.  Without that command, you will have inadvertently pushed the *old* image to GHCR and pulled it down to the remote instance, getting exactly the same version you had the first time.

If the success code is reported, the "bug" has been fixed.  Commit and push the revised source code.

### Terminate the EC2 instance

Once you are done with all the above steps, terminate the EC2 instance.  **You will accumulate charges until the instance is terminated.** See Assignment&nbsp;2 for details.

### Reflecting and looking ahead

Using container technology simplified some things and made others more complex.

Simpler operations:

* **Building locally, running remotely:** We didn't have to transfer all the build files.  Instead, we could build the executable locally, then transfer it to the remote machine packaged up in a container.  Our build environment is only a few files but most production systems are *much* bigger. This is significant simplification.

More complicated operations:

* **Getting the logs:** Getting the logs from a detached container requires a new command, `docker container logs`, that works with container technology. This is more work than simply glancing at a terminal window.
* **Granting access to GHCR:** Because GHCR is centralized and globally visible, we had to restrict access to our image via use of an access token. That token had to be transferred to the remote instance to enable it to pull the image from GHCR.

Same operations:

* **Data files:** We still had to send the data files to the remote instance, same as we did for Assignment&nbsp;2.
* **Managing the remote instance:** We had to do the same AWS operations to create and terminate the remote instance as we did in Assignment&nbsp;2.

In the next assignment, we will use Kubernetes. We will see that it manages the remote machine instances and pulls the images, while its security requirements make GHCR access harder.

### Repository stores versus container (image) registries

The distinctions between GitHub and GHCR and between Git repositories ("repos") and container images can be confusing, so we've summarized them in this table:

<table>
<caption>Repository stores versus container (image)<sup>a</sup> registries</caption>
<thead>
<th>Purpose</th>                     <th>Name</th>              <th>Domain name</th>            <th>Contents</th>        <th>Sample push</th>                   <th>Sample pull</th>
<thead>
<tbody>
<tr>
<td>Developing source code</td>      <td>GitHub<sup>b</sup></td><td><code>github.com</code></td><td>Git repositories</td><td><code>git push</code></td>            <td><code>git pull</code></td>
</tr>
<tr>
<td>Distributing executable code</td><td>GHCR<sup>c</sup></td>  <td><code>ghcr.io</code></td>   <td>container images</td><td><code>docker image push</code></td><td><code>docker image pull<sup>d</sup></code></td>
</tr>
</tbody>
</table>

**Notes:**

<sup>a</sup> Although these are universally called "container registries" they in fact store container *images*. The images become containers when they are pulled from the registry and run via commands such as `docker container run`.

<sup>b</sup> Other development platforms include [GitLab](https://about.gitlab.com/) and [BitBucket](https://bitbucket.org/product), as well as internal tools installed and run within large organizations.

<sup>c</sup> Other container registries include [Docker Hub](https://hub.docker.com/) and [Red Hat Quay](https://quay.io/), as well as registries run by all the major cloud vendors ([Amazon ECR](https://aws.amazon.com/ecr/), [Google Container Registry](https://cloud.google.com/container-registry), [Microsoft Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/), [Alibaba Container Registry](https://www.alibabacloud.com/help/product/60716.htm), [IBM Cloud Container Registry](https://cloud.ibm.com/docs/Registry)).

<sup>d</sup> The `docker image pull` command is implicitly invoked by `docker container run` if the image to be run is not already present on the local machine. In that case, the image is pulled from the registry before running.

## Part&nbsp;2: Exploring the `docker` command

In this section, we will explore some of the options for the `docker` command.

Perform all these subections in the tools container on your own machine.

### Interactive containers: `-it`

A huge number of calls to run containers begin `docker container run -it --rm ...`.  What do those parameters do?  What happens if they are omitted?

We'll start with `-it`.  The sequence is in fact two options coalesced, `-i` ("interactive") and `-t` ("allocate a pseudo-TTY"). They are almost always used together.  When both used, they connect the container to the current terminal for input.

Try running the music service without them:

~~~bash
/home/k8s# docker container run --rm -v $HWD/s2/standalone:/data -p 30001:30001 s2-standalone:v0.75
~~~

It actually works, though we don't recommend doing this in production.

Shut down the running service by entering `^C` and now test the client without `-it`:

~~~bash
/home/k8s# docker container run --rm --name mcli mcli:v0.8 python3 mcli.py 0.0.0.0 30001 | more
~~~

Whoops!  The client attempts to read input but without the `-it` options no terminal exists.  The result is simply an infinite sequence of end-of-file (EOF) inputs. We added the `| more` pipe to prevent the client from filling your window with hundreds of lines of `*** Unknown syntax: EOF`.

Exit `more` by entring `q` at its `--More--` prompt.

The client container continues to run in the background, so you have to kill it:

~~~bash
/home/k8s# docker container kill mcli
~~~

In summary, there are two typical use cases:

* For interactive containers that will send terminal output and read terminal input, use `-it`.
* For non-interactive containers, do *not* include `-it` but do include `-d` to run "detached".

### Clean up: `--rm`

The `--rm` option (note the two leading `-` characters---this is a single option with a two-character name) tells docker to delete the container as soon as it stops.  Most of the time, this is what we want, but sometimes we want a container to stick around for a while after it completes.

Try running the music service without `--rm` (but with `-it`, of course):

~~~bash
/home/k8s# docker container run -it -v $HWD/s2/standalone:/data -p 30001:30001 --name s2 s2-standalone:v0.75
....
~~~

Terminate the service by entering `^C`.  Normally, this would delete the container. But containers started without `--rm` are retained.

Verify that the container was retained by listing containers with the `-a` option, which includes stopped containers in the list:

~~~bash
/home/k8s# docker container ls -a
CONTAINER ID   IMAGE                                                   COMMAND                 CREATED          STATUS                      PORTS     NAMES
4d3ec64b2898   s2-standalone:v0.75                                     "python app.py 30001"   56 seconds ago   Exited (0) 50 seconds ago             s2
....
~~~

Why would you want to retain a stopped container?  Well, you might want to see its logs, for example:

~~~bash
/home/k8s# docker container logs s2
[2021-12-10 23:33:22,541] ERROR in app: Unique code: f189212871ce7d0e6c4f5101c14b6a288d7c92509dd231473d2c135f281a162e
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:30001/ (Press CTRL+C to quit)
...
~~~

This can be important in getting all the messages logged by a container before it unexpectedly failed, for example.

On the other hand, you don't want to retain every container indefinitely, as that will quickly fill all available storage.  There is no single solution for all use cases, unfortunately.

Remove the stopped container explicitly:

~~~bash
/home/k8s# docker container rm s2
~~~

### Container plumbing: Mapping volumes via `-v`

An important feature of containerized software is that the container's meta-data describes its connections to its environment and then at runtime different resources can be mapped to those connections. One class of such connections is filesystem directories (called *volumes*, making the relevant option `-v`).

The container metadata specifies the directories that the container will read when it runs.  When the container is run, the user can map those directories to *any directory on the host filesystem*. You map a host directory to one required by the container using the `-v` option.

We'll demonstrate this by starting the music service with a different initial list of songs.  So far, whenever we have started the service, it has read the file `music.csv` to get its list of songs.  The `s2/standalone` directory has a subdirectory, `odd`, with a distinct `music.csv` providing a different initial list.

Start the music service, mapping its `/data` directory to `s2/standalone/odd` rather than `st/standalone`:

~~~bash
/home/k8s# docker container run -it --rm -v $HWD/s2/standalone/odd:/data -p 30001:30001 --name s2 s2-standalone:v0.75
[2021-12-11 00:21:31,034] ERROR in app: Unique code: f189212871ce7d0e6c4f5101c14b6a288d7c92509dd231473d2c135f281a162e
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:30001/ (Press CTRL+C to quit)
~~~

In the music client, issue a `read` request.  You will get a different list of songs from the ones that you saw in earlier runs.

Similar possibilities of mapping can be applied to network ports using the `-p` option.

### Image history, layers, and the overlay filesystem

BLERG:  George, I have some ideas for this but this assignment is already quite long and it's taken too long to write.

* Would you like a section on layers?
* Would you like to delete an earlier section to make room for this and keep the assignment at this length?

## Submission

BLERG