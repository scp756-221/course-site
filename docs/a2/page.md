## Introduction

This assignment comprises two parts.  First, you will start an Amazon
EC2 instance and then build and run the music service remotely on it. You will
continue to run the client on your local machine. This will be your
first example of of the client and server running on
different machines.

Second, you will explore the features of the Git version control system.

**This assignment has two prerequsites:**

**1. You must have completed the full AWS sign up section of
  Assignment&nbsp;0.**

**2. You must have completed all of Assignment&nbsp;1.**

## Part 1: Running an Amazon EC2 instance

In this part you will repeat the build-debug-fix-test-commit cycle that you performed in Assignment&nbsp;1, only this time the server will be running in the cloud, on a remote instance of an Amazon EC2 virtual machine.

### Setting up your AWS credentials

If you set up your AWS credentials in Assignment&nbsp;1 and instantiated the templated code, you can skip this subsection.

If you have obtained your AWS credentials but not yet configured your system, you will do it in this section.

BLERG CHECK

In the tools container, update the AWS credentials and instantiate the templates.  The parameter `AWS-CRED` is the path to the AWS credential file you downloaded in the AWS section of Assignment&nbsp;0:

~~~bash
/home/e-k8s# tools/aws-cred.sh AWS-CRED
/home/e-k8s# make -f k8s-tpl.mak templates
~~~

### Creating and running an EC2 instance

Your first step is to start an EC2 instance. You will do this through the AWS Console.

1. Sign on to the AWS console, `https://console.aws.amazon.com/` , using your IAM administrative userid. If you have set up Multi-Factor Authentication (MFA), you will need to enter the one-time code as well.
2. In the search bar at the top of the page, enter `EC2` and press Return.
3. In the top right, select `Oregon` from the menu of regions.
3. Click on `Launch instances`, in the upper right.
4. **Step 1 Page:** In the search bar, enter `deep learning` and press return.
5. From the list, click on `Select` for the entry labelled `Deep Learning AMI (Amazon Linux 2) Version 55.0`.  Note: We will not be doing deep learning in this exercise. This configuration just happens to have all the tools we need already installed.
6. **Step 2 Page:** Click the check box to the left of the row with `t1.micro`. Then click on the label `6. Configure Security Group` at the top of the page. (We are skipping Steps 3--5.)
7. **Step 6 Page:** In the `Configure Security Group` dialogue:
   0. Select `Create a new security group` (it is the default).
   1. *Security group name:* `Music service (port 30001)`
   2. Click `Add Rule`.
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
10. Click `View Instances`.
11. In the Instance view, wait until *Instance state* is `Running`.  However, the instance is not ready to run yet. Watch the "Status check" column.  Occasionally click the refresh button (the clockwise circle icon), waiting until the field reads `2/2 checks passed`.
12. Click on the instance ID link (it will look like `i-0fada0697f42aed7e`).  This will bring up the instance summary.
13. Locate the `Public IPv4 DNS` entry, mid-right of the page. It will look like `ec2-34-210-56-181.us-west-2.compute.amazonaws.com`. Don't do anything with it for now.

You have just instantiated a Linux virtual machine in one of Amazon's Oregon datacentres.  The machine you started isn't very powerful and won't cost much to run but it is sufficient for this exercise.

### Build and run the music server

Now that the instance is running, we need to build the music server and run it there.

0. In the Amazon instance summary, locate the instance ID again and this time, click the copy icon ("overlapping squares") to the left of the name.  This will copy the long name into your copy buffer.
1. In the tools container, enter the following commands, where `KEY-FILE` is the name you gave to your key file (including the `.pem` extension) saved in Assignment&nbsp;0 and `EC2-DNS-NAME` is pasted in from the copy you just made:

   ~~~bash
   /home/e-k8s# cd s2/standalone/
   /home/k8s/s2/standalone# ./transfer.sh ~/.ssh/KEY-FILE ec2-user EC2-DNS-NAME
   ... files transferred ...
  ~~~

  If you see the message and input prompt (with different values for the address and fingerprint):

  ~~~bash
  The authenticity of host 'ec2-35-162-179-25.us-west-2.compute.amazonaws.com (35.162.179.25)' can't be established.
  ECDSA key fingerprint is SHA256:HJ5/KcAAVadyTNp/OL00V9j8ue/+spC5nU1tmUCSim0.
  Are you sure you want to continue connecting (yes/no/[fingerprint])?
  ~~~

  answer `yes`. The message simply indicates that you have never connected to this machine before.

