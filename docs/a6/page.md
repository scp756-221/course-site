## Introduction

In this assignment you will explore the range of speedups enabled by
graphics processing units
(GPUs). These are just one example of the growing range of
hardware designed to accelerate specialized computations. Other
approaches include

* Google's
  [Tensor Processing Units (TPUs)](https://en.wikipedia.org/wiki/Tensor_processing_unit)
  for neural networks
* Datacenters featuring
  [Application-Specific Integrated Circuits (ASICs)](https://dl-acm-org.proxy.lib.sfu.ca/doi/10.1109/ISCA.2016.25) for
  algorithms such as video transcoding and deep learning

The survey ["Domain-Specific Hardware Accelerators"](https://dl-acm-org.proxy.lib.sfu.ca/doi/10.1145/3361682)
provides good background on these trends.

As the name implies, GPUs were originally designed to accelerate
real-time graphics for games. Over the last 15 years however, GPU vendors have
extended the architecture to support a wider range of domains that
benefit from the style of parallelism supported by these chips,
renaming the devices as "General Purpose GPUS" (GPGPUs). However, the
original name, GPU, remains in common use and that is how we refer to
them in the course notes and this assignment.

In every one of these architectures, the design increases performance
for a specialized computation at the heart of a commercially-important
algorithm, the *computational kernel* of that algorithm, at the expense
of not supporting computations outside that narrow focus.

The algorithms for training neural networks satisfy all the
requirements for support from specialized hardware:

* Commercially important
* Computationally intensive
* High potential parallelism

Although they claim to be general, "General Purpose" GPUs remain
sufficiently specialized that they only accelerate some net
designs. The steps in this assignment ask you to compare the degree of
speedup for several net configurations.

## Remote sign in to the Blusson Hall machines

You will perform this assignment by signing in remotely to the Blusson
Hall lab for the Professional MSc. Programs. These machines are Dell
Precision workstations featuring a Geforce RTX 2080 GPU.

**Note:** If you are running a VPN, you may need to turn it off to be able
to sign in to the CSIL machines.

If you haven't already signed-up for a workstation, do so now via the
[GDoc sign-up](https://docs.google.com/spreadsheets/d/1rs7guBFRHY5aVtiHSTUEiJ7yxMYOJHsx3ko6tfuP0Ro/edit?usp=sharing).


As this class is markedly larger than the lab (109 students vs 36 workstations),
there will be contention for the workstations. The sign-up sheet will manage the contention
by clustering three students for each workstation. This small cluster of students can more readily balance
the contention by negotiating/working around each other's schedule. This assignment has an extended
deadline to accommodate this.



### Linux / MacOS / Windows Subsystem for Linux (WSL) systems

To complete this exercise, you will require exclusive access to a machine.
As the lab is not set up to grant exclusive access, we will achieve the same
by having each student use distinct machines from the pool available and
check for other users before starting.

Begin by setting up convenient ssh access via pre-defined configurations within
your `~/.ssh/config` file. Create the file (`touch ~/.ssh/config`) and add the following lines:

~~~
Host blu-XYZ
Hostname MACHINE_NAME
Port 24
User LOGIN_NAME
~~~

Substitute XYZ for the trailing three characters of the workstation you signed up for
previously. For example, if you signed up for `blu9402-b08.csil.sfu.ca`, you would use `b08`.

Substitute MACHINE_NAME with fully-qualified hostname of the machine you signed for (e.g., "blu9402-b08.csil.sfu.ca" but without the double-quotes).

Substitute LOGIN_NAME with your SFU login ID.

Save the file.

From this point on, you can use the short command line:

~~~bash
$ ssh blu-XYZ
~~~

to log in to your selected workstation. (If you are keen, you can go step further and set up a key file for password-less login.)

### Windows users without WSL/WSL2

Windows users who have not installed
[WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10) will
need to follow the
[instructions for remote sign on to CSIL from Windows](http://www.sfu.ca/computing/about/support/csil/how-to-remote-access-to-csil.html).

If you are using PuTTY, you will minimally need to configure your access to use port 24 (instead of the default 22).
It's a good idea to do the same as for Linux etc and create configurations with a the same shortened name (e.g., blu-b08) for your reserved Blusson machine.
Refer [here](https://www.ssh.com/ssh/putty/putty-manuals/0.68/Chapter4.html#config) for full details.

### Ensuring you are the only current user

**Note: From this point on, all the sample commands are to be executed
  remotely on the Blusson Hall machine, not on your local machine.**

You will need to be the only user signed in to your machine, ensuring
you exclusive use of its CPU and GPU. Hopefully, you have already arranged
with your clustered class-mates for this time/duration.  

To verify, begin by querying Linux for a list of all `sshd`
processes. This should return only the daemon `/usr/sbin/sshd` and two instances
of your LOGIN_NAME:

~~~bash
$ pgrep -a sshd
1741 /usr/sbin/sshd -D
9959 sshd: LOGIN_NAME [priv]
10093 sshd: LOGIN_NAME@pts/0
~~~

You also need to ensure that no one has signed on to Remote
Desktop. The following command should not show any output:

~~~bash
$ pgrep -a xrdp-chanserv
~~~

If another user has signed on to this machine, either via
SSH or Remote Desktop, make a note of the user name and pass it along to
your instructor/TA.

You can add the above commands to your .bashrc to automate/remind of this check.

## The architecture of the machine

The Dell Precision machines in the Blusson lab feature an 8-core Intel
Core&nbsp;i7 CPU. Hyperthreading is disabled, limiting the hardware to
8 concurrent threads, 1 per core:


~~~bash
$ lscpu
...
CPU(s):              8
On-line CPU(s) list: 0-7
Thread(s) per core:  1
...
~~~

The main memory has 32 GiB.

Each machine includes a single Nvidia GeForce RTX 2080 GPU with nearly
8&nbsp;GiB of on-card memory. This card uses Nvidia's
[Turing architecture](https://www.nvidia.com/content/dam/en-zz/Solutions/design-visualization/technologies/turing-architecture/NVIDIA-Turing-Architecture-Whitepaper.pdf). Data must be transferred from main
memory to the GPU card's memory before the GPU can process it, while
GPU results have to be transferred back to the main memory before they
can be used or output.

### Querying CPU utilization

This assignment requires you to query the utilization of the CPU cores and the
GPU. Two commands provide this information.

To query current CPU utilization, use `top`.  This will display a
system summary and a "leaderboard" of processes, sorted by descending
CPU utilization, updated every two seconds or so. Here's a screen shot
of the system when idle:

~~~
top - 15:21:25 up 12:10,  1 user,  load average: 0.00, 0.01, 0.00
Tasks: 241 total,   1 running, 168 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.0 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 32763692 total, 29319348 free,   891036 used,  2553308 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used. 31415720 avail Mem
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 1734 root      20   0 1498736  43744  24404 S   0.7  0.1   0:38.91 containerd
30245 ted       20   0   52216   4032   3280 R   0.3  0.0   0:00.12 top
    1 root      20   0  226464  10076   6740 S   0.0  0.0   0:21.89 systemd
...
~~~

The key columns are the ones labelled `%CPU` and `%MEM`.  In this
idle state, the program with the highest CPU utilization for the most
recent 2&nbsp;s, listed in the top row of the leaderboard, was `containerd`, consuming a
miniscule 0.7% of CPU and 0.1% of available memory.

The listed CPU percentage is for a single core. Given the eight cores
in these machines, maximum utilization is 800%. In a few
moments, you will see that training a PyTorch neural net can consume
close to this proportion of the machine.

### Querying GPU utilization

Use the `nvidia-smi dmon -s um` command to display snapshots of the GPU
utilization:

~~~bash
$ nvidia-smi dmon -s um
# gpu    sm   mem   enc   dec    fb  bar1
# Idx     %     %     %     %    MB    MB
    0     0     0     0     0    60    12
    0     0     0     0     0    60    12
    0     0     0     0     0    60    12
    0     0     0     0     0    60    12
    0     0     0     0     0    60    12
...
~~~

The relevant columns are `sm` (% utilization of GPU processor), `mem`
(% utilization of GPU memory bandwidth), and `fb` (GPU memory
allocated in MiB). Ignore the other columns.

### Use three `ssh` sessions to observe the performance

You will want to observe the CPU and GPU performance statistics as you train
several neural nets. To do this, you will need to open two more `ssh`
sessions to the same machine, where you can run `top` and `nvidia-smi`
while the main session is running the training.  Arrange the windows
with the training run in the largest window and performance monitors
of the host CPU and the GPU in smaller windows:

![Three ssh windows, the first showing a training run, the second showing `top`, the third showing `nvidia-smi dmon`](GPU-monitoring-screen.png)

Summarizing the five measures you will observe in this assignment:

Host CPU utilization (via `top`---range 0--800%)
: The proportion of time the 8 CPU cores are in active use.

Host memory allocation (via `top`---range 0--100%, as a proportion of 32&nbsp;GiB)
: The proportion of host memory allocated. All 8 processor cores share the same
memory, so the maximum is 100% rather than 800%. The main memory is
32&nbsp;GiB, so each 10% represents 3.2&nbsp;GiB.

GPU process utilization (via `nvidia-smi dmon`---range 0--100%)
: The proportion of time at least one thread is running on the
GPU. Informally, you can use this as a measure of "how much of the GPU
is in use".

GPU memory bandwidth utilization (via `nvidia-smi dmon`---range 0--100%)
: The proportion of time GPU memory is read to or written from.

GPU memory allocation (via `nvidia-smi dmon`---range 60--7973&nbsp;MiB)
: The amount of GPU memory allocated. The Linux system uses
60&nbsp;MiB at all times, so that is the minimal baseline before you
train any nets.

The two limit types affect the training differently:

* **Allocation** determines whether a model can be *trained at
  all*.  If a model is too large for the available memory, it cannot be
  trained, while if the model fits, the training can run.

* **Utilization** determines the *rate of training*. When a net is
  large enough to approach the maximum utilization for any single
  resource, that resource will limit the training rate for any larger
  nets.

## The sample neural net application

You will compare the performance of the CPU and GPU as you train
several neural nets using the PyTorch library. Fetch the code by clone
the assignment repo as follows *from a Blusson machine*:

**Due to the size of the data file, you will likely need to work from the scratch folder `/usr/shared/CMPT/scratch/$USERNAME`. Please change to this directory before following the instructions.**


~~~bash
$ git clone https://github.com/scp-2021-jan-cmpt-756/gpu-assignment
~~~

You are welcome to review the code if you like but you can complete this
assignment by simply running the application without reading it.

### Some machine learning basics

This assignment does not require deep
knowledge of machine learning but it helps to have some basic
background.

The sample nets classify images from the
[CIFAR10](https://www.cs.toronto.edu/~kriz/cifar.html) dataset,
comprising 60,000 small images, each 32 by 32 pixels of 24 bits, for a
size of 3,072 bytes.

Each image is labelled with exactly one of these ten classes:

* airplane
* automobile
* bird
* cat
* deer
* dog
* frog
* horse
* ship
* truck

The dataset is divided into 50,000 **training** images, 5,000 from each
class, and 10,000 **test** images, 1,000 from each class.

The net training is performed as a sequence of **epochs**.  In each
epoch, the net is first trained on the 50,000 training images, in
random order. The net is then run against the 10,000 testing images,
in a fixed order used for every epoch. The net's accuracy is computed
for the training images (only) and if this version is more accurate
than the previous best (or this is the first epoch), the net weights
are saved in subdirectory `checkpoint`. If the accuracy does not exceed the current best, this
epoch's net is discarded.

The epoch's training phase is computationally expensive due to the
back-propagation. The testing phase is much faster because it only
requires forward inference.

The sample program allows the user to run one of 16 neural nets but this assignment will only
use two:

NaiveNet
: A simple net that is fast to train but that never becomes very accurate.

RegNetX_200MF
: A [sophisticated net](https://arxiv.org/abs/2003.13678) that is slower to train but can become very
  accurate if trained for enough epochs. This net is the default.

## Run the naive net

This repository is derived from a project by
[Liu Kuang](https://github.com/kuangliu/), comparing the accuracy of
15 neural net designs on a classification task. We added a naive net,
taken from the
[PyTorch Tutorial](https://pytorch.org/tutorials/beginner/blitz/cifar10_tutorial.html#sphx-glr-beginner-blitz-cifar10-tutorial-py).
The naive net is faster to train but performs less accurate
classifications.

The main program is `./main.py`. Its arguments, options, and defaults can be
displayed by `./main.py -h`.

To familiarize yourself with the software, run a short (single-epoch)
training of the naive net on the CPU.
The first run will be slower as the system must download and unpack
CIFAR10 into the `./data` subdirectory. This step only needs to be done once.

~~~bash
$ ./main.py --ecount 1 --cpu NaiveNet
==> Preparing data from root directory "./data" ..
Downloading https://www.cs.toronto.edu/~kriz/cifar-10-python.tar.gz to ./data/cifar-10-python.tar.gz
170500096it [00:05, 34019363.30it/s]
Extracting ./data/cifar-10-python.tar.gz to ./data
Files already downloaded and verified

Epoch: 0
 [=========================== 391/391 ============================>]  Step: 5ms | Tot: 10s737ms | Loss: 1.959 | Acc: 26.182% (13091/50000)
 [=========================== 100/100 ============================>]  Step: 2ms | Tot: 687ms | Loss: 1.765 | Acc: 36.370% (3637/10000)
Saving..
Total time for 1 completed epochs   11.13 s, per epoch  11.13 s
Best test accuracy 36.37 %
~~~

The `--ecount` option specifies the number of epochs, while the
`--cpu` forces the training to be done entirely on the CPU. The training phase
processes the 50,000 training images in 391 batches of 128 images,
while the testing phase processes the 10,000 training images in 100
batches of 100 images.

The above run required 11.13&nbsp;s for the training. The testing phase found an
accuracy of 36.37% (3,637 images were correctly recognized out of
10,000) and the net weights were saved because this was the first epoch.

### Dealing with long training times

The first run used a small simple net that didn't approach any system
limit and ran quickly.
Training larger nets can require a long time, especially (spoiler alert!)
on a CPU. Here are some tips to keep the process manageable:

1. Control-C will give you the results to date: If the run is taking
   longer than you expected (or are willing to wait), you can press
   Control-C. This will interrupt the current epoch and present the
   timing results up to the *last complete* epoch, not counting the
   interrupted one. The most accurate net
   found so far will have been saved for possible resumption via
   `--resume`.
2. The `--resume` option will restart a saved net: The latest most
   accurate version of the two net types can be restarted by running
   with the `--resume` option.  This allows you to pause a session via
   Control-C, sign off the machine, and resume later. Because the checkpoints are stored
   in your home directory and the Blusson Hall machines are identical,
   you can even resume the training on a different machine and still get
   consistent results.

With this background out of the way, you are ready to start the assignment.

## Step 1: Train the naive net

Train the simple net, `NaiveNet`, on both the CPU (via `--cpu`) and the
GPU (the default).
Specify
enough epochs that you can accurately estimate the time / epoch for each hardware. Do not worry about how
accurate the net is---we will compare classification accuracy in a later step.  Record
the time / epoch for each case. Also record the typical values of the
five performance measures (host CPU utilization, host memory allocation, GPU utilization, GPU
memory bandwidth utlization, GPU memory allocation) listed above. The precise values of these
measures will fluctuate; simply record a value that seems close to the
median for the training phases of the run.

## Step 2: Train the sophisticated net

Train the sophisticated net, `RegNetX_200MF` (the default), on both the CPU and
GPU. Record the same performance statistics as you recorded for the
naive net.

## Preliminary analysis: What seems to be going on?

You can display the structures of the two nets via:

~~~bash
$ ./main.py --ecount 0 --print_net NaiveNet
...
$ ./main.py --ecount 0 --print_net
...
~~~

Focus on the relative sizes and complexities of the two nets. You do
not need to know or understand the details.

Take stock of what you've learned so far:

1. How much speedup did you see from the GPU on each of the nets? Did
   they speed up differently? If so, why?
2. In which cases did you see one or more utilizations limiting the
   rate?
3. Tentatively generalizing from these examples, what algorithms might
   benefit most from GPUs?

## Step 3: Train larger versions of `NaiveNet`

Now you're going to explore the effect of different sizes of the
`NaiveNet` design. The test program has an option, `--width`, that
specifies the width of the two convolutions in the network. Increasing
these widths may not improve the net much---its structure is too simple to
ever be very good---but it will make the net bigger.

**Stop. Think. What effects do you expect to see as the net size
  increases? Which factor is likely to be limit performance?**

The default width, which you used in Step&nbsp;1, is 6. Rerun the
tests on the GPU and CPU for varying widths of `NaiveNet`, including
`--width 1`. What is the largest width that will still fit in the GPU's
memory?  Do not run nets any larger than that.

To save time, run these tests for a single training epoch and record
that time. Also record the allocation and utilization statistics.

This is more exploration than science: You are looking to identify the
major effects on execution time, not to elaborate a precise theory.
Your numbers need only be approximate and one run of one epoch for a
given width is enough. The results from `top` and `nvidia-smi` will
fluctuate. The `%MEM` column of `top` is particularly volatile when
training a net on the CPU. Only record the values during the training,
not during the data load, model building, or testing phases.

The range of sizes is huge and running even one epoch on the CPU will
take time, so you will want to explore the range strategically.
Start by finding the largest width that the GPU can handle and then
explore the values between 1 and that maximum. Predict where you might
see a distinct change in performance and test that prediction. Analyse
your results as you go, choosing the next width to try.

## Step 4: Compare accuracies of several net designs

As a final step, explore how the different sizes of net affect
accuracy.  Make these runs longer, giving the training enough time to
stabilize. In this step, we are only concerned with accuracy, so do
all the training with the GPU to keep the runs as short as possible.

You do not have to achieve research-level accuracies, such as the
[94% accuracy Liu Kuang reported for `RegNetX_200MF`](https://github.com/kuangliu/pytorch-cifar). This
requires a systematic search of parameters such as the learning rate,
`lr`, and training the models for far longer than you will want to wait
(Liu Kuang trained the models for over 250 epochs). Your concern in
this assignment is simply with the big differences between models.

Train the following models until they stabilize and note their best
accuracy in the test phase, as printed out at the end of the run:

* The sophisticated net (`RegNetX_200MF`)
* The default width (6) for `NaiveNet`
* A range of non-default widths for `NaiveNet`

Questions:

1. Which model gives the best results?
2. What trends do you see as the `NaiveNet` width varies?

The following command will break down a model's accuracy by each image class (car
vs. dog. vs. frog ...) for the most recently-saved model of a given
type:

~~~bash
$ ./main.py --test_only [--width=WIDTH] [MODEL_TYPE]
==> Preparing data from root directory "./data" ..
Files already downloaded and verified
Files already downloaded and verified
==> Building model for MODEL_TYPE net ..
==> Loading checkpoint..
==> Resuming from epoch 1
 [=========================== 100/100 ============================>]  Step: Xms | Tot:XXms | Acc: XX%
Accuracy of airplane   : X%
Accuracy of automobile : X%
Accuracy of bird       : X%
Accuracy of cat        : X%
Accuracy of deer       : X%
Accuracy of dog        : X%
Accuracy of frog       : X%
Accuracy of horse      : X%
Accuracy of ship       : X%
Accuracy of truck      : X%
~~~

## Analysis

Analyze and write up these results, generalizing them. Consider the following:

1. What kinds of computations might gain most speedup from a GPU?
2. How did performance change as the CPU utilization increased (in
   CPU-based training)?  As the GPU utilization increased (in
   GPU-based training)?
3. What is the relative importance of model size and model design for
   accuracy? How might this apply more generally?

Support your answers with performance statistics from your runs.

## Report submission

### Create a PDF

Make a copy of the [submission template](https://docs.google.com/document/d/1uKqm4ScpwXQns-T2E7eIgLof2i3_L_weeheCxKuO3Nc/edit?usp=sharing)(GDoc format).

Fill in:

a. The header box at the top of the document.

b. Your analysis

Generate a PDF when you are done.

You must name your PDF according to the pattern: **SFU-student-no**`-a2-submission.pdf` where **SFU-student-no** is the  numeric id (typically 30...) assigned to you upon entering SFU. Unfortunately, you will be penalized for incorrect filename because of cascading dependencies for a large class.

**A penalty will be assessed for failure to name your submission appropriately.**

### Canvas submission

Navigate to this assignment and upload the generated PDF.
