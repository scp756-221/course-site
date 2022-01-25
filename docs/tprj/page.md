## Introduction

The term project provides an opportunity for a small group to explore the ideas, apply/observe the principles of the course while gaining hands-on experience through building and observation of a working distributed application. This project for CMPT 756 requires a small team (5 members; exceptions subject to approval) working over the course of 9 weeks.

The project comprises 4 stages:

1. Propose an application in a domain of your choice and define the interaction/API.
2. Implement the API using the supplied reference architecture.
3. Run your application with a load and analyse its behaviour.
4. Write up a report summarizing your findings.
5. Walk through and describe your application in a video recording.

On completing this assignment, you will connect the various ideas from the course together including:

1. Scrum methodology for a distributed team
2. Infrastructure as code
2. Containerized application
4. The micro-services distributed system architecture pattern (REST)
5. Test coverage and load testing of your system.


Key dates:

| Date | Offset | From Project Start | Note |
|-|-|-|-|
| Jan 12, 2021 (Tue) | -- | First day of class. |
| Jan 22, 2021 (Fri) | --  | Class mixer |
| Feb 10, 2021 (Wed) | -- | Project team confirmed. |
| Feb 12, 2021 (Fri) | -- | Project announced. |
| Mar 13, 2021 (Sat) | 4 weeks | First milestone |
| Mar 27, 2021 (Sat) | 6 weeks | Second milestone |
| Apr 13, 2021 (Tue) | --  | Last day of class. |
| Apr 16, 2020 (Fri) | 8 weeks | Project completion. Final submission. |


## 0. Set up

