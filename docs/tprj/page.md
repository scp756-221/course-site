## Introduction

The term project provides an opportunity for a small group to explore the technologies introduced to date. You are to apply and observe the principles of the course while gaining hands-on experience through building, scaling and observation of a working distributed application. This project for CMPT 756 requires a small team (5 members; exceptions subject to approval) working over the course of 7 weeks.

The project comprises 5 stages:

1. Choose your technology platform. AWS EKS as introduced by this course is a reasonable choice but if you are interested in exploring alternate Kubernetes platform (e.g., Azure AKS, GCP EKS, etc), you may do so. Kubernetes provides a high degree of portability across the various clouds.
2. Implement three public micro-services. The material presented in the assignments repo already includes two public microservices ("music" and "user"); you only need to add an additional service. This can be trivially a clone/derivative of the existing service (e.g., "playlist") or something that your team wish to explore (e.g., packaging a machine learning model).
3. Run your application with various loads and measure/observe its behaviour.
4. Write up a report summarizing your findings.
5. Record a video walking through and describing your team, approach and application.

Through this project, you will connect the various ideas from the course together including:

1. Scrum methodology for a distributed team
2. Infrastructure as code
2. Containerized application
4. The micro-services distributed system architecture pattern (using REST)
5. Test coverage and load testing of your system.


Key Dates:

| Date | Event |
|-|-|
| Jan 11, 2022 (Tue) | First day of course. |
| Feb 19, 2022 (Sat) | Project team confirmed. |
| Mar 19, 2022 (Sat) | Interim milestone |
| Apr 8, 2022 (Fri) | Last day of course. |
| Apr 12, 2022 (Tue) | Project completion; Final submission. |


## 0. Set up

Organize yourself into a team as you see fit. To use CourSys' group feature, one member of a team creates the group and invites the other team members in. Your team must comprise 4-5 team members. (Exceptions is allowed but *must be approved*.)

After you've formed your team, each member of the project team must navigate to [Github Education](https://classroom.github.com/a/xFDX11DX) to accept the "assignment". The first member of the team will then create and name the team. The repo from Github Education classroom is intentionally empty but provides full access to the teaching team of your team's work.

While the project is graded as a whole, marks are assigned individually to each member according to documented contribution to the project. Therefore, document your contributions via appropriate means (e.g., commit comments, issues assignments, etc). Commit early and often!

## 1. Distributed System Focus

Because this course is focused on the distributed system aspects (as opposed to machine learning or other "AI" aspects) of a cloud application, your term 
project will be similarly focused. To be clear, there is insufficient time to program any of the (albeit interesting) data science 
techniques into your application. If you do choose to implement a wholly new microservices, take care to manage the scope of that. 
Continuing with the example above, 
do not implement smart playlist using a cool ML technique from CMPT 726. These techniques are important but, for the purpose of this course, 
are only overlays for your application. (I will leave them to other courses and/or your future employers.)


## 2. Practice Agile:

As you are working in a team environment and potentially in a distributed (maybe even timezone-wise) fashion, use the tools introduced to date including git and GitHub:

1. Build a backlog of activities to plan and track the team's ideas and work
2. Use a Kanban project ("GitHub project") to monitor your team's activities and progress.
3. Branch and merge each team member's contributions

This course did not cover CI or microservices in-depth but your team can explore these as background/interest dictate.


## 2. Implement/Reuse Supplied Material 

You are encouraged to reuse as much as you can of the material supplied to date within the course. You may also bring in other technologies/libraries with approval; when in doubt, ask first.

Within the constraints of time (about eight weeks max), your application can not be too sophisticated. Keep this in mind!


### Guides
There are three additional guides that introduce additional material within the course repo. These guides are structured similar to assignment but have no submissions. This are resources for your team to use in building out your system.  

