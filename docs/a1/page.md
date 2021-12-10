## Introduction

This assignment comprises two parts. First, you will set up a
development environment on your Windows, macOS or Linux
laptop/desktop. This will require the installation of several large
software products, which will take some time. However, this is the
only assignment that requires large installations and the tools you
install this week will be used throughout the course.

Second, after completing the installations, you will use the installed
software to run and modify a simple application.

**This assignment has two prerequisites:**

**1. You must have set up a GitHub userid and joined GitHub
Education---see the instructions for this in the first half of
Assignment&nbsp;0.**

**2. You will need access to a high-bandwidth Internet connection for
rapid download of the large installation files.**

## Part 1: Install tools

You will need to setup the appropriate tools on the machine you plan
to use for this course's assignments and term project. The heavy
lifting will be typically on the cloud side (AWS or other services)
but you will need a certain amount of tooling on your computer to
operate the cloud.

**If you have less than 8GB of RAM on your home machine**, please use the MPCS
lab in Blusson Hall for the course exercises. Contact the instructor
for instructions.

### Visual Studio Code (VSC)

We will standardize on
[Visual Studio Code](https://code.visualstudio.com/) (VSC) for this
semester. Install the version for your operating system.

A good text editor raises your productivity significantly with syntax
highlighting, syntax validation/hinting, support for multiple
files/source trees, integration with version control (Git) and
numerous other creature comforts. **Do not skip this step.**

### Git

Install Git per
[the Git community's instructions](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).
In many environments (e.g., macOS), Git comes pre-installed; check.

Install [GitHub Desktop](https://desktop.github.com/) as well to
complement the command-line Git.

We have
[a course Git summary](https://canvas.sfu.ca/files/13170454/download?download_frd=1).
(Additional tutorials at
[DZone Git reference card](https://dzone.com/refcardz/getting-started-git)
and
[Matt Harrison's supervisual Git cheatsheet](https://github.com/mattharrison/Git-Supervisual-Cheatsheet/blob/master/gitcheat.png).)

### Docker

Install Docker Desktop (for macOS and Windows) or Docker Engine (for Linux) from
[docker.com](https://docs.docker.com/get-docker/).

### The course code repository

**You must have successfully installed Git before starting this
step.**

**You must have a GitHub userid (see the first part of
Assignment&nbsp;0) before starting this step.**

All code for the course is in a single code repository, available as a
template on GitHub.

1. Sign in to GitHub using your userid.
2. Go to the
   [`scp756-221/c756-exer` page](https://github.com/scp756-221/c756-exer).

3. Click the `Use this template` button.

4. Fill in the dialogue fields:

   * **Repository name: `c756-exer`**
   * Description: CMPT 756 course code
   * Choose "Private" rather than "Public"
   * Do not check "Include all branches"

5. Click `Create repository from template`.

### The course tools container

**You must have successfully installed Docker before starting this
step.**

The repository that you just cloned contains the sample and starter
code for the course assignments. In addition to this code, you
also need the *course tools*, a Linux command line with all the software
required for this course. We have packaged these tools as a container.
You will explore containers and Docker in detail in Assignment&nbsp;3.
For Assignments&nbsp;1 and 2, you will just call Docker from scripts.

You do not explicitly download or install a container. Instead, you
run it and if the container *image* (we will clarify this term in
Assignment&nbsp;3) is not yet downloaded to your machine, Docker will
first download it. Once the image has been downloaded, Docker will
retain it in a cache and you will not need to download it the next
time you run it.

#### Running the course tools container

The tools container is a Linux command-line with a suite of Linux
commands. Regardless of your machine's operating system,
most of the commands and scripts you run in this course will be run in
a Linux environment overlaid on your own operating system.

We can summarize these different layers in a table:

<table>
<caption>Layers when running the tools container</caption>
<thead>
<th>Layer Name</th><th>Location</th><th>OS</th><th>Prompt</th>
<thead>
<tbody>
<tr>
<td>Guest OS</td><td>Container</td><td>Linux</td><td><code>.../home/k8s...#</code></td>
</tr>
<tr>
<td>Host OS</td><td>Actual machine</td><td>Windows/macOS/Linux/...</td><td><code>... $</code></td>
</tr>
</tbody>
</table>

The `...` in the prompts indicates other information that may appear
in the prompt prefix.  When we write sample code for you to run in the
shell, we will simply indicate which operating system to run them in
by the appropriate suffix: `/home/k8s#` indicates the Guest OS, while
`$` indicates the Host OS.

We'll get into more details of the above structure in
Assignment&nbsp;3 but for now this is enough to start.

A final, crucial point about running the course tools container: **The
current directory must always be the `c756-exer/e-k8s` subdirectory of
the course code repository when you run it.**

You start the container by running `tools/shell.sh`:

(Note: As this is the first time you will have run the container, its image
will have to be downloaded. This will be indicated by a message that
Docker could not find the image locally. As the image is over 2 GiB, it can
take several minutes to download and expand. This will not happen with
later invocations, as the image will already be downloaded and remain
cached on your machine.)

~~~bash
# Use whatever path you need to get to this directory
$ cd c756-exer/e-k8s
$ tools/shell.sh
...
Unable to find image 'ghcr.io/scp756-221/c756-tool:...' locally
...
/home/k8s# 
~~~

The `...` sequences in the above indicate passages that will vary with
your execution.

If the tools container starts up completely, exit it and return to
the command line of the Host OS:

~~~bash
/home/k8s# exit
$
~~~

There are two exceptions, tools that you run in your Host OS rather
than from inside the tools container. Both work with source code:

* Visual Studio Code, which you will use to edit source code and YAML
  files.
* Git, which you will use to commit source code changes to version control.


### Entering your template parameters

Some of the files in the course repository `c756-exer` are templates,
generic code that will be instantiated with values specific to youre
use.  All the files controlling this process are in the directory
`c756-exer/e-k8s/cluster`. In that directory, we have provided a file
named `tpl-vars-blank.txt`.

1. In your Host OS, make a copy of `tpl-vars-blank.txt` named
   `tpl-vars.txt`, in the same directory.
2. In Visual Studio Code, edit `tpl-vars.txt`:

   * On the line starting `ZZ-REG-ID=`, append your GitHub userid. For
     example, if your userid were `div_by_0`, the line would become
	 `ZZ-REG-ID=div_by_0`. Note that there are **no spaces around the
	 `=` sign**.
   * If you have completed your AWS signon, enter your IAM
     administrative user's access key ID and key value in the
	 appropriate lines. If you have not completed your AWS signon yet,
	 you can do that next week.
    BLERG BUT NOT IF WE USE `aws-cred.sh`.
3. Save the file.

Next you need to instantiate every template file.  In the tools
container, run this command:

~~~bash
/home/k8s# make -f k8s-tpl.mak templates
tools/process-templates.sh
/home/k8s#
~~~

**SECURITY NOTE:** (This really should be in blinking red text.) The
`cluster` directory and all its files should be treated with extreme
care:

* Do not copy any of its files outside the directory.
* The directory should be only readable by you.
* Do not remove any lines from its `.gitignore` file.
* Do not create any other files in this directory.
* Do not display any of these files on your terminal using programs such
  as `cat`. If you want to review their contents, do so in an editor
  such as Visual Studio Code or `vi` (available in the tools
  container).

**Your AWS access key is essentially your credit card number---without
requiring a date or CVV security code.  Treat any file containing it
as you would a file with your credit card data in plain text.**

### GitHub access token

[Create a personal access token (PAT)](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token)
for your GitHub account. You will need the three scopes:
`read:packages`, `write:packages` and `delete:packages`. For increased
security, we recommend setting a token expiration date so that the
token becomes unusable a few weeks after this course ends.

Save your token in the file
`c756-exer/e-k8s/cluster/ghcr.io-token.txt`. (To repeat: The `cluster`
directory is where we place all secure files.)

### Make `cluster` only accessible to you

For security, ensure that the `cluster` directory's
permissions prohibit anyone else from accessing its files.

In the tools container:

~~~bash
/home/k8s# chmod go-rwx cluster
/home/k8s# chmod go-rwx cluster/*
~~~

### End of installation---possible break time

This concludes the installation part of this assignment. You are
welcome to take a break before starting Part&nbsp;2. The second part
will not make heavy use of the Internet, so you do not need
high-bandwidth access.

## Part 2: Running a simple application

In this second part, you will run and modify a simple client-server
application.  The coding you do will be simple; the point of this
exercise is to gain familiarity with the tools and applications that
you will be using for the rest of the course.

### The music client-server application

Cloud-based systems, a main topic of this course, make heavy use of
client-server designs. In this assignment, we will look at the
simplest possible form of such a design, with both components running
on your computer. In future assignments, we will move the server
component further away, until in Assignment&nbsp;4 it will run in
the cloud, on AWS under Kubernetes.

The foundation of client/server designs is a separation of concerns.
The server provides a *service*, accessed by a standard Application
Programming Interface (API). The client provides a more intelligible
interface, customized to the needs of the user:

<table>
<caption>Structure of a simple client-server application</caption>
<thead>
<th>Component</th><th>Code</th><th>Location</th><th>Persistent state?</th><th>Instances</th>
<thead>
<tbody>
<tr>
<td>Client</td><td>Simple</td><td>Local</td><td>No</td><td>Multiple</td>
</tr>
<tr>
<td>Server</td><td>Complex</td><td>Remote</td><td>Yes</td><td>Single</td>
</tr>
</tbody>
</table>

The above table describes this simple first application. In future
lectures and assignments, we will consider much more complicated
combinations. In particular, a *server* offering a higher-level
service will often also be a *client* calling another server that
provides a lower-level service.

For now though, the above table describes our simple system.

### The simple music service

For the first few assignments, the client and server will implement a
simple music service, listing songs and the artists that perform them.
The actual music will not be stored, only the metadata. Every song
will be identified by a system-defined universal ID such as
`6ecfafd0-8a35-4af6-a9e2-cbd79b3abeea`, designed to have a very high
probability of being unique. The probability of duplication is so low
in fact that we shall ignore it and assume these IDs are genuinely
unique.

(You may be wondering, "Why not generate IDs *guaranteed* to be
unique? Why resort to all this probabilistic stuff?" The short answer
is that although this would be easy for the simple service in this
assignment, guaranteeing uniqueness can incur severe inefficiencies in
high-volume cloud systems. The details will be covered in later
lectures.)

The music service supports three basic operations: create, read, and
delete. The system does *not* support update. In typical use, the
service will simply accumulate song/artists as new music is released.

The server accepts requests as specially-formatted HTTP requests.
These are awkward for human users, so we provide a client implementing
a simple command-line query interface.

### Running the simple music service

Open up two command-line windows.  In each one, start the tools
container, remembering that first you have to make `c756-exer/e-k8s`
the current directory.

In one window, build and run the server:

~~~bash
/home/e-k8s# cd s2/standalone
/home/e-k8s/s2/standalone# ./builda1.sh
... build messages ...
/home/e-k8s/s2/standalone# ./runa1.sh
... server log messages ...
~~~

In the other window, build and run the client:

~~~bash
/home/e-k8s# cd mcli
/home/e-k8s/mcli# make build-mcli
... build messages ...
/home/e-k8s/mcli# make run-mcli
docker container run ...

Command-line interface to music service.
Enter 'help' for command list.
'Tab' character autocompletes commands.

mql: 
~~~

Explore issuing commands via the client.  Start with `read` to list
all available songs, then create and delete some songs. Use `help` for
more details.

Watch the output from the server. This is the server's *log*. For
every request you issue from the client, the server log will display a
line such as

~~~
172.17.0.6 - - [03/Dec/2021 00:57:16] "GET /api/v1/music/ HTTP/1.1" 200 -
172.17.0.6 - - [03/Dec/2021 00:57:37] "POST /api/v1/music/ HTTP/1.1" 200 -
~~~

recording (left to right) each operation's

* client IP address
* date
* time
* HTTP operation (GET/POST/DELETE)
* path
* protocol
* status code

We typically are only interested in the operation, path, and status code.

At some point, run the `test` command.  What do you see in both
windows? We will return to this result in the next subsection. For
now, just note this behaviour.

Once you are comfortable with the service exit the client via the
`quit` command and the server via `CTRL-C`. Leave the windows
open---you'll be using them again in a moment.

A note about this version of the music service: Its database is not
truly persistent. Any changes you make will be thrown away when you
exit the server. In Assignment&nbsp;4 you will begin using a full
version of the service that stores the songs persistently on AWS
DynamoDB.

### Fixing a defect in a cloud service is hard

As an experienced programmer, you've done the typical defect-removal
cycle many times:

1. Note that a system's behaviour contradicts its specification.
2. Look at system diagnostics (from system logs, debug print statements, debugging
   tools, and other sources) to isolate the source of the defect.
3. Revise the source code to remove the defect.
4. Build and run the revised system, verifying that the defect has
   been removed (and no new defects introduced).
5. Commit the revision to version control, using a tool such as Git.

This cycle still applies to cloud systems with microservices
architecture, with an important difference: Getting the system
diagnostics (Step&nbsp;2) and running the revised system (Step&nbsp;4)
are tricky. Cloud services are more difficult to debug because they
are running on remote machines, surrounded by security barriers
designed to limit access.

In this exercise and the three that follow, we're going to give you
practice debugging cloud systems. This exercise begins with the
simplest case, debugging a server running on your local machine. By
Assignment&nbsp;4, we'll have moved up to debugging a server running
remotely on AWS.

In each of these exercises, the service will have the same "bug", that
the `test` command throws an exception in the server and returns a
`500` status code, indicating
["Internal Server Error"](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).
Removing the exception and returning a `200` status code (indicating
["OK"](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)) will
be trivial and require no knowledge of Python or even of how the
server works. **The point of this part of the assignment is to
practice reading a service's log output and restarting the fixed
service.** The trivial "bug" is just an excuse to practice this cycle.

### Fixing a defect in a local service (the easy case)

For this assignment, we'll start with the easiest case. The server log
is just the output in its terminal window. The log shows the stack
traceback from the exception, as well as other information. In
particular, the first line of server output will have something like

~~~
[2021-12-03 01:17:29,510] ERROR in app: Unique code: c69314133f0dfb2a79c93278335d6c10ed60498c20b03d49168db2344d579d89
~~~

Ignore the `ERROR` tag---this is just an informational message. The
unique code you see on the right will differ from the one above and it will
vary with each assignment.

To fix the "bug" this time, you copy the code (the 64 hex characters,
such as `c69314...579d89` in the above line), open the
file`c756-exer/e-k8s/s2/standalone/app.py` in Visual Studio Code, and
paste it between the single quotes in the `if` statement of the
`test()` function, replacing whatever is between them:

Before (the 64-character code may differ from this or even just be `''`):

~~~python
@bp.route('/test', methods=['GET'])
def test():
    if '1e0715252b48ed14858ae1ce646d67195183ffb8f9dc02d73c82323d8d75f482' != ucode:
        raise Exception("Test failed")
    return {}
~~~

After (but use **your code**, not this one):

~~~python
@bp.route('/test', methods=['GET'])
def test():
    if 'c69314133f0dfb2a79c93278335d6c10ed60498c20b03d49168db2344d579d89' != ucode:
        raise Exception("Test failed")
    return {}
~~~

### Testing the fix

Save the revised file and (in the tools container) rebuild and rerun
the server in the server window:

~~~bash
/home/e-k8s/s2/standalone# ./builda1.sh
... build messages ...
/home/e-k8s/s2/standalone# ./runa1.sh
... server log messages ...
~~~

The client didn't change, so you don't have to rebuild it, only rerun
it in its window:

~~~bash
/home/e-k8s/mcli# make run-mcli
docker container run ...

Command-line interface to music service.
Enter 'help' for command list.
'Tab' character autocompletes commands.

mql: test
mql: 
~~~

This time, the server log will simply show a successful call (`200` status),

~~~
172.17.0.6 - - [03/Dec/2021 17:49:20] "GET /api/v1/music/test HTTP/1.1" 200 -
~~~

and the client will have no output, indicating a successful operation.

Congratulations!  You've fixed the bug ... this time.

The process was trivial in this case but consider: What might be
required to read the log from a server running on the other side of
the world, packaged and isolated in a container, surrounded by
multiple layers of security preventing unauthorized access? And what
might be required to get the fixed software running on that remote
machine?

### Committing and pushing the fix

You've found the defect and tested the fix but one more step remains:
Committing the fix to version control to make it permanent and visible
to other team members.

In the next assignment, you will go into the details of Git. For now,
just execute the following commands. Recall that Git, along with
Visual Studio Code, is the only tool that you run in your
regular *Host OS*:

~~~bash
$ cd c756-exer/e-k8s/s2/standalone
c756-exer/e-k8s/s2/standalone $ git add app.py
c756-exer/e-k8s/s2/standalone $ git commit -m 'Add missing code to "test"'
c756-exer/e-k8s/s2/standalone $ git push origin
~~~

### Review of this step

The coding part of this assignment was simple enough: Beginning with a
basic client-server design, find and fix a defect in the server. In
coming exercises, we will repeat this cycle of read
logs-fix-rebuild-test-commit.  As the server becomes more remote and
more protected, the "read logs" and "rebuild and test" steps will
require more tooling.

## Submission

BLERG what and where?

BLERG Include CI result from own page.

### Create a PDF

Make a copy of the [submission template](https://docs.google.com/document/d/10kVl3R5tMVgxqdzO0QAXcRRxxRcWcGcdQnlPqvBT6WE/edit?usp=sharing)(GDoc format).

Fill in:

a. The header box at the top of the document.

b. Content from the steps above.

Generate a PDF when you are done.

You must name your PDF according to the pattern: **SFU-id**`-e1-submission.pdf` where **SFU-id** is the portion of your email address preceding `@sfu.ca`. Unfortunately, you will be penalized for incorrect filename because of cascading dependencies for a large class.

**A penalty will be assessed for failure to name your submission appropriately.**

### Canvas submission

Navigate to this assignment and upload the generated PDF.
