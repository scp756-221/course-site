# Assignment 1&mdash;Toolset & tool container; Local client/server

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

You will need to setup the appropriate tools on your own laptop for 
this course's assignments and term project. The heavy
lifting will be typically on the cloud side (AWS or other services)
but you will nonetheless need a certain amount of tooling on your computer to
operate the cloud.

**If you have less than 8GB of RAM on your own machine**, you should use the MPCS
lab instead (while accepting some degradation). Please inform the instructor
for allowances and/or specific instructions.

### Visual Studio Code (VSC)

We will standardize on
[Visual Studio Code](https://code.visualstudio.com/) (VSC) for this
semester. Install the version for your operating system. You are not required to use it but 
it will be the reference tool.

A good editor/IDE raises your productivity significantly with syntax
highlighting, syntax validation/hinting, support for multiple
files/source trees, integration with version control (Git) and
numerous other creature comforts. **Do not skip this step.**

### Git and GitHub Desktop

Install Git per
[the Git community's instructions](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).
In many environments (e.g., macOS), Git comes pre-installed; check.

Install [GitHub Desktop](https://desktop.github.com/) as well to
complement the command-line Git. Also configure GitHub Desktop's integration (under Preferences) to use your chosen editor.



We have
[a course Git summary](https://canvas.sfu.ca/files/13170454/download?download_frd=1).
(Additional tutorials at
[DZone Git reference card](https://dzone.com/refcardz/getting-started-git)
and
[Matt Harrison's supervisual Git cheatsheet](https://github.com/mattharrison/Git-Supervisual-Cheatsheet/blob/master/gitcheat.png).)

### Docker Desktop/Docker

Install Docker Desktop/Docker from
[docker.com](https://docs.docker.com/get-docker/). For MacOS and Windows, it will be Docker Desktop which includes a convenient dashboard. For Linux, it will be plain Docker.

### Generate course code repo instance

**You must have successfully installed Git before starting this
step.**

**You must have a GitHub userid (see the first part of
Assignment&nbsp;0) before starting this step.**

All code for the course is in a single code repository (a mono-repo) that has been setup as an assignment in the GitHub Education Classroom.

1. Sign in to GitHub using your userid.
2. Sign in to [GitHub Education](https://education.github.com/) using the same userid. 
3. Accept the assignment via this [link](https://classroom.github.com/a/1YFBko6w)
4. Wait while a copy of the repo is generated for you.
5. Bookmark the URL of your copy of the repo http://github.com/scp756-221/assignments-YOUR-GITHUB-ID. As the repo resides in this class' organization, you will not see it listed under "Your repositories". But you will find it at the left-hand margin of your [GitHub home page]](https://github.com) after you sign in.


#### Downloading the course code to your machine

You now have a copy of the assignment repo in your GitHub account.  Next you need to download a copy of it to your machine in an Git operation called *cloning*.

In the Web page for your repository,

1. Click the green `Code` button, producing a dropdown dialogue.
2. Check that the `HTTPS` tab is checked (it is the default).
3. Click the "copy" icon (the overlapping boxes) to copy the URL into your clipboard.
4. In a command line window, `cd` to the parent directory where you will put all your CMPT&nbsp; 756
   materials and type, where `URL` is the pasted value of the URL in your clipboard:

   ~~~bash
   ... $ git clone URL
   ~~~

You now have a copy of the course code repository on your local machine, linked to the copy on your GitHub account.

### Course tools container

**You must have successfully installed Docker before starting this
step.**

The repository that you just cloned contains the sample and starter
code for the course assignments. In addition to this code, you
also need the *course tools*, a Linux command line with all the software
required for this course. We have packaged these tools as a container. (Or more accurately, an image.)
You will explore Docker and containers in detail in Assignment&nbsp;3.
For Assignments&nbsp;1 and 2, you will just call Docker from scripts.

You do not explicitly download or install a container. Rather, Docker automatically
downloads and caches each container you use. Thus, you only need to
reference an image (typically via a pull or run) and the container *image* (we will clarify this term in
Assignment&nbsp;3) will be retrieved as required by Docker.

#### Running the course tools container

**WARNING: for students working on Apple Silicon (e.g., M1/Pro/Max) machines, bear in mind you are working on 
hardware whose [software support is rapidly evolving](https://docs.docker.com/desktop/mac/apple-silicon/). Please inform the teaching team (via email) that you are using Apple Silicon hardware.**

**The tools container was developed in anticipation of the ARM
architecture (as opposed to the x86 architecture of the prior generation) but the ARM-variant of the container is untested as we did not have access to Apple Silicon hardware. Please be patient and work with the teaching team should any problem arise.**


The tools container is a collection of Linux command-line tools in ready-to-use form. It is provided to 
level-set all students to a known good state (e.g., all tools are available and ready-to-use). I recommend
each student to switch to a direct install of the tools into the native OS when one is comfortable with and can work out the specifics required.

To be clear, regardless of your machine's host operating system, you will be working in a Linux environment when you are working inside this container. To differentiate
between your host environment (e.g., MacOS, Windows WSL2 Linux, native Linux) from the container environment (Linux), we will use
the following convention:

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
in the prompt prefix. (This may differ significanlty in the Host OS if you have customized
your environment heavily.)  When we introduce sample commands for you to run in the
shell, we will simply indicate which environment to run them in
by the appropriate suffix: `/home/k8s#` indicates the Guest OS, while
`$` indicates the Host OS.

We'll get into more details of the above structure in
Assignment&nbsp;3 but for now this is enough to start.

A final, crucial point on starting the course tools container: **Always start the container from the `c756-exer/` directory of
the course code repository.**

To start the container, run `tools/shell.sh` as follows:


**Note:** As this is the first time you will have run the container, its image
will be be downloaded. This will be indicated by a message that
Docker could not find the image locally. The image is large (>2&nbsp;GB) and can
take several minutes depending upon the speed of your Internet service. This download is not needed on
subsequent invocations (unless you explicitly purge it), as the image will already be downloaded and remain
cached on your machine.

~~~bash
# Use whatever path you need to get to this directory
$ cd ..../c756-exer
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

### Tools that run on your host operating system 

There are two tools that you will run in your Host OS rather
than from inside the tools container. Both are for handling "source code":

* Visual Studio Code, which you will use to edit code of one form or another: scripts, YAML, etc.
* Git, which you will use to commit code changes to version control.


### Instantiate configuration template

Some files within the course repository `c756-exer` are templates:
generic files that must be filled in with values specific to each student. 
All the files controlling this process are in the directory
`c756-exer/cluster`. Look for the file `tpl-vars-blank.txt` in this directory.

1. In your Host OS, make a copy of `tpl-vars-blank.txt` named
   `tpl-vars.txt`, in the same directory.
2. Start up Visual Studio Code and open this directory.
3. Use the Explorer (upper-left tool on the left-hand-side toolbar) and the browser just to its right to locate `tpl-vars.txt` that you just created.
4. Open and edit it:
   * On the line starting `ZZ-REG-ID=`, append your GitHub userid. For
     example, if your userid were `div_by_0`, the line would become
	 `ZZ-REG-ID=div_by_0`. Note that there are **no spaces around the
	 `=` sign**.
   * If you have completed your AWS signon, enter your IAM
     administrative user's access key ID and secret key in the
	 appropriate lines. If you have not completed your AWS signon yet,
	 you can do that next week. 
3. Save the file.

Next you need to instantiate every template file.  In the tools
container, run this command:

~~~bash
/home/k8s# make -f k8s-tpl.mak templates
tools/process-templates.sh
/home/k8s#
~~~

You also need to commit one file created by the instantiation process. In your Host OS,

~~~bash
$ cd .../c756-exer/s2/standalone
$ git add README.md
$ git commit -m 'Instantiate CI badges'
~~~

where `...` is whatever directory path takes you to `c756-exer`.

**SECURITY NOTE:** (This really should be in blinking red text.) The
`cluster` directory and all its files should be treated with extra
care:

* Do not copy any of its files outside the directory.
* The directory should be only readable by you.
* Do not remove any lines from its `.gitignore` file. This file protects you from inadvertent exposure of various secrets.
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
`c756-exer/cluster/ghcr.io-token.txt`. (To repeat: The `cluster`
directory is where we place all secure files.)

### Secure `cluster`

For security, ensure that the `cluster` directory's
permissions prohibit anyone else from accessing its contents.

In the tools container:

~~~bash
/home/k8s# chmod go-rwx cluster
/home/k8s# chmod go-rwx cluster/*
~~~

### (Optional) Configuring bash and Git aliases

*This section is optional.*

If you have frequently-used bash aliases or Git configuration options that you would like to use in the tools container, you can define them in this environment by editing files `bash_aliases` or `gitconfig` in the subdirectory `profiles`. See `profiles/README.md` for the details.

### Coffee Break 

This concludes the installation part of this assignment. You are
welcome to take a break before starting Part&nbsp;2. The second part
does not use much of the Internet so you do not need
high-bandwidth access.

## Part 2: Run a simple application

In this second part, you will run and modify a simple client-server
application.  The coding you do will be simple; the point of this
assignment is to gain familiarity with the tools and applications that
you will be using for the rest of the course.

### Client-server application

Cloud-based systems, a main topic of this course, make heavy use of
client-server designs. In this assignment, we will look at the
simplest possible form of such a design, where both components are running
on your computer. In future assignments, we will move the server
component further away, until in Assignment&nbsp;4 it will run in
the cloud, on AWS under Kubernetes.

The foundation of client/server designs is a separation of concerns.
The server provides a *service*, accessed by a pre-defined Application
Programming Interface (API). The client in turn provides a more intelligible
interface, tailored to the needs of the client's operator:

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
lectures and assignments (and in real-world scenarios), you will encounter more complicated
combinations. As well, one component within a system may be both
a client (using a lower-level service) and a server (providing some 
service/facility to other component).

For now though, the above table describes our simple system.

### The simple music application

For the first few assignments, the client and server together implement a
simple music application that stores and retrieves songs and the artists who performed them.
(The actual music media file is not stored, only the metadata.) Every song
will be identified by a system-defined universal ID such as
`6ecfafd0-8a35-4af6-a9e2-cbd79b3abeea`, designed to have a very high
probability of being unique. The probability of duplication is so low
in fact that we shall ignore it and assume these IDs are genuinely
unique.

(You may be wondering, "Why not generate IDs *guaranteed* to be
unique? Why resort to all this probabilistic stuff?" The short answer
is that although this would be easy for the simple service in this
assignment, guaranteeing uniqueness can incur severe inefficiencies in
high-volume cloud systems. The details may be covered in later
lectures.)

The music service supports three basic operations: create, read, and
delete. The system does *not* support update. In typical use, the
service will simply accumulate song/artists as new music is released.

The server accepts requests as specially-formatted HTTP requests.
These are awkward for human users, so we provide a client implementing
a simple command-line query interface.

### Running the music application

Open up two terminal windows: one each for the client and the server.  In each one, start the tools
container, remembering that first you have to make `c756-exer/`
the current directory.

In one window, build and run the server:

~~~bash
/home/k8s# cd s2/standalone
/home/k8s/s2/standalone# ./builda1.sh
... build messages ...
/home/k8s/s2/standalone# ./runa1.sh
... server log messages ...
~~~

In the other window, build and run the client:

~~~bash
/home/k8s# cd mcli
/home/k8s/mcli# make build-mcli
... build messages ...
/home/k8s/mcli# make run-mcli
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
* HTTP operation (GET/POST/DELETE/etc)
* path
* protocol
* status code

We typically are only interested in the operation, path, and status code.

At some point, run the `test` command.  What do you see in both
windows? We will return to this result in the next subsection. For
now, just note this behaviour.

Once you are comfortable with the application, terminate each component: the client using the
`quit` command; the server via `CTRL-C`. But keep the two windows
open---you'll be using them again in a moment.

A note about this version of the music application: its data is not
truly persistent. All data is thrown away when the server exits. 
In Assignment&nbsp;4 you will begin using a full
version of the service that stores its data persistently on AWS
DynamoDB.

### Fixing a defect in a cloud service is hard

As a programmer, you have likely follow the typical defect-removal
cycle:

1. Observe a system's behaviour contradicts its specification.
2. Inspect system diagnostics (from system logs, debug print statements, debugging
   tools, and other sources) to identity the source of the defect.
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

In this assignment (and the three to follow), we're going to give you
practice debugging cloud systems. This assignment begins with the
simplest case, debugging a server running on your local machine. By
Assignment&nbsp;4, we'll have moved up to debugging a server running
remotely on AWS.

In each of these assignments, the service has the same "bug": that
the `test` command throws an exception in the server and returns a
`500` status code, indicating
["Internal Server Error"](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).
Removing the exception and returning a `200` status code (indicating
["OK"](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)) will
be trivial and require no deep understanding of Python or even of how the
server works. **The point of this part of the assignment is to
practice reading a service's log output and restarting the fixed
service.** The trivial "bug" is at most a [MacGuffin](https://en.wikipedia.org/wiki/MacGuffin) to practice this cycle.

### Fixing defect in a local service (the Easiest Case)

This assignment encompasses the Easiest Case where the service is local to you. The server log
is just the output in its terminal window. The log shows the stack
traceback from the exception, as well as other information. In
particular, look for a line of output similar to this:

~~~
[2021-12-03 01:17:29,510] ERROR in app: Unique code: c69314133f0dfb2a79c93278335d6c10ed60498c20b03d49168db2344d579d89
~~~

Ignore the `ERROR` tag---this is just an informational message. The
unique code you see will differ from the one above as it varies for each assignment/student.

To fix this "bug", copy the code (the 64 hex characters, in this case
the sequence `c69314...579d89`), open the
file`c756-exer/s2/standalone/app-a1.py` in Visual Studio Code, and
paste it between the single quotes in the `if` statement of the
`test()` function, replacing whatever is between them:

Before (the 64-character code may differ from this or even just be `''`):

~~~python
@bp.route('/test', methods=['GET'])
def test():
    if ('1e0715252b48ed14858ae1ce646d67195183ffb8f9dc02d73c82323d8d75f482' !=
            ucode):
        raise Exception("Test failed")
    return {}
~~~

After (but use **your code**, not this one):

~~~python
@bp.route('/test', methods=['GET'])
def test():
    if ('c69314133f0dfb2a79c93278335d6c10ed60498c20b03d49168db2344d579d89' !=
            ucode):
        raise Exception("Test failed")
    return {}
~~~

### Testing the fix

Save the revised file (in Visual Studio Code) and (in the tools container) rebuild/rerun
the server in the server window:

~~~bash
/home/k8s/s2/standalone# ./builda1.sh
... build messages ...
/home/k8s/s2/standalone# ./runa1.sh
... server log messages ...
~~~

The client didn't change, so you don't need to update (rebuild) it, only rerun
it in its window:

~~~bash
/home/k8s/mcli# make run-mcli
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

Congratulations!  You've fixed the bug.

The process was trivial in this case but consider: What might be
required to read the log from a server running remotely (e.g., on the other side of
the world), packaged and isolated in a container, surrounded by
multiple layers of security providing very narrow access? And what
might be required to get the fixed software running on that remote
machine? And what would you need to do if you didn't know the nature/fix of problem?

### Committing the fix

You've found the defect and tested the fix but one more step remains:
Committing the fix to version control to make it permanent and visible
to the world.

In the next assignment, you will go into the details of Git. For now,
just execute the following commands. Recall that Git, along with
Visual Studio Code, is the only tool that you run in your
regular *Host OS*:

~~~bash
$ cd c756-exer/s2/standalone
c756-exer/s2/standalone $ git add app-a1.py
c756-exer/s2/standalone $ git commit -m 'Add missing code to "test"'
c756-exer/s2/standalone $ git push origin
~~~

### Review of this step

The coding part of this assignment was simple enough: Beginning with a
basic client-server design, find and fix a defect in the server. In
coming assignments, we will repeat this cycle of read
logs-fix-rebuild-test-commit.  As the server becomes more remote and
more protected, the "read logs" and "rebuild and test" steps will
require more tooling.

## Submission

Create a PDF file and provide the following:

1. Screen-capture of a terminal session showing the `c756-tool` container image. Use `docker image | grep c756` to retrieve this. (Aside: for students on MacOS or Windows, go to your Docker Desktop tray and bring up Dashboard/Images and compare the content.)

2. Screen-capture of a terminal session with your git commit. You can use `git log` to retrieve the history. 

3. Screen-capture of a terminal session with the permission of your cluster directory. You can use `ls -la cluster` from the `c756-exer` folder.

4. Your unique code for the error in the s2 application.

5. URL of the line of code with the fix in your Github repo's copy of `c756-exer/s2/standalone/app-a1.py`. Navigate to your repo inside Github and locate the file/line. Click on the line number and select "Copy permalink". 
![AWS Image](https://github.com/scp756-221/course-site/blob/main/docs/a1/github-permalink.png?raw=true)



Submit the file to [Assignment 1](https://coursys.sfu.ca/2022sp-cmpt-756-g1/+a1/) in CourSys.