1. [Guide 1: Grafana](https://scp756-221.github.io/course-site//#/g1-graf/page?embedded=true&hidegitlink=true)
    Grafana as a dashboard for monitoring your system.
2. [Guide 2: Prometheus](https://scp756-221.github.io/course-site//#/g2-prom/page?embedded=true&hidegitlink=true)
    Promethesus as a metrics gathering tool for your system.
3. [Guide 3: Service Mesh & istio](https://scp756-221.github.io/course-site//#/g3-mesh/page?embedded=true&hidegitlink=true)
    istio and Kiali for visualizing and reasoning about your system.

### Mandatory Elements

Your application must:

1. Run within a managed Kubernetes cluster in the public cloud (e.g., EKS, AKS, GKE, or equivalent)
1. Follow a microservices architecture and serve up a number of REST APIs
1. Persists its data to DynamoDB. You may use additional data stores (e.g, AWS RDS, Kafka, etc) if so inclined but this is not required.
1. Create traffic load using Gatling. 

The choices above were made in consideration of the technology/pattern's prevalence, track-record in the market-place and ease of 
use/learning. This in turn translates into community know-how, reliability and easy/available tooling. **Your team should spend no 
more than half your time/effort development.**
However, if you have prior experience implementing REST API using an alternate language/library, you can certainly do so.


## 3. Run Your Application and Document its Behaviour.

Guide 2 uses [Gatling](https://gatling.io/) as a tool for creating a synthetic load on the application. Visit [Gatling Academy](https://gatling.io/academy/) and sign-up. Work through [Module 1](https://academy.gatling.io/courses/Run-your-first-tests-with-Gatling) to learn how Gatling works and how to build/use a simulation. (Gatling uses the term _simulation_ to refer to a load that you program.)
The guide introduces a containerized Gatling container that is suitable for reuse. 

You must run your application with up to a load of thousands of users (e.g, >1000 users) and hundreds of thousands requests (e.g., >100,00 requests/second).

## 4. Write a Report summarizing your findings.

Your team's final report is a coherent story that summarizes the observations and learnings
through the process of developing and analyzing your distributed system.

1. Working in a team environment following the scrum methodology
2. Using a distributed source control system for collaboration
3. Developing for the cloud environment
5. Developing microservices using a highly decoupled design
6. Observing a system at load using a variety of tools
7. Exploring the failure modes of a system
8. Exploring various approaches to remediate such failures

Use this outline to guide your thinking and presentation. Remember, the reports is
to summarize your observations and learnings. Do not merely drop in diagrams and
charts; narrate and explain your line of thinking.

Keep your report to 20 pages max. 

It needs to cover minimally:

1. Reflection on the scrum methodology: 
    i. What did you observe from applying the scrum methodology? 
    ii. What worked well? What didn’t? What surprised you? 
    iii. If you have other/professional experience with scrum, how did your team performed in comparison to past teams?
2. Observations of the system while:
    i. operating at a stable load;
    ii. responding to increasing loads;
    iii. scaling (either manual or automatic) to the changing load;

Optional topics that you may include:
1. Exploration of service mesh features:
    i. Retry/timeout
    ii. Circuit breaker
    iii. Rolling deployment
    iv. Blue/green deployment
2. Other ideas of your own. 

## 5. Record a Video guide of the system.

The video is to provide insight into the practical aspects of your
system and the team that built it. The video is also the occasion to demonstrate each team member's contribution
to the project via their participation. Note that this is *in addition*
to the material inside Github (e.g., issues & commits).

Note that this recording is *not* to be an elaborate production. **Do not spend time
on music, elaborate visuals, or post-production work.** It suffices to make a recording
in Zoom while walking thru a script. 

Your script **must** contain minimally the following. (Please have each team member
take on at least one item.)

1. A tour of the team's Github repo. Narrate the structure and  content of the repo.

2. A walk thru of a couple of issues/branches/PR to illustrate the collaboration between
team members.

3. A deployment (e.g., a ``make -f k8s.mak provision`` or equivalent) of your system into an empty cluster. You may want to start up
a cluster prior to the meeting/recording to reduce waiting within the recording. Take care that screen sharing is setup and that the terminal window is legible.

4. A run of the load simulation on your system with no disturbance. As this run is long, you do not need to
have it complete. About 5 minutes of run-time will suffice during which a narrator
can point out any observations. If you have summary findings (e.g., Grafana summary reports)
to present, collect those from a separate run.

5. A second run of the load simulation on your system during which there is some disturbance
to the system: a change in load, a network fault (delays), a
machine failure, introduction of a circuit breaker, etc. Include your follow-up actions to either
remediate or to understand the behaviour of the system.

The purpose of the video is to showcase the various aspects of the team's work. Assume that the audience viewing your video does not know the specifics of this course; they are encountering your project for the first time. However, you can assume that they are sufficiently familiar/technical with the basic components (e.g., GitHub, cloud, REST API, micro-services). After viewing your video, the audience should know where to look in your repo for relevant components, the run-time environment of the system, the approach the team took to developing/testing/observing the behaviour of the system at load and the behaviour that you discovered.


## Submission

Throughout the remainer of the term, your team will be making steady progress. Your team will be graded on the one interim milestone (5%) and the final submission (25%). **Marks will be assigned individually for the final submission.**

| Milestone | Deliverable(s)/Submission |
| - | - |
| Interim Milestone | [interim report](https://coursys.sfu.ca/2022sp-cmpt-756-g1/+tprjm/) |
| Project Completion | [final report & project guide video](https://coursys.sfu.ca/2022sp-cmpt-756-g1/+tprj/); code (automatic) |


(The code for your application is automatically maintained and collected by GitHub Education.)

### Report

Create a document in a cloud of your choice (Google Doc, OneDrive, etc). As you will work on this project in a team, you will find 
it less confusing to maintain one team copy of the submission report. (E.g., One team member owns the document and shares it 
with the other team members.) I will be using 
any and all available material (e.g., commit history, issue history, etc) at hand to determine that the work was spread evenly across 
the team. In the ideal situation where this is the case, all members of the team should receive the same score for the submission.
If this is not the case, offsetting awards/penalty will be made to reflect the effort of specific team members.  

#### Interim Milestone

The Interim Milestone is a progress check that your team is on track. Submit a draft of your report with at least a skeletal outline and whatever content has been produced. (5%)

#### Project Completion

At Project Completion, submit the final copy of your report with all sections fully fleshed out. 

### Video

Your [SFU Zoom account](https://sfu.zoom.us/) now includes a Record feature. **Every team member must participate (including speaking) in this meeting.** 

Download the recording and post it to YouTube as [an unlisted video](https://support.google.com/youtube/answer/157177). (This will ensure the video remains private except for whoever has its URL.)

A good video runs between 20-30 minutes.

## Grading

Your team's project submission (report, video & code) will be graded on:

1. Presentation (report & video)
   Clarity and format of material
   Participation of entire team

2. Application of agile principles/scrum methodology (video & code)
   Evidence of application
   Learnings from the process

3. System Implementation & Operation (report & code)
   Organization of artifacts
   Maintainability of artifacts
   Appropriate use of technologies
   Learnings

4. Technology (code)
   Use of taught technologies
   Inclusion of new/novel technologies and/or techniques
