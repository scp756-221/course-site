---
title: Exercise 1: Dev tools & cloud
style:
  background: red
---

## Introduction

This exercise sets up a development environment on your Windows, MacOS or Linux laptop/desktop for the remainder of the course. You will be using a mix of Azure and AWS.

Upon completing this exercise, you will be ready to handle subsequent exercises and your term project.

**Please refer to the [Exercise 1 FAQ](https://docs.google.com/document/d/119mmEvvFgcXfsjlZNdnYlXGevLKkXYqK8fCN_t4tPIo/edit?usp=sharing) and [Exercise 1A FAQ](https://docs.google.com/document/d/1ln8JYxFjnpaT5IdrAmKlfgIRaQonj1BGqTbycz-ciEs/edit?usp=sharing) for up-to-date answers on common problems. I will update them continually as new information rolls in.**



## 1. Client tools

You will need to setup the appropriate tools on the machine you plan to use for this course's exercise, assignments, and project. The heavy lifting will be typically on the cloud side (AWS or Azure) but you will need a certain amount of tooling on your computer to operate the cloud.

The submission for this exercise will comprise the version information for these software. (Copy and paste the entire terminal output.) In your submission document, be sure to indicate which platform you used for this exercise in the top box. If you are using a CSIL lab machine for this exercise, most of the packages are installed already. You will still need to configure your account/access.

Note that section 1 is the installation; in many case, you will continue with configuration in section 2.


### 1.1 Editor

If you are not already using a GUI text editor, install [Atom](https://atom.io/). Other good choices include [Sublime Text](https://www.sublimetext.com/) and [Visual Studio Code](https://code.visualstudio.com/).

A good text editor raises your productivity significantly with syntax highlighting, syntax validation/hinting, support for multiple files/source trees, integration with version control (git) and numerous other creature comforts. **Do not skip this step.**

### 1.2 Package Manager

If you are not already using a package manager, install the appropriate one for your environment.

| Platform     | Package Manager     |
| :------------- | :------------- |
| Windows (10+) | [Chocolately](https://chocolatey.org/)       |
| MacOS | [Homebrew](https://brew.sh/) |
| Linux | Take your pick per your distro |

You will use the corresponding package manager to simplify the install of subsequent packages.

### 1.3 git

Install git per [the git community's instructions](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git). In many environments (e.g., MacOS), git comes pre-installed; check.

Install [Github Desktop](https://desktop.github.com/) too to complement the CLI git.

See [here](https://canvas.sfu.ca/files/13170454/download?download_frd=1) for a git cheatsheet. (Additional tutorial at [DZone](https://dzone.com/refcardz/getting-started-git) and a [supervisual cheatsheet by Matt Harrison](https://github.com/mattharrison/Git-Supervisual-Cheatsheet/blob/master/gitcheat.png).)

### 1.4 Docker

Install docker & Docker Desktop from [docker.com](https://docs.docker.com/get-docker/). If you have less than 8GB of RAM on your laptop, please upgrade. The alternative is to use CSIL/Blusson.

### 1.5 AWS

Install the AWS CLI per [Amazon's instruction](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)

Continue with [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html).

### 1.6 Microsoft Azure

Install the Azure CLI per [Microsoft's instruction](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

### 1.7 Google Cloud Platform

Install the gcloud CLI per [Google's instruction](https://cloud.google.com/sdk/docs/install)

### 1.8 Kubernetes & friends

Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) and [k9s](https://k9scli.io/).


### 1.9 istio

Install istioctl per [the istio community's instructions](https://istio.io/latest/docs/ops/diagnostic-tools/istioctl/). Note that you do not need to install the entire istio package; just istioctl will suffice.

Important: the ``export`` statement needs to be added to your ``.bashrc``.

Install helm per [the helm community's instructions](https://helm.sh/docs/intro/install/)

### 1.10 Gatling

[Install Gatling](https://gatling.io/open-source/start-testing/),
using the version appropriate for your system.

Note the path in which you installed it. You will need it in the next
step.

Test that Gatling is compatible with your version of Java by running
the following from the directory in which Gatling was installed. If
you have a compatible Java, this will print the Gatling help message:

~~~
$ bin/gatling.sh --help
~~~

Upgrade your Java install if this fails and record the path to the new
`java` for use in the next step.


## 2. Accounts

You will likely want to create bookmarks in your browser for the following services. As well, if you aren't using a password manager already, now's the time to get started. Finally, consider why you are *not* using 2FA. Each of these services supports 2FA which elevates the security of your account hugely.

### 2.1 Github

We will be using github.com in this course for a number of purposes:
1. Distribution of and/or grading of various homeworks (including the term project).
2. Collaboration with team members (including your term project).
3. Practicing the scrum methodology
4. Hosting of container images

Instruction:

1. If you do not have a GitHub account, create an account [here](https://github.com/join).

1. Follow the instructions [here](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration) for the command-line client. See [here](https://docs.github.com/en/github/authenticating-to-github/accessing-github-using-two-factor-authentication) to configure your git for github.com with 2FA.

1. Sign up on [Github Education](https://education.github.com/students). Start with 'Get benefits for students'.

1. Activate the Github Container Registry feature by [following the instructions](https://docs.github.com/en/free-pro-team@latest/packages/guides/enabling-improved-container-support).


### 2.2 Container Registry

We will be using containers in this course to streamline development. See [here](https://dzone.com/articles/container-registriesa-battle-royale) for an introduction to container registries.

1. If you do not have a docker.com account already, create an account at [docker.com](https://hub.docker.com/signup). Note that this grants you access to both [Docker.com](https://www.docker.com) and [DockerHub](https://www.dockerhub.com), Docker's container registry service. However, we will not be using DockerHub. Due to recent changes by DockerHub to [throttle requests from free accounts](https://www.docker.com/increase-rate-limits), we will be using [GitHub's new container registry feature](https://github.blog/2020-09-01-introducing-github-container-registry/) instead to host our container images. (See [here](https://docs.github.com/en/free-pro-team@latest/packages/guides/migrating-to-github-container-registry-for-docker-images) for more information.)

1. [Create a personal access token (PAT)](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token) for your Github account. You will need the three scopes: `read:packages`, `write:packages` and `delete:packages`.

2. Configure your docker client to use GitHub Container Registry (ghcr.io) using this new token as follows:

    ```bash
    $ export CR_PAT=<your-token>
    $ echo $CR_PAT | docker login ghcr.io -u <your-github-id> --password-stdin
    > Login Succeeded
    ```

### 2.3 Cloud Providers

1. Apply for course credits

   Refer to [Exercise 1A](https://canvas.sfu.ca/courses/59479/assignments/577244) for details on applying for student credits. You will apply for all three sets of credits.

2. Configure your cloud CLIs

   1. AWS: Refer [here](https://stackoverflow.com/questions/40515079/access-key-id-and-secret-access-key-for-aws-educate-account) for instruction on configuring your environment to use the Starter account. The important note is that Starter account feature a session token which ages out. You will need to refresh this periodically if your account is idle (~1h of inactivity).

   2. Microsoft Azure: Refer [here](https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli) for instruction to sign into Azure. The easiest is the interactive option (at the top of the page). Then set your defaults,

   3. Google Cloud Platform: Set your GCP defaults with `gcloud init`.

3. Confirm your credits

   1. AWS: If you are using a starter account, confirm the amount of your available credits.

   a. Navigate to https://aws.amazon.com/education/awseducate/
   b. Click on “Sign in to AWS educate”,
   c. Enter your email and password,
   d. Click on “AWS Account” from the top menu,
   e. On this page, you should see a statement such as: “Your account has an estimated X credits remaining and access will end on Y ,Z.”
   f. By clicking on the “AWS Educate Starter Account” orange button, you can see your AWS account status.
   g. Screen-grab this page and submit.

   (If you are using a personal account, make a note of this on the top "Cloud Providers" box and skip this step.)

   2. Microsoft Azure: Explore your account and locate the page that shows your available credits.
   Start your navigation at the [Azure Portal](https://portal.azure.com). Then find 'Subscriptions' (or navigate [directly](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)). Click on the entry 'Azure for Students'. You should find a link for "To check your remaining credit...". Navigate to this and screen grab the page showing your available credit.

   (If you are funding your account with a credit card, make a note of this on the top "Cloud Providers" box and skip this step.)

   3. Google Cloud Platform: Explore your account and locate the page that shows your available credits. Screen grab the page showing your available credit.

   (If you are funding your account with a credit card, make a note of this on the top "Cloud Providers" box and skip this step.)


## Submission

### Create a PDF

Make a copy of the [submission template](https://docs.google.com/document/d/10kVl3R5tMVgxqdzO0QAXcRRxxRcWcGcdQnlPqvBT6WE/edit?usp=sharing)(GDoc format).

Fill in:

a. The header box at the top of the document.

b. Content from the steps above.

Generate a PDF when you are done.

You must name your PDF according to the pattern: **SFU-id**`-e1-submission.pdf` where **SFU-id** is the portion of your email address preceding `@sfu.ca`. Unfortunately, you will be penalized for incorrect filename because of cascading dependencies for a large class.

**A penalty will be assessed for failure to name your submission appropriately.**

### Canvas submission

Navigate to this exercise and upload the generated PDF.