Each project team is named according to letter of the alphabet. Refer to [Canvas](https://canvas.sfu.ca/courses/59479/groups#tab-26517) for your team's name.

Even though this is a group project, each member of the project team must navigate to the [Github Education classroom](https://classroom.github.com/g/L2FU5On3) to accept this assignment. The first member of each team to accept this assignment will also need to create and name the team. **Please name the team according to the name as it appears in Canvas (e.g., Team ?). This will keep the collation consistent.**

The repo from this Github Education classroom contains a standardized directory tree for use in organizing your project. There is very minimal content here; it's setup here mainly for standardized structure and access.

Even as the term project is a team submission, marks will be assigned individually according to documented contribution to the project. Therefore, document your contributions via appropriate means (e.g., commit comments, issues assignments, etc). Commit early and often!

Practice what you've learn to date:
1. use branches appropriately (perhaps to isolate each team member's contributions);
1. issues to maintain a backlog of ideas/stories/tasks; and
1. boards to manage your team's activities.

## 1. Propose an application within some domain

Your application is to be a distributed system for the operation of an organization.
This organization needs only be hypothetical though it should represent a reasonable domain.
For example, a ride-sharing company (or a similar business in the so-called sharing economy) is a valid choice. Something conventional such as a bank or insurance company is also valid.

Because this course is focused on the system aspect of a distributed system, your term project's focus will be similarly focused. Specifically, there is insufficient time to program any of the (albeit interesting) data science techniques into your application. Continuing with the example above, you are to skip implementing the rider/car matching algorithm. Or the GPS feed of the driver's location to the rider. These data processing/handling need are important but are overlays onto the distributed nature of the application. (I will leave them to other courses and/or your future employers.)

Your application must be stateful: user (or a proxy) and session/login are a minimum. You can go as far as desired based on your chosen domain, creativity and effort.

You can certainly model this application on a problem/system from your professional experience. However, do not replicate any proprietary schema or techniques in your project. Within the constraints of the remainder of the term (~eight weeks), the application need/can not be too sophisticated.

Your application must feature at least 3 types of entities (e.g., user, widget, gizmo) and 5 operations that relate 2 or 3 of them (e.g., operate-on-widget-with-gizmo, pair-widget-with-gizmo, etc).

Each such operation (a public call) needs to make use of additional underlying calls (e.g., read-from-database, validate-user-has-funds, etc). Finally, your application must use DynamoDB for storing its data.

## 2. Implement your application using exercise material supplied to date

To accelerate the development of your team's application, you will be using templates and material that you have accumulated over the exercises to date and yet to come. The following attributes are mandatory:

### Technologies
1. To be operated on AWS (personal account), Azure or GCP
1. Within a managed Kubernetes cluster (EKS, AKS or GKE)
1. With istio as a service mesh
1. Following a Microservices architecture
1. Serving up a number of REST API
1. Validated/tested/analysed with traffic load generated by Gatling.io
1. Persisting data to DynamoDB

The choices above were made in consideration of the technology/pattern's prevalence, track-record in the market-place and ease of use/learning. This in turn translates into community know-how, reliability and easy/available tooling. **Your team should spend no more than ½ the project time/effort on development.**

I highly encourage you to implement your REST API using Python and Flask especially if this is your time. However, if you have prior experience implementing REST API using an alternate language/library, you may make a request to the instructor for approval to use an alternative.


The remaining ½ of your time is devoted to operating, measurement and analyzing the produced system. Note that this does not mean testing and debugging! You will need the time to work with and analyze your system and (most importantly) write it up.

## 3. Run your application with a load and document its behaviour.

Exercise 5 will introduce you to Gatling. In the interim, visit [Gatling Academy](https://gatling.io/academy/) and sign-up. Work through [Module 1](https://academy.gatling.io/courses/Run-your-first-tests-with-Gatling) to learn how Gatling works and how to build/use a simulation.

You should use the containerized Gatling setup inside the exercise to run your simulation. But it will be more convenient to develop your simulations using a local install of Gatling. (Recall that you have previously installed Gatling as part of Exercise 1.)

Build a _coverage simulation_ that exercises your entire API. This first simulation need to exercise every public API call that you've implemented. Consider this the equivalent of every API target that was implemented in `api.mak`: `cuser`, `uuser`, `duser`, `apilogin`, `apilogoff`, `cmusic`, `rmusic`, & `dmusic`. Note that private APIs (e.g., `db`'s  `read`, `write`, `update`, `delete`) are not needed in this coverage simulation. Run, save and submit a run of this simulation.

Create a second _load simulation_ (based on the first) that simulates a number of users interacting with your system. For your load simulation, use the _closed model_. This is not as accurate for web applications (compared to the open model) but it is easy to understand. Start with [`constantConcurrentUsers(50)`](https://gatling.io/docs/current/general/simulation_setup/#closed-model) for a run of at least 30 minutes. (You may wish to experiment with `rampUsersPerSec` in the _open model_ but that is not required for the term project.) Again, run, save and submit the test run.

## 4. Write up a report summarizing your findings.

Your team's final report is a coherent story that summarizes the observations and learnings
through the process of developing and analyzing your distributed system. The term project's
goal (and by extension, the course's purpose) is to give your team the basis--a distributed system together with
an amount of tooling--to explore and practice the various ideas:

1. Working in a team environment following the scrum methodology
2. Using a distributed source control system to collaborate
3. Developing for the cloud environment
5. Developing microservices following a highly decoupled design
6. Observing a system using a variety of tools
7. Exploring failure modes of a system
8. Exploring various approaches to remediate such failures

Use this outline to guide your thinking and presentation. Remember, the reports is
to summarize your observations and learnings. Do not merely drop in diagrams and
charts from your work; narrate and explain your line of thinking.


There is no limit on the length of the report but it need to cover minimally:

1. Problem domain: What concrete problem is your application solving?

1. Reflection on the scrum methodology: What did you observe from applying and using the scrum methodology? What worked well? What didn’t? What surprised you? If you have professional experience with scrum, how did your team performed in comparison to past teams?

1. Results of testing with Gatling: How did the coverage simulation help with your development? Where and when did you used it and what was the outcome for the application? The development process? The team?

1. How did the load simulation help with testing of your completed system? What types of failures did you simulate and what were the outcomes? How did your application respond to various disturbances to the network?


## 5. Record a short video guide of the system.

The intent of the video is to provide insight into the practical aspects of your
system and the team that built it. I will also evaluate each team member's contribution
to the project via their participation in this meeting. Note that this is *in addition*
to the material inside Github (e.g., issues & commits).

Note that this recording is *not* to be an elaborate production. Please do not spend time
on music, elaborate visuals, or post-production work. It suffices to make a recording
with Zoom while walking thru a script.

Your script should contain minimally the following. (Please have each team member
take on at least one item.)

1. A tour of the code inside Github. Narrate the structure of the repo and point out how
your team used the material/structure from the exercise to arrive at the repo for the team.
2. A walk thru of a couple of issues/branches/PR to illustrate the collaboration between
team members.
3. A deployment (e.g., a ``make -f k8s.mak provision`` or equivalent) of your system into an empty cluster. You may want to start up
a cluster prior to the meeting/recording to reduce waiting within the recording. Take
care that screen sharing is setup and that the terminal window is legible.
4. A run of the coverage simulation on your system. This is a short run which can complete
over the course of the recording. Present any findings here.
5. A run of the load simulation on your system with no disturbance. As this run is long, you do not need to
have it complete. About 5 minutes of run-time will suffice during which a narrator
can point out any observations. If you have summary findings (e.g., Grafana summary reports)
to present, collect those from a separate run.
6. A second run of the load simulation on your system during which you will disturb
the system via any of the means I've outlined previously (e.g., injecting a delay, simulating a
machine failure, inserting a circuit breaker, etc). Include your follow-up actions to either
remediate or to understand the behaviour of the system.

You can of course add additional items. The purpose of the video is to showcase the various aspects of the team's work. Assume that the audience viewing your video does not know the specifics of this course; they are encountering your project for the first time. However, you can assume that they are sufficiently familiar/technical with the basic components (e.g., GitHub, cloud, REST API, micro-services). After viewing your video, the audience would know where to look in your repo for relevant components, the run-time environment of the system (in a cloud of your choosing), the approach the team took to testing/observing the behaviour of the system at load and the behaviour that you discovered.

Note that your [SFU Zoom account](https://sfu.zoom.us/) now includes a Record feature to record your meetings. (Turn off your laptop's camera but record the screen sharing and your narration.) **Every team member** must participate (including speaking) in this meeting.

You should aim for a meeting/recording of about 20-30 minutes in length.

## Submission

Throughout the eight weeks of the project, your team will be making steady progress and making interim submissions. Your team will be graded on two milestones and the final submission. **Marks will be assigned individually for the final submission.**

| Milestone | Deliverable(s)/Submission |
| - | - |
| Milestone 1 | [report draft 1](https://canvas.sfu.ca/courses/59479/assignments/576403) |
| Milestone 2 | [report draft 2](https://canvas.sfu.ca/courses/59479/assignments/576404) |
| Project Completion | [final report (individual submission)](https://canvas.sfu.ca/courses/59479/assignments/576656); code (automatic); [Zoom recording (team submission)](https://canvas.sfu.ca/courses/59479/assignments/576406) |

### Create a PDF

Make a copy of the [submission template](https://docs.google.com/document/d/1xYzX3sta87WntJHNqHvbpE5lGvgblkvtiWblX4wRXJw/edit?usp=sharing)(GDoc format).

As you will work on this project in a team, you will find it less confusing to maintain one team copy of the submission report. (E.g., One team member makes the copy and share this copy with the other team members.) However, each team member will be making individual submission for the term project. I will be using any and all available material (e.g., the commit history, issue history, etc) at hand to determine that the work was spread even across the team. In the ideal situation where this is the case, all members of the team will receive the same score for the submission. If this is not the case, offsetting awards/penalty will be made to reflect the effort of specific team members.  

Generate a PDF of this document.

You must name your PDF (or ZIP) according to the pattern: **SFU-student-no**`-tprj-submission.pdf` (or **SFU-student-no**`-tprj-submission.zip`) where **SFU-student-no** is the  numeric id (typically 30...) assigned to you upon entering SFU.

**A penalty will be assessed for failure to name your submission appropriately.**


1. Milestone 1 is a progress check that your team is on track. Submit a draft of the report (PDF) with section 1 (Problem Statement), 2 (Github Repo Guide) and 3 (Reflection on Development) roughed in. These section need to be sufficiently complete to be useful for the reader (though not finalized). (5%)

2. Milestone 2 is a completeness check: Your application will be functional and stable. Submit another draft of the report (PDF) with section 4 (Analysis) _outlining_ the approach and preliminary results of your Gatling tests. (5%)

3. The final project submission will comprise three components:
  a. the code (automatically collected via  Github Education classroom repo);

  b. the final report with all sections completed and two Gatling simulation runs (zip);
  (Gather the PDF and HTMLs and compressed into one zip file for submission.)

  c. a Zoom recording (at most 30 minutes) (URL submission).
  (You can upload the video to your personal YouTube account as an **unlisted** video. This will ensure the video remains private except for whoever you share its URL.)
