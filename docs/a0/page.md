## Creating userids for GitHub and Amazon Web Services (AWS)

This assignment guides you through the signup process for GitHub and
AWS.

You will need a GitHub userid to complete the Week&nbsp;1 assignment.

You will need an AWS userid to complete the Week&nbsp;2 assignment.

## Do you already have a GitHub id?

Even though you already have a GitHub userid you may not have set up all
the features we need for this course:

* GitHub Education
* GitHub container registry
* Container registry personal access token

To set up each of these, proceed to the appropriate section below.

## Do you already have an AWS id?

Even though you already have an AWS userid you may not have set up all
the features we need for this course:

* AWS root user
* IAM administrative user with an active access key
* At least one ssh key pair for EC2
* Budget alert

To set up each of these, proceed to the appropriate section below.

---

## Sign up for GitHub

We will be using GitHub for a number of purposes:

1. Distribution of and/or grading of various homeworks (including the term project).
2. Collaboration with team members (including your term project).
3. Hosting container images.

### Create a GitHub account

1. If you do not have a GitHub account,
   [create one](https://github.com/join).  We *strongly recommend* setting
   up Multi-Factor-Authentication (MFA) for this account.
2. [Configure your Git command-line client](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration).  **You must use the short form of your SFU email address for this. (This is typically a 5-6 letter sequence following by a number of digits.)** This is to disambiguate between students in the future.
3. If you have Multi-Factor Authentication (MFA) for your GitHub account, 
   [configure your command-line for Github.com with MFA](https://docs.github.com/en/github/authenticating-to-github/accessing-github-using-two-factor-authentication).

### Join GitHub Education

Sign up for [GitHub Education](https://education.github.com/students).
Start with 'Get benefits for students'.

### Activate GitHub Container Registry

[Activate the GitHub Container Registry](https://docs.github.com/en/free-pro-team@latest/packages/guides/enabling-improved-container-support)
for your GitHub userid.

### Generate a GitHub access token

The final step of setting you GitHub account, generating an access
token, will be done in Assignment&nbsp;1, as it requires some
tooling that will be installed in that Assignment.

---

## Sign up for AWS

If you do not already have an AWS account, go to
[aws.amazon.com](https://aws.amazon.com/) and click `Create an AWS
Account`.  You will need a credit card number to complete the
process. Note that AWS accepts prepaid credit cards, if you do not
have a Canadian credit card. Prepaid credit cards are also a great way
to ensure an absolute maximum to your AWS charges for the semester.

See
[How do I create and activate a new AWS account?](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)
for detailed instructions. For support plans, select the free "Basic
Support" plan.

**We strongly recommend setting
  [Multi Factor Authentication (MFA)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html)
  on your AWS root user account.**

## Set up an IAM user with full administrative privileges

The AWS account you set up in the previous step is the
[root user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_root-user.html). This
user has total control over your account. *Use the root user
sparingly.* Really, you only need to use it once, to set up an "IAM
administrative user".  This id will have the same high privileges
as your root user but if this id is compromised you can sign in to
your root user and disable it. Of course, for the administrative
user to actually provide a second level of security, **the IAM
administrative user must have a different password from your root
user.**

Follow Amazon's instructions for
[how to create your IAM administrative user](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html).

**As with the root user, we strongly recommend setting
  [Multi Factor Authentication (MFA)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html)
  on your IAM administrative account.**

### Create an access key 

Finally, you need an access key for this IAM administrative
user. Follow Amazon's instructions to
[create an access key](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html). This
key will be used with the AWS command-line interface. Refer to this
[gist to configure Amazon command-line](https://gist.github.com/overcoil/4d0bf31d8a9c8f4ec6f58b2bd289668f). (But do not create an access key for your root AWS account.)

## Create an ssh key pair for EC2

You will also need a key pair for SSHing into Amazon EC2 virtual
machine instances.

**Perform this step on a private computer (e.g., your personal laptop), not a publicly-shared
  machine (e.g., a CSIL workstation). You will be downloading a private key that will serve as
  your identification to AWS machine instances.**

1. Sign on using your IAM administrative user.
2. Type `key pair` into the service search bar at the top of the page.
3. Click on `Key pairs` (around the middle of the search results,
   under `Features`).
4. If you have at least one key pair in the list, you do not need to
   do anything more.
5. If the list is empty, click `Create key pair` (top right)
6. In the "Create key pair" dialog:

   * Name (Text): *Enter a name* (You might include "AWS" in the name
     to remind you of the key's purpose)
   * Key pair type (Radio button): RSA [Default]
   * Private key file format (Radio button):
     [Pick the type appropriate for the SSH tool you use; most times that's OpenSSH]
   * Tags: [None needed]

   Click `Create key pair`.

**The private key will be downloaded to your machine.**

On your machine, do the following:

1. Save the key file in ssh's default hidden directory (`~/.ssh`)
2. Set the access modes on the key file so that only you have access:

   ~~~
   $ chmod go-rwx <KEYFILENAME>
   ~~~

3. Set the access modes on the key file so that even you cannot
   modify it:

   ~~~
   $ chmod u-w <KEYFILENAME>
   ~~~

You're almost done! There's one more step, which will help you avoid
getting nasty surprises on your credit card bill.

## Set a budget alert

In this final step, you will set up an alert so that AWS emails you
henever your monthly bill has gone above a specified limit. Note that
AWS bills in US dollars though its dashboard does perform dynamic currency
conversion to show an approximation of your charge in the local
currency. (The charge to your credit card remains in US dollars, though.) We
suggest a limit of $20 US but you can set any value you find
comfortable. These alerts are useful if you forget to turn a service
off and leave it running or if a hacker gains access to your
account&mdash;AWS will alert you that you're racking up unexpected
charges.

There are also mobile apps (for
[iOS](https://apps.apple.com/us/app/aws-console/id580990573) and
[Android](https://play.google.com/store/apps/details?id=com.amazon.aws.console.mobile&hl=en_US))
to monitor and manage EC2 and some (but not all) other AWS services.

0. Sign on using your IAM administrative user.
1. Type `budget` in the service search bar at the top of the page.
2. Click `AWS Budgets`.
3. Click `Create a budget`.
4. Select `Cost budget` radio button [Default].  Click `Next`.
5. In the "Set budget amount" step:

   * Period (Dropdown): `Monthly` [Default]
   * Effective date (Radio button): `Recurring Budget` [Default]
   * Start month (Date): *Current month* [Default]
   * Choose how to budget (Radio button): `Fixed` [Default]
   * Enter your budgeted amount (Text): *Threshold for the alert: $20 US is a good start*
   * Budget name (Text): *Name*

   Click `Next`.
6. In the "Configure alerts" step:

   * Click `Add an alert threshold`
   * Threshold (Numeric): 100%
   * Trigger (Dropdown): `Actual` [Default]
   * Email recipients (Text): *Enter your email*

   Click `Next`.

7. In the "Attach actions" step:
   * Do not add any actions---click `Next`.

8. In the "Review" step"
   * Review your settings and if correct, click `Create budget`.

If you'd like you can set multiple levels of budget alerts. We ask
that you set at least one.

---

## Submission

Create a PDF file with the following:

1. Screen shot of your Budgets Overview. Your screen shot should
include both the budget and the IAM userid in the top right corner, as
highlighted in the following screen shot.  You do not need to add the
highlights to your screen shot.

<img src="https://github.com/scp756-221/course-site/blob/main/docs/a0/AWS-IAM-id-with-budget.png?raw=true" alt="Screen shot of budget set for IAM administrative userid" width="800"/>

Submit the file to [Assignment 0](https://coursys.sfu.ca/2022sp-cmpt-756-g1/+a0/) in CourSys.