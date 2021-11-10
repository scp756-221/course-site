---
title: Exercise 7: Service mesh with Istio and Kiali
style:
  background: red
---

## Introduction

In the last exercises, you used Prometheus and Grafana to gather and
report system metrics. The metrics for that exercise were gathered
by our services using the Python
[`prometheus_flask_exporter`](https://github.com/rycus86/prometheus_flask_exporter)
library.  These metrics record the flow of requests entering a node,
with no record of where they came from or any outgoing requests. The
Grafana dashboards summarizing the metrics show rate of flow at each
node but provide no indication of the routes of individual requests.

Our platform extends Kubernetes by installing [Istio](https://istio.io/), a
[service mesh](https://www.redhat.com/en/topics/microservices/what-is-a-service-mesh).
Istio works by injecting a second container into every pod. This injected
container, referred to as a *sidecar*, monitors the network traffic into and out of the
pod. These sidecar can be configured to route requests, provide
metrics to Prometheus, limit excessive traffic, and test system
robustness by deliberately introducing transmission errors and delays.

By tracking traffic as it enters and exits their pods, the sidecars
work within the application structure. They record which services call
each other and can even define the structure of such calls.

In this exercise, you will get a brief introduction to the Istio
service mesh and its dashboard, [Kiali](https://kiali.io/).  In
the last exercise you looked at services individually; in this one you
will look at the connections between services and thereby arrive at a wholistic picture
of the application. This exercise concludes by mutating the
system with the addition a new version of the music service, thereby providing
a sandbox for experimenting with shifting the traffic between the versions.

## Setup
To prepare, change to subdirectory `e-k8s` of the course repository.
Start up and provision a cluster as you did in Exercise&nbsp;5.

Once the cluster is provisioned and running, retrieve the URLs for
Kiali and Grafana as before:

~~~
$ make -f k8s.mak kiali-url
http://35.227.144.239/kiali
$ make -f k8s.mak grafana-url
http://35.233.168.89:3000/
~~~

Bear in mind that the values you print will differ from the above. In
particular, AWS clusters will have long hostnames rather than IP
addresses.

As in the prior exercises, use `gatling.sh` to send traffic into the cluster/application: one user on the User service and five users on the Music service.

~~~
$ tools/gatling.sh 1 ReadUserSim
... output ...
^c to interrupt

$ tools/gatling.sh 5 ReadMusicSim
... output ...
~~~

Paste the Kiali and Grafana URLs into two browser windows.

Select the Grafana window and verify that the Gatling jobs are
correctly running. You should see the specified number of requests
sent to the User and Music services.

## The Kiali graph

Once you have confirmed the Gatling commands are running, select the
Kiali window and click on the `Graph` label on the left sidebar. Using
the controls, configure the graph as follows (controls listed starting
at top left, proceeding left to right):

* Namespaces: *c756ns*
* Graph type: *Versioned app graph*
* Display interval: *Last 1m*
* Refresh interval: *Every 30s*
* Display:
  * Show Edge Labels: *Request rate*
  * Show: *Compressed Hide*, *Operation Nodes*, *Service Nodes*, *Traffic
    Animation*
  * Show Badges: *Virtual Services*

This graph shows the structure of our microservice application,
together with the sources and sinks of requests.
Nodes in this graph indicate either a service, a request source, or a
request sink. The arcs between nodes indicate traffic, as measured by
the mesh (gathered by the sidecars on both sides of the arc).

### Nodes in the Kiali graph

The two leftmost nodes in the graph are labelled
`istio-ingressgateway` and `unknown`. These are where incoming
requests enter the mesh.

Requests entering from `istio-ingressgateway` are from outside the
cluster.  All the requests made by the Gatling simulations enter
via this node.

The `unknown` node is Prometheus. Requests from this node are requests
for metrics values, one per node every 30 seconds. Prometheus is
listed as `unknown` because in our configuration it is not included in
the mesh.

The three middle nodes represent our microservices, `cmpt756s1`,
`cmpt756s2`, and `cmpt756db`. Each service is represented by a service
definition (indicated by a triangle), and a deployment (indicated by a
square). The deployment is also labelled by its version.  For now, every service has
deployed just a single version. We will add a second version of S2 later
in this exercise.

**Terminology note:** istio refers to Kubernetes Deployments as
"Workloads".

At the far right is a five-sided node representing Amazon
DynamoDB.

### Arcs between Kiali nodes

The arcs between nodes are derived dynamically, in response to metrics
reported by the sidecars (i.e., the Envoy proxy) running alongside every service. The presence of an arc
indicates traffic has been observed between those two nodes within the
most recent display interval. The stream of bubbles show the volume
and latency of the requests. The number of circles represents the
volume of requests, while their rate of travel represents their
latency.

The request rate, in requests/s, is also displayed on each arc. If a
rate is hard to read due to the animated circles, hover your cursor
over the arc to pause the animation and to display the rate.

If the two Gatling jobs are running correctly, the path from
`istio-ingressgateway` to S2, through the DB service to DynamoDB,
should be very active, while only modest traffic is indicated from
`istio-ingressgateway` to S1, through the DB service to DynamoDB.

The arcs are currently all green, indicating that every request was
successful. If a small proportion of requests fail, the arc will turn
yellow, while a red arc indicates a high proportion of failures. Failed
requests are indicated by animated squares rather than circles.

The request rate on every arc from Prometheus (`unknown`)
is 0.03&nbsp;requests/s, equivalent to 2&nbsp;requests/min. Recall
that our system set the Prometheus sampling interval to 30&nbsp;s.

Verify that the rates of requests from `istio-ingressgateway` to S1
and S2 match the rates of the Gatling jobs you started earlier. Each
simulated user in those jobs submits requests at a rate slightly less
than specified by the invocation of `gatling.sh` because metric requests have been excluded. (Thus, S1 will indicate as < 1; S2, < 5.)

The units for the rate to DynamoDB are different: Because DynamoDB
uses a protocol inaccessible to istio (or Kubernetes, for that
matter), Kiali reports the rate in KB/s.

Health check requests are not recorded by the sidecars, so they are
not included in the displayed rates.

### The zoomed view

Zoom in to the S2 node by double-clicking anywhere within the
enclosing rectangle. This zoomed view shows S2 at its centre, with its
sources to the left and its sinks to the right.  The sum of the
incoming Gatling and metric requests entering the service (the
triangle) equals the requests sent to the deployment (the square).
Every Gatling request is a read from the Music table, so all of them
are passed to the DB microservice. On the other hand, the metric requests are not passed
on, so the outgoing rate from
the deployment to DB exactly matches the incoming rate from
`istio-ingressgateway`.

Click on the arc between the `cmpt756s2` service (triangle) and the
`v1` deployment (square). The arc will be emboldened and the right
sidebar will show detailed statistics and a plot.

Set the display interval to "Last 5m" and the refresh interval to
"Pause" (both at top right).  Pausing the display will suspend the
update/animation of the plot while you review it.

Select the "Traffic" tab in the right sidebar. The top row shows the
proportions of requests returning success and error codes.  The bar
below that further subdivides the status codes by their leading digit. Below
that is a plot of the average latency and its 50th, 95th, and 99th
percentiles. The plot has one point for the most recent
sample and for each of the preceding five minutes.  Depending upon the
cloud vendor, the P99 value will likely be under 150&nbsp;ms.

Reset the display interval to "Last 1m" and the refresh interval to
"Every&nbsp;30s". Press the blue "Cycle" button for an immediate
refresh of the graph of the services.

## Deploying Version 2 of S2

Now you will deploy a second version of S2.  This will be a
[blue/green deployment](https://www.redhat.com/en/topics/devops/what-is-blue-green-deployment),
where V1 continues to run and we use istio to transfer 10% of the
traffic to V2 to determine whether it is sufficiently reliable to
handle the full load. You will use istio VirtualService's DestinationRule
feature to perform this split.

### The S2 Deployment, Version 2

The deployment of Version 2 of S2 is specified in file
`s2-dpl-v2.yaml`. The relevant portion is:

~~~
    ...
    spec:
      ...
      containers:
      - name: cmpt756s2
        image: ghcr.io/YOUR-GITHUB-ID/cmpt756s2:v2
	...
~~~

where `YOUR-GITHUB-ID` is your GitHub id.

The key change from V1 is the `:v2` tag for the container image. This
marks the image as Version 2 of the code.

### The S2 VirtualService, Version 2

Once the above Deployment is installed (we'll do that in a moment) and
a container run from the `v2` image, traffic needs to be split between
the original Version&nbsp;1 deployment and the new Version&nbsp;2
one. This is specified by the VirtualService defined in the file
`s2-vs-v2.yaml`.  The relevant portion is:

~~~
...
spec:
  ...
  http:
  - match:
    ...
    route:
    - destination:
        host: cmpt756s2
        subset: v1
      weight: 90
    - destination:
        host: cmpt756s2
        subset: v2
      weight: 10
...
~~~

The specification weights 90% of the traffic to `v1`, with the
remaining 10% going to `v2`.

### Deploying Version 2

Ensure that your Kiali graph is zoomed in on `cmpt756s2`.  Then deploy
the new version, updating both the code and VirtualService, via the
script `tools/s2ver.sh`:

~~~
$ tools/s2ver.sh v2
~~~

The script consists of a single call to make target `s2`, modified by
the environment variable `S2_VER`:

   ~~~
   S2_VER=v2 make -e -f k8s.mak s2
   ~~~

Observe the Kiali graph. It will take a minute to a minute and a half
for the updated traffic flows to be recorded by the sidecars,
propagated through Prometheus, and updated on Kiali. After this delay,
you should see that the S2 Service traffic is split between two
versions, v1 and v2. Recall that the VirtualService continues to route
90% of the traffic v1 and only 10% to v2.  The volume of animated
icons should reflect that split.

However, there's a problem:  Our new Version&nbsp;2 code has a bug!
It rejects about 50% of the read requests it receives with a 500
error status. Click on the arc from the S2 Service (triangle) to the
V2 Deployment (square) and explore its details in the right
sidebar. Set the display interval to "Last 5m" or "Last 10m" to get a
more reliable estimate of the proportion of error responses.

Our Blue/Green deployment strategy worked&mdash;we discovered a bug
while only disrupting about 5% of our traffic.  If we had done a complete
switchover to the new version, half the traffic would have suffered.

## Analyzing the flow (for submission)

Before repairing the disruption, you must do a short analysis for
submission.

Allow the system to run for 10&nbsp;minutes with both versions, then
go to the Kiali graph zoomed in to S2. The graph should show the
incoming requests, the S2 service (breaking out two versions of it), and the traffic
entering `cmpt756db`. Set the refresh interval to "Pause" and the
display interval to "Last 10m".  Turn off traffic animation (under the
Display dropdown menu) so that the traffic rates are readily visible.

Make a copy of the
[Ex. 7 submission template](https://docs.google.com/document/d/1M8UOisRjChiuGtjbojg6cbQOaOixcURKdvm7EY0C4Ug)(GDoc
format).

Fill in the header box at the top of the document.

Fill in the answers for the following questions:

1. Take a screen shot of the graph zoomed-in graph. **Make sure that
   the traffic rates are clearly readable.** Tip: Hover the cursor
   in the outer rectangle of S2 but not on any symbol&mdash;the rates
   will be displayed in a larger font.

   Paste the screen shot into your document.  Your screen shot should
   look somewhat like a labelled version of the following:

   ![Flow graph of S2 with two versions, showing sources and DB sink](Sample-unlabelled-zoomed-graph-of-s2.png)

2. What is the total incoming rate of requests from the Gatling
   simulation and Prometheus?
3. What is the total rate of requests sent to `cmpt756db`?
4. Why is the total rate of requests entering S2 larger than the total
   rate of requests passed on to `cmpt756db`? What two types
   of requests are not passed on to `cmpt756db`?
5. What is the total rate of requests from the S2 service (triangle)
   to the v1 and v2 deployments (squares)?
6. Is the total sent to the deployments (Question&nbsp;4) equal to the
   total incoming requests (Question&nbsp;1) or the total passed to
   `cmpt756db` (Question&nbsp;2)? Note: Sums may slightly differ
   due to rates being rounded to the nearest .01. Focus on substantive
   differences.
7. For each deployment, v1 and v2, break down its incoming rate into
   three categories: successful music request and the two other
   categories you identified in Question&nbsp;3.

### Create a PDF

Generate a PDF when you are done filling in your copy with the above answers.

You must name your PDF according to the pattern:
**SFU-student-no**`-e7-submission.pdf` where **SFU-student-no** is the  numeric id (typically 30...) assigned to you upon entering SFU. Unfortunately, you will be
penalized for incorrect filename because of cascading dependencies for
a large class.

**A penalty will be assessed for failure to name your submission appropriately.**

Submit your answer after completing this exercise.


## Repairing the disruption (overview)

Now that we have uncovered the problem, we must take four steps to
address it:

1. Adjust the routes so that all traffic goes to Version&nbsp;1, which
   is correct.
2. Fix the bug in Version&nbsp;2 and deploy the fix.
   Adjust the routes back to the 90/10 split and check that the bug
   was in fact fixed.
3. Once we are confident that Version&nbsp;2 is running correctly,
   adjust the routes to send all the traffic to Version&nbsp;2.
4. Delete the Version&nbsp;1 deployment, which is no longer receiving
   traffic.

## 1. Reroute all traffic to Version 1 of S2

Fortunately, reversing the failing update is simply a matter of
specifying the old version:

~~~
$ tools/s2ver.sh v1
~~~

Observe the results in the zoomed Kiali view. Set the display interval
to "Last 1m" to see the results more quickly.

You will see some disruption of service (confusingly, Version&nbsp;1
might vanish for a display cycle), settling down into a configuration
where all the `istio-ingressgateway` traffic, the requests from our
Gatling simulated users, goes to `v1` and only the Prometheus metric
requests go to `v2`.

## 2. Fix and redeploy Version 2 of S2

Fortunately for us, fixing the bug in the Version&nbsp;2 code is easy:
In file `s2/v2/app.py`, edit the `PERCENT_ERROR`
definition so that it sets the error rate to 0:

~~~
PERCENT_ERROR = 0
~~~

Save the updated file, clean the log files, and redeploy `v2`:

~~~
$ make -f k8s.mak clean
$ tools/s2ver.sh v2
~~~

**Note:** The `make -f k8s.mak clean` is required due to a limitation
  of the makefiles used in this course. It ensures that your update to
  `app.py` will actually get incorporated into the deployed
  containers.

Observe S2 in the Kiali zoomed view, once again with a display
interval of "Last 1m".  After a brief disruption (one or two minutes),
it should settle in to show the traffic split 90/10 between the two
versions, with *all* traffic successful. Every arc should be green.

**Sidebar:** One of the challenges in diagnosing
  problems with Kubernetes is knowing what code is *actually running* in
  a container.  If Version&nbsp;2 is not performing as you expect, you
  can open a shell directly into the deployed container via the
  following command:

~~~
$ kubectl exec -it -n c756ns deploy/cmpt756s2-v2 -c cmpt756s2 -- bash
root@cmpt756s2-v2-76995ccd68-qpfzc:/code#
~~~

The prompt following the command indicates that you are invoking
commands in the shell in the S2 container.

To see the Python code in the container, run `more`:

~~~
root@cmpt756s2-v2-76995ccd68-qpfzc:/code# more app.py
"""
SFU CMPT 756
Sample application---music service.
"""
...
~~~

Once you are done, use the `exit` command to end the guest session and
return to the shell on your local machine:

~~~
root@cmpt756s2-v2-76995ccd68-qpfzc:/code# exit
exit
$
~~~

## 3. Route all traffic to Version 2 of S2

Once you have determined that the bug is fixed and Version&nbsp;2 is
correctly handling its 10% of the traffic, it is time to route all the
traffic to the new version.

Edit the file `cluster/s2-vs-v2.yaml`.  Change the `v1` weight to 0 and
the `v2` weight to 100:

~~~
    ...
    - destination:
        host: cmpt756s2
        subset: v1
      weight: 0
    - destination:
        host: cmpt756s2
        subset: v2
      weight: 100
~~~

Redeploying `v2` will update the route weights. You do not need to
make the `clean` target this time:

~~~
$ tools/s2ver.sh v2
~~~

After an initial disruption, the system should settle into a pattern
where all the `istio-ingressgateway` traffic flows to `v2` and only
Prometheus's 0.03/s requests continue to `v1`.  If you look closely,
the arc from `v1` to `cmpt756db` will be gray and have no rate
displayed, indicating that no database requests were made by `v1`
during the display interval.

## 4. Delete Version 1 of S2

Once you have determined that `v2` is correctly handling all the
traffic, you can delete `v1`, which is only receiving metric
requests. Issue the following command:

~~~
$ kubectl -n c756ns delete deploy/cmpt756s2-v1
~~~

After a minute or two's propagation delay, `v1` should vanish and the
S2 service should only have `v2`. Note also that Prometheus
(`unknown`) will now only be sending half as many metric requests to
the S2 service because it only has a single deployment.

## If you need to restart this exercise from scratch ...

If you find the system in a confused or inconsistent state and you
want to start over, you can delete the application and any changes you
made to the files, leaving a clean stack to restart from:

~~~
$ make -f k8s.mak scratch            # Delete the microservices on the cluster
$ git checkout s2/v2/app.py          # Undo any changes to the V2 code
$ git checkout cluster/s2-vs-v2.yaml # Undo any route updates
~~~

The Kiali graphs will become empty because the `c756ns` namespace has
been cleared.

## Cleanup

If you are running this exercise on a vendor and will not be using
this cluster for a while, you can reduce charges by shutting down the
cluster.

List all your running clusters via the `allclouds.mak` target `ls`,
which scans AWS, Azure, and GCP for running clusters in your
account. It will also print the current level of DynamoDB service, as
well as a reminder to check for background Gatling jobs.

~~~
$ make -f allclouds.mak ls
....
~~~

Shut down clusters via the `stop` target of the vendor's
makefile. Stop the Gatling jobs via the shell `kill` command.

## Summary

The istio service mesh injects a "sidecar" container (of the Envoy proxy) into each pod of each
service. Because the sidecar monitors the service's communications
in to and out of the container, it can report metrics on the
interactions *between* services, as opposed to metrics that only
summarize actions at a *single* service.  The Kiali dashboard collects
metrics from the istio service mesh into a graph displaying the
information flows between services.

In addition to reporting on these flows, the sidecar can
also intelligently rout requests between service. The operations staff can
redirect a fraction of traffic to an updated or new version of a
service, only cutting over entirely when the new version is
demonstrably stable.

By separating the definition of a service's connection to its
upstream and dependent services from the
definition of the service itself (e.g., a container's `app.py`),
service meshes provide a flexible tool for operating a
microservice architecture.