2. Sign on to the remote instance:

  ~~~bash
  /home/k8s/s2/standalone# ./signon.sh ~/.ssh/KEY-FILE ec2-user EC2-DNS-NAME
  ... login output ...
  [ec2-user@ip-172-31-25-98 ~]$ 
  ~~~

  The window is now showing a terminal session on *the remote Amazon instance*, not your machine. (The `ip-172-31-25-98` portion of the prompt will vary with the IP address of your particular instance.)

3. Build and start the music service on this remote instance:

   ~~~bash
   [ec2-user@ip-172-31-25-98 ~]$ ./builda2.sh
   ... lengthy build output ...
   Successfully tagged s2-standalone:v0.5
   [ec2-user@ip-172-31-25-98 ~]$ ./runa2.sh
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

Once you are verified that the music system is running, try the `test` command.  It will cause a failure because the required SHA code for Assignment&nbsp;2 differs from the one required in Assignmment&nbsp;1.

Terminate the music service by entering the `shutdown` command in the music client. 

### Find and fix the "bug"

Use the same sequence of steps that you used in Assignment&nbsp;1 to locate and fix the bug, with the difference that you are editing the service code on your **local** machine but building it on the **remote** machine:

1. *On the remote machine*, locate the required code in the output from the music service.
2. *On your local machine, in your Host OS*, add the code to `app.py` using Visual Studio Code.
3. *On your local machine, in the tools container*, rerun `./transfer.sh` with the same arguments as last time. This will transfer the revised code to the remote machine.
4. *On the remote machine*, rerun `./builda2.sh`.
5. *On the remote machine*, rerun `./runa2.sh`.
6. *On your local machine, in the tools container*, in the `mcli` client try the `test` command. It should now work, with no error status code displayd by the client and with no traceback in the server log.
7. You will commit the revision and push it in the next section, where you do further work with Git.

### Terminate the EC2 instance

Amazon is charging you by the minute for running the EC2 instance, so as soon as you're done with this part of the assignment, terminate the instance:

1. If you are still signed in to the remote instance, exit it:

  ~~~bash
  ^C to exit the music service, if running.
  [ec2-user@ip-172-31-25-98 ~]$ exit
  /home/k8s/s2/standalone# 
  ~~~

2. In the Amazon Console Instance summary, select `Terminate instance` from the `Instance state` dropdown menu.  Click `Terminate` to confirm.
3. Verify that the instance was actually terminated by going to the instance list and wait until the *Instance state* shows `Terminated`. You may have to refresh the display a few times to see it.

### Reflecting and looking ahead

Running the service remotely required much more work than was required to run it locally in Assignment&nbsp;1. The extra work fell under three categories:

1. Configuring and starting a remote machine.
2. Transferring the source files to the remote machine.
3. Security to ensure that attackers cannot charge their use of AWS to your account.

The next two assignments will focus on technologies that address the first two issues ... but require even more stringent security measures:

* In Assignment&nbsp;3, we will explore *containers*, which allow us to build and package a service on one machine and run that package elsewhere.
* In Assignment&nbsp;4, we will explore *Kubernetes*, a container *orchestrator*, which handles the allocation and management of remote machine resources.

The cost of these improvements will be more elaborate security requirements. Consider the security risk of an automated system that would accept arbitrary programs and run them on your machines. We'll obviously need some signifcant access controls and monitors to ensure only authorized use. And those controls will inevitably make development and debugging more awkward for developers.

## Part 2

BLERG George lists some Katacoda Git scenarios.