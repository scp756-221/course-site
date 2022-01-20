# Assignment 2&mdash;AWS EC2 & git

## Introduction

This assignment comprises two parts.  First, you will start an Amazon
EC2 instance and then build and run the music service remotely on it. You will
continue to run the client on your local machine. This will be your
first example of of the client and server running on
different machines.

Second, you will explore the features of git, GitHub Desktop and GitHub.

**This assignment has two prerequsites:**

**1. You must have completed the full AWS sign up section of
  Assignment&nbsp;0.**

**2. You must have completed all of Assignment&nbsp;1.**

### Generate course code repo instance

**You must have successfully installed Git before starting this
step.**

**You must have a GitHub userid (see the first part of
Assignment&nbsp;0) before starting this step.**

Due to corrections made since Assignment 1, you will need to regenerate a repo for Assignment 2. The process is similar:

1. Sign in to GitHub using your userid.
2. Sign in to [GitHub Education](https://education.github.com/) using the same userid. 
3. Accept this assignment via this [link](https://classroom.github.com/a/3cAM2ygY)
4. Wait while a copy of the repo is generated for you.
5. Bookmark the URL of your copy of the repo http://github.com/scp756-221/assignment2on-YOUR-GITHUB-ID. As the repo resides in this class' organization, you will not see it listed under "Your repositories". But you will find it at the left-hand margin of your [GitHub home page]](https://github.com) after you sign in.
6. Reuse Assignment 1's `tpl-vars.txt`. **Do not delete the old Assignment 1 repo.** If you'd cloned Assignment 1's repo into `c756-exer` locally, you will find it convenient to rename this so as to free up the name for this new copy:
```sh
$ mv c756-exer c756-exer-assignment1
```

7. Clone the repo from step 4 to your laptop and re-instantiate the templates. (You may need to adjust the Assignment 1 repo name below.)
```sh
$ git clone https://github.com/scp756-221/assignment2on-YOUR-GITHUB-ID.git c756-exer
$ cp c756-exer-assignment1/cluster/tpl-vars.txt c756-exer/cluster/tpl-vars.txt
$ cd c756-exer
$ make -f k8s-tpl.mak templates
```



## Part 1: Running an Amazon EC2 instance

In this part you will repeat the build-debug-fix-test-commit cycle of Assignment&nbsp;1 with one difference. This time the server will be running in the cloud, on an Amazon EC2 virtual machine ("instance"). Note the point of Part 1 is to arrive at an EC2 instance while providing some visibility into the EC2 service for students who are either unfamiliar with the concept of virtualization or cloud service. Thus, the steps provided far from optimal but adequate for this. 

### Setting up your AWS credentials

If you set up your AWS credentials in Assignment&nbsp;1 and instantiated the templated code, you can skip this subsection.

If you have obtained your AWS credentials since you completed Assignment&nbsp;1, you need to add them to the template variables file now. Update the AWS credentials in `cluster/tpl-vars.txt` and then re-instantiate the templates in the tools container:

~~~bash
/home/e-k8s# make -f k8s-tpl.mak templates
~~~

### Creating and running an EC2 instance

Your first step is to start an EC2 instance. You will do this through the AWS Console.

1. Sign on to the AWS console, `https://console.aws.amazon.com/` , using your IAM administrative userid. If you have set up Multi-Factor Authentication (MFA), you will need to enter the one-time code as well.
2. In the search bar at the top of the page, enter `EC2` and press Return.
3. In the top right, select `Oregon` from the menu of regions.
3. Click the orange `Launch instance` button, in the mid-lower left (you may have to scroll down).
4. **Step 1 Page:** In the search bar, enter `Deep Learning AMI (Amazon Linux 2)` and press Return.
5. From the list, click on `Select` for an entry labelled `Deep Learning AMI (Amazon Linux 2) Version 57.0` or a similar version.  Note: We chose this machine image as a matter of convenience (this image happens to have all the tools we need already) and not because we are performing any deep learning in this assignment.
6. **Step 2 Page:** Click the check box to the left of the row with `t1.micro`. Then click on the label `6. Configure Security Group` at the top of the page. (We are skipping Steps 3--5.) (The instance type is not significant... we are choosing the smallest instance. Something slightly larger/newer is okay too. The cost will be higher but for what you're doing here, the difference is not consequential.)
7. **Step 6 Page:** In the `Configure Security Group` dialogue:
   0. Select `Create a new security group` (it is the default).
   1. *Security group name:* `Music service (port 30001)`
   2. Click `Add Rule` and then `Custom TCP Rule` from the dropdown.
   3. In the new row that is added:
      * *Port Range:* `30001`
      * *Source:* (Menu) `Anywhere`
      Ignore Amazon's warnings about the insecurity of using "Anywhere".
   4. Click on `Review and Launch`.
8. **Step 7 Page:** Review that the above settings are correct. The security group rule that you added in the previous step will have expanded to two rules, `0.0.0.0/0` (IPV4) and `::/0` (IPV6). Once you are satisfied, click `Launch`.
9. In the key pair selection dialogue:
   * Select `Choose an existing key pair` (the default)
   * In the drop down list, select the key pair that you created in Assignment&nbsp;0.
   * Click the box acknowledging that you have access to the key created in Assignment&nbsp;0.
   * Click `Launch Instances` (only one will be launched).
10. Click `View Instances` (lower right).
11. In the Instance view, wait until *Instance state* is `Running`.  However, the instance is not ready to run yet. Watch the "Status check" column.  Occasionally click the refresh button (the clockwise circle icon), waiting until the field reads `2/2 checks passed`.
12. Click on the instance ID link (it will look like `i-0fada0697f42aed7e`).  This will bring up the instance summary.
13. Locate the `Public IPv4 DNS` entry, mid-right of the page. It will look like `ec2-34-210-56-181.us-west-2.compute.amazonaws.com`. Don't do anything with it for now.

You have just instantiated a virtual machine running Linux in one of Amazon's Oregon datacentres.  The hardware for this machine is rather weak (~1 CPU core with ~0.5GB of RAM) and doesn't cost much to run (USD0.02/h which works out about CAD0.60/day). But it is more than sufficient for this assignment.

### Build and run the music server

Now that the instance is running, we need to build the music server and run it there.

1. In the Amazon instance summary, locate the instance ID again and this time, click the copy icon ("overlapping squares") to the left of the name.  This will copy the long name into your copy buffer.

2. In the tools container, enter the following commands, where `KEY-FILE` is the name you gave to your key file (including the `.pem` extension) saved in Assignment&nbsp;0 and `EC2-DNS-NAME` is pasted in from the copy you just made:

   ~~~bash
   /home/k8s/s2/standalone# ./signon.sh ~/.ssh/KEY-FILE ec2-user EC2-DNS-NAME
   ... login output ...
   [ec2-user@ip-172-31-25-98 ~]$
   ~~~


   If you see the message and input prompt (with different values for the address and fingerprint):

   ~~~bash
   The authenticity of host 'ec2-35-162-179-25.us-west-2.compute.amazonaws.com (35.162.179.25)' can't be established.
   ECDSA key fingerprint is SHA256:HJ5/KcAAVadyTNp/OL00V9j8ue/+spC5nU1tmUCSim0.
   Are you sure you want to continue connecting (yes/no/[fingerprint])?
   ~~~

   answer `yes`. The message simply indicates that you have never connected to this machine before.

   The window is now showing a terminal session on *the remote Amazon instance*, not your machine. (The `ip-172-31-25-98` portion of the prompt will vary with the IP address of your particular instance.)

3. Clone the Assignment 2 repo generated above with the following, where `REGID` is your GitHub userid. **You will need to authenticate/configure git on this EC2 instance as it is a fresh installation. Use the PAT from Assignment 1 as you would your password when prompted.**

   ~~~bash
   [ec2-user@ip-172-31-25-98 ~]$ git clone https://github.com/scp756-221/assignment2on-REGID.git c756-exer
   [ec2-user@ip-172-31-25-98 ~]$ cd c756-exer/s2/standalone
   ~~~

4. From your host environment, transfer student-specific files to your EC2 instance, performing the same substitution as in Step&nbsp;2. 

   ~~~bash
   $ cd .../c756-exer/s2/standalone
   .../c756-exer/s2/standalone $ ./transfer.sh ~/.ssh/KEY-FILE ec2-user EC2-DNS-NAME
   + scp -i /home/...  Dockerfile ec2user@EC2-DNS-NAME:...
   ...
   + scp -i /home/...  unique_code.py ec2user@EC2-DNS-NAME:...
   + scp -i /home/...  music.csv ec2user@EC2-DNS-NAME:...
   ...
   ~~~

   You will then need to move `unique_code.py` (on your EC2 instance) into the appropriate location. If you used the suggested `c756-exer`, this will be `c756-exer/s2/standalone`:

   ~~~bash
   [ec2-user@ip-172-31-25-98 ~]$ mv ~/unique_code.py ~/c756-exer/s2/standalone/unique_code.py
   ~~~


5. Build and start the music service on this remote instance:

   ~~~bash
   [ec2-user@ip-172-31-25-98 standalone]$ ./builda2.sh
   ... lengthy build output ...
   Successfully tagged s2-standalone:v0.5
   [ec2-user@ip-172-31-25-98 standalone]$  ./runa2.sh
   [2021-12-07 00:47:38,204] ERROR in app: Unique code: bb6f4ceb0082f107f759d61a861600ec6b7266c18ffb0d6ca3b42c27df2fa6f4
    * Serving Flask app "app" (lazy loading)
    * Environment: production
      WARNING: This is a development server. Do not use it in a production deployment.
      Use a production WSGI server instead.
    * Debug mode: off
    * Running on http://0.0.0.0:30001/ (Press CTRL+C to quit)
   ~~~

The music service is now running remotely on the Amazon machine in Oregon.

### Run the client

To test the service, you will run the client on your local machine, where `EC2-DNS-NAME` is the DNS name of the remote Amazon instance where the music service is running:

~~~bash
/home/e-k8s# cd mcli/
/home/k8s/mcli# make SERVER=EC2-DNS-NAME run-mcli
docker container run -it --rm --name mcli mcli:v0.8 python3 mcli.py ec2-34-210-56-181.us-west-2.compute.amazonaws.com 30001

Command-line interface to music service.
Enter 'help' for command list.
'Tab' character autocompletes commands.

mql: 
~~~

Try a few commands, reading the music list, perhaps adding or deleting a song. (No changes that you might have made to the song list in Assignment&nbsp;1 will be available, as this is a completely different instance.)

Once you have verified that the music system is running, try the `test` command.  It will cause a failure because the required SHA code for Assignment&nbsp;2 differs from the one required in Assignmment&nbsp;1.

Terminate the music service by entering the `shutdown` command in the music client. 

### Find and fix the "bug"

You will follow a similar sequence of steps as from Assignment&nbsp;1 to locate and fix the bug. However, this time around, you will use GitHub Desktop instead to 'operate' git. 

1. *On the remote machine*, locate the required code in the output from the music service.

2. *On your local machine, in your Host OS*, add the code to `app-a2.py` using Visual Studio Code.

3. Launch GitHub Desktop and:

   * Add the repo
   * Observe the visual diff that GitHub Desktop provides
   * Commit the code to the local repo as before with one change: use a multi-line comment.
   * Check that your own GitHub repo does not contain this fix
   * Push the code up to your own copy of the repo
   * Recheck that your GitHub repo now has the fix

4. *On the remote machine*, pull the
   revised code down from GitHub (note the directory) :

   ~~~bash
   [ec2-user@ip-172-31-25-98 standalone]$ cd ../..
   [ec2-user@ip-172-31-25-98 c756-exer]$ git pull
   ~~~

5. *On the remote machine*, rerun `./builda2.sh`.
6. *On the remote machine*, rerun `./runa2.sh`.
7. *On your local machine, in the tools container*, in the `mcli` client try the `test` command. It should now work, with no error status code displayd by the client and with no traceback in the server log.
8. You will commit the revision and push it in the next section, where you do further work with Git.

### Terminate the EC2 instance

As Amazon is charging by the minute for the EC2 instance, you will want to dispose of the instance as soon as reasonable. On the other hand, keep in mind the absolute charges involved. As this instance is very small and cheap (USD0.02/h), you may wish to explore the mobile apps ([iOS](https://apps.apple.com/us/app/aws-console/id580990573) & [Android](https://play.google.com/store/apps/details?id=com.amazon.aws.console.mobile&hl=en_US)) to observe and manage your resources.

1. If you are still signed in to the remote instance, exit it:

  ~~~bash
  ^C to exit the music service, if running.
  [ec2-user@ip-172-31-25-98 ~]$ exit
  /home/k8s/s2/standalone# 
  ~~~

2. If you wish to terminate the instance from the Web console, start from the Amazon Console Instance summary and select `Terminate instance` from the `Instance state` dropdown menu.  Click `Terminate` to confirm.

3. Verify that the instance was actually terminated by going to the instance list and wait until the *Instance state* shows `Terminated`. You may have to refresh the display a few times to see it.

### Reflecting and looking ahead

Running the service remotely required more work than was required when you were running it locally in Assignment&nbsp;1. The extra work fell under three categories:

1. Configuring and starting a remote machine. (This is something that may be streamlined by process and/or improved tooling.)
2. Transferring the source files to the remote machine. 
3. Security to ensure that attackers cannot charge their use of AWS to your account.

The next two assignments will focus on technologies that address the first two issues ... but require even more stringent security measures:

* In Assignment&nbsp;3, we will explore *containers*, which allow us to build and package a service on one machine and run that package elsewhere.
* In Assignment&nbsp;4, we will explore *Kubernetes*, a container *orchestrator*, which handles the allocation and management of remote machine resources.

The cost of these improvements will be more elaborate security requirements. Consider the security risk of an automated system that would accept arbitrary programs and run them on your machines. We'll obviously need some signifcant access controls and monitors to ensure only authorized use. And those controls will inevitably make development and debugging more awkward for developers.

## Part 2

Do Scenarios&nbsp;1--6 of the [Katacoda Git exercises](https://www.katacoda.com/courses/git). You only need to sign on with your email---you do not need to pay anything.

Continue with [GitHub Desktop Docs, Part 3: Contributing to projects with GitHub Desktop](https://docs.github.com/en/desktop/installing-and-configuring-github-desktop/overview/getting-started-with-github-desktop#part-3-contributing-to-projects-with-github-desktop)


Follow the instructions there to:

1. Create a **public** repo with the name "Created-in-GitHub" in your GitHub account and clone it down to your laptop.

    1. Create the file `file.txt` as follows:
    ```bash
    $ uname -a > file.txt
    ```
    2. Add `file.txt` and commit this into your local repo **using the git CLI**. Work out how to write a long multi-line commit comment for this.
    2. Push this up to GitHub **using the git CLI**.


2. Create a repo with the name "Created-in-git" on your laptop and push it up to GitHub.

    1. Create the file `file.txt` as follows:
    ```bash
    $ docker info > file.txt
    ```
    2. Add `file.txt` and commit this into your local repo **using GitHub Desktop**. Write a long multi-line commit comment for this.
    3. Push this up to GitHub using GitHub Desktop. 


## Submission

Create a PDF file and provide the following:

1. URL of your GitHub repo "Created-in-GitHub"

2. URL Of your GitHub repo "Created-in-git"

3. Screen-capture of a terminal session with the git commit that correct the problem. You can use `git log` to retrieve the history.  (Do this on the host side.)

4. URL of the line of code with the fix in your Github repo's copy of `s2/standalone/app-a2.py`. Navigate to your repo inside Github and locate the file/line. Click on the line number and select "Copy permalink". 
![AWS Image](https://github.com/scp756-221/course-site/blob/main/docs/a1/github-permalink.png?raw=true)

5. What are some potential difficulties or obstacles for building the service in your EC2 instance? What potential solutions can you offer for handling these?
 
Submit the file to [Assignment 2](https://coursys.sfu.ca/2022sp-cmpt-756-g1/+a2/) in CourSys.
