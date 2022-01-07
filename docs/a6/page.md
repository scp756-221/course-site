## Introduction

In this assignment you will explore the range of speedups enabled by
graphics processing units
(GPUs). These are just one example of the growing range of
hardware designed to accelerate specialized computations. Other
approaches include

* Google's
  [Cloud Tensor Processing Units (TPUs)](https://cloud.google.com/tpu/docs/tpus) for accelerating the vector and matrix computations at the heart of machine learning algorithms,
* Datacenters featuring
  [Application-Specific Integrated Circuits (ASICs)](https://dl-acm-org.proxy.lib.sfu.ca/doi/10.1109/ISCA.2016.25) for
  algorithms such as video transcoding and deep learning,
* Alibaba's [FPGA-accelerated instances](https://www.alibabacloud.com/help/doc-detail/25378.htm#d22e35)
  for accelerating algorithms specific to a customer's domain,
* Amazon [DL1 instances](https://aws.amazon.com/ec2/instance-types/dl1/) featuring
  [Habana Gaudi AI processors](https://habana.ai/aws-launches-ec2-dl1-instances/) for training
  machine learning models,
* Amazon [Trn1 instances](https://aws.amazon.com/ec2/instance-types/trn1/),
  featuring [Trainium chips](https://aws.amazon.com/machine-learning/trainium/) for training machine learning models, and
* Amazon [Inf1 instances](https://aws.amazon.com/ec2/instance-types/inf1/), featuring [Inferentia chips](https://aws.amazon.com/machine-learning/inferentia/) for running inference on previously-trained models.

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

### Controlling costs

In this assignment, you will run some of the more expensive EC2 instances, costing US$&nbsp;0.526&nbsp;/&nbsp;hr and more. You can minimize your expenses by planning the assignment. Review the whole assignment completely before executing any commands, so that you know the assignment's structure and steps before you start.

## Manage EC2 instances using the command line

You will manage your EC2 instances using a set of command-line tools, predefined in the `profiles` subdirectory and described in `profiles/README-aws.md`. These tools simplify working with EC2 instances, avoiding the busywork required by the EC2 Web console.

The commands require a small amount of one-time setup, modifying file `profiles/ec2.mak` to define the following variables:

  * `SGI_WFH`
  * `KEY`
  * `LKEY`

See `profiles/README-aws.md` for the details.

The commands must be enabled every time you start a new instance of the tools container:

~~~bash
/home/k8s# . tools/profile
~~~

### Start a GPU instance

Once you have set up the EC2 commands, start a GPU-enabled EC2 instance
by running the following commands:

~~~bash
/home/k8s# epkg gpu_small
/home/k8s# # Wait 1--2 minutes for the instance to completely initialize
/home/k8s# esn NAME ec2-user
The authenticity of host 'ec2-54-191-192-143.us-west-2.compute.amazonaws.com (54.191.192.143)' can't be established.
ECDSA key fingerprint is SHA256:gMoqq/rEr7T6VoyVfgnceMmv0RzfaxqNC9+kVxQOopE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ec2-54-191-192-143.us-west-2.compute.amazonaws.com,54.191.192.143' (ECDSA) to the list of known hosts.
~~~

where `NAME` is the "adjective_name" name generated for your instance,
such as `fervent_lichterman`.  If the `esn` command fails, it is likely because the instance
has not fully started. Wait 15 seconds or so and try again.

Once the `esn` command has completed, sign on to the instance:

~~~bash
/home/k8s# esshn NAME ec2-user
... signon messages ...
[ec2-user@NAME/ec2 ~]$ 
~~~

The prompt includes some useful reminders: the signon userid, the name of the machine, and that this session
is a remote EC2 instance.

## The architecture of the machine

The `g4dn.xlarge` instance created by the `gpu_small` reques features a CPU with the following parameters:

* x86_64 Intel Cascade Lake CPU
* 4 vCPUs on 2 Cores
* 16 GiB of main memory

The `lscpu` command provides more detail:

~~~bash
[ec2-user@fervent_lichterman/ec2 ~]$ lscpu
Architecture:        x86_64
...
CPU(s):              4
On-line CPU(s) list: 0-3
Thread(s) per core:  2
Core(s) per socket:  2
Socket(s):           1
...
L1d cache:           32K
L1i cache:           32K
L2 cache:            1024K
L3 cache:            36608K
...
~~~

Each instance also features a single [Nvidia T4 GPU](https://www.nvidia.com/en-us/data-center/tesla-t4/) based on
Nvidia's [Turing architecture](https://www.nvidia.com/content/dam/en-zz/Solutions/design-visualization/technologies/turing-architecture/NVIDIA-Turing-Architecture-Whitepaper.pdf)
with the following parameters:

* 16 GiB of GPU memory,
* 320 Turing Tensor cores, and
* 2,560 CUDA cores.

The main and GPU memories are separate.
Data must be transferred from main
memory to the GPU's memory before the GPU can process it, while
GPU results have to be transferred back to the main memory before they
can be used or output.

This assignment requires you to query the utilization of the CPU cores and the
GPU. Two commands provide this information.

### Query CPU utilization

Use `top` to query current CPU utilization.  This will display a
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

The listed CPU percentage is for a single core. Given the two cores
in these machines, maximum utilization is 200%. In a few
moments, you will see that training a PyTorch neural net can consume
nearly the full utilization of the machine.

Press `q` to exit `top`.

### Query GPU utilization

Use the `nvidia-smi dmon -s um` command to display snapshots of the GPU
utilization:

~~~bash
[ec2-user@angry_cray/ec2 ~]$ nvidia-smi dmon -s um
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

Press `^C` to exit `nividia-smi dmon`.

## Set up the environment

To run the sample machine learning application, you must first activate the [`conda`
environment](https://docs.conda.io/en/latest/) containing the required libraries:

~~~bash
[ec2-user@angry_cray/ec2 ~]$ source activate pytorch_latest_p37
(pytorch_latest_p37) [ec2-user@angry_cray/ec2 ~]$ 
~~~

Conda adds `(pytorch_latest_p37)` to the prompt to mark the active
Python environment.

### Manage your costs

**Note:  The EC2 instance continues to run whether or not you are
signed in to it. The GPU instances are relatively expensive to run
(US$&nbsp;0.526&nbsp;/&nbsp;hr and up), so you want to kill an instance as soon as you are done with it.**

Kill an instance by running the following in your own machine, not the EC2 instance, replacing `NAME` with the "adjective_name" of your instance:

~~~bash
/home/k8s# ekn NAME
~~~

## The sample neural net application

You will compare the performance of the CPU and GPU as you train
several neural nets using the PyTorch library. Clone
the assignment repon and `cd` into it:

~~~bash
[ec2-user@angry_cray/ec2 ~]$ git clone https://github.com/scp-2021-jan-cmpt-756/gpu-assignment
[ec2-user@angry_cray/ec2 ~]$ cd gpu-assignment
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

The epoch's training phase is computationally expensive due to
back-propagation. The testing phase is much faster because it only
requires forward inference.

The sample program allows the user to run one of 16 neural nets but this assignment will only
use two:

*NaiveNet*
: A simple net that is fast to train but that never becomes very accurate.

*RegNetX_200MF*
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

To familiarize yourself with the software, you will first run a short (single-epoch)
training of the naive net on the CPU.
This first run will have a long delay before starting to train the net because the CPU must establish a connection to the GPU and initalize it. This step can take as long as several minutes. Once the connection has been established, future runs can simply reuse it.

~~~bash
(pytorch_latest_p37) [ec2-user@fervent_lichterman/ec2 gpu-assignment]$ ./main.py --ecount 1 --cpu NaiveNet
... long delay here ...
==> Preparing data from root directory "./data" ..
Downloading https://www.cs.toronto.edu/~kriz/cifar-10-python.tar.gz to ./data/cifar-10-python.tar.gz 170499072it [00:03, 47019815.60its]
Extracting ./data/cifar-10-python.tar.gz to ./data
Files already downloaded and verified
==> Building model for NaiveNet net ..

Epoch: 0
 [=========================== 391/391 ============================>]  Step: 9ms | Tot: 9s585ms | Loss: 1.942 | Acc: 26.208% (13104/50000)
 [=========================== 100/100 ============================>]  Step: 11ms | Tot: 1s162ms | Loss: 1.801 | Acc: 32.840% (3284/10000)
Saving..
Total time for 1 completed epochs   16.63 s, per epoch  16.63 s
Best test accuracy 32.84 %
~~~

The `--ecount` option specifies the number of epochs, while the
`--cpu` forces the training to be done entirely on the CPU. The training phase
processes the 50,000 training images in 391 batches of 128 images,
while the testing phase processes the 10,000 training images in 100
batches of 100 images.

The above run required 16.63&nbsp;s for the training. The testing phase found an
accuracy of 32.84% (3,284 images were correctly recognized out of
10,000) and the net weights were saved because this was the first epoch.

### Dealing with long training times

The first run used a small simple net that didn't approach any system
limit and ran quickly, once the data and GPU had been initialized.
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

## Use three sessions to observe the performance

You will need two more terminal windows into this EC2 instance.
In two other windows, run the same `esshn NAME ec2-user` you ran
above. In one window run `top`, in the other run `nvidia-smi dmon -s um`.
Arrange the windows
with the training run in the largest window and performance monitors
of the host CPU and the GPU in smaller windows:

![Three ssh windows, the first showing a training run, the second showing `top`, the third showing `nvidia-smi dmon`](GPU-monitoring-screen.png)

Summarizing the five measures you will observe in this assignment:

Host CPU utilization (via `top`---range 0--200%)
: The proportion of time the 2 CPU cores are in active use.

Host memory allocation (via `top`---range 0--100%, as a proportion of 16&nbsp;GiB)
: The proportion of host memory allocated. All processor cores share the same
memory, so the maximum is 100% . For machines whose main memory is
16&nbsp;GiB, each 10% represents 1.6&nbsp;GiB.

GPU process utilization (via `nvidia-smi dmon`---range 0--100%)
: The proportion of time at least one thread is running on the
GPU. Informally, you can use this as a measure of "how much of the GPU
is in use".

GPU memory bandwidth utilization (via `nvidia-smi dmon`---range 0--100%)
: The proportion of time GPU memory is read to or written from.

GPU memory allocation (via `nvidia-smi dmon`---range 3--15100&nbsp;MiB)
: The amount of GPU memory allocated. The Linux system uses
3&nbsp;MiB at all times, so that is the minimal baseline before you
train any nets.

The two limit types affect the training differently:

* **Allocation** determines whether a model can be *trained at
  all*.  If a model is too large for the available memory, it cannot be
  trained, while if the model fits, the training can run.

  When recording allocations, we're only concerned with whether the
  value is close to the total available.  Simply record whether the value
  is above or below 90%.

* **Utilization** determines the *rate of training*. When a net is
  large enough to approach the maximum utilization for any single
  resource, that resource will limit the training rate for any larger
  nets.

## Step 1: Train the naive net

Train the simple net, *NaiveNet*, on both the CPU (with the `--cpu` option) and the
GPU (without that option).
For nets that compute an epoch quickly, specify
around 5 epochs to help accurately estimate the time / epoch.  For nets that train more slowly, 1 epoch
is enough. Do not worry about how
accurate the net is---we are only concerned with relative speeds.  Record
the time / epoch for each case. Also record the typical values of the
five performance measures (host CPU utilization, host memory allocation, GPU utilization, GPU
memory bandwidth utlization, GPU memory allocation---but recall that for allocations, we're only concerned if they are above or below 90%) listed above. The precise values of these
measures will fluctuate; simply record a value that seems close to the
median for the training phases of the run.  We are interested in big-picture differences, not subtle variations.

Record the values in the top row of Table&nbsp;1 of the BLERG document URL and they need to make a copy or what?
Pay attention to the rounding and recording requirements.  We are looing for high-level patterns, which can be obscured by recording too many figures.

The first time you run a net on the GPU, it has to be downloaded, another tedious process.
You can watch the GPU memory slowly increase in the `nvidia-smi` window.  When the epoch actually begins, the memory required will stabilize and the GPU utilization will rise.

Note that when training on the CPU, the most important values are the ones returned by `top` (host CPU utilization, host memory allocation), while when training on the GPU, the most important values are the ones returned by `nvidia-smi` (GPU utilization, GPU memory bandwidth utilization, GPU memory allocation).

## Step 2: Train the sophisticated net

Train the sophisticated net, *RegNetX_200MF* (the default), on both the CPU and
GPU. Record the same performance statistics as you recorded for the
naive net.  Given how long it takes to train this net on a CPU, it is OK to base your estimate of the epoch time using just a single epoch.

Note: Training this net on the CPU will require tens of minutes, by far the longest run of this assignment.

Add your data to the bottom row of Table&nbsp;1 of the BLERG document

## Preliminary analysis: What seems to be going on?

You can display the structures of the two nets via:

~~~bash
[ec2-user@fervent_lichterman/ec2 gpu-assignment]$ ./main.py --ecount 0 --print_net NaiveNet
...
[ec2-user@fervent_lichterman/ec2 gpu-assignment]$ ./main.py --ecount 0 --print_net RegNetX_200MF
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

## Step 3: Train larger versions of *NaiveNet*

Now you're going to explore the effect of different sizes of the
*NaiveNet* design. The test program has an option, `--width`, that
specifies the width of the two convolutions in the network. Increasing
these widths may not improve the naive net much---its structure is too simple to
ever be very good---but it will make the net bigger.

**Stop. Think. What effects do you expect to see as the net size
  increases? Which factor is likely to be limit performance for the CPU?  For the GPU?**

Run the tests on the CPU and GPU, using the widths specified in Table&nbsp;2 of the BLERG document.  Enter your results in the x86 columns of that table.

To save time, run these tests for a single training epoch and record the epoch time and allocation and utilization statistics for that run.

In addition to the above, find a single width above 5,000 that 
is too large for the GPU&mdash;it fails before running a single epoch.  You do not have to find a tight upper bound, just a value too large to be allocated in the GPU.

Enter the unsuccessful width in the x86 column of Table&nbsp;3 of the BLERG document.

Once you have all the data, you can terminate this instance, as you will not be using it again. See "Managing your costs" above for how to terminate an instance.  The 
"Terminating instances" section of `profiles/README-aws.md` describes additional methods.

## Step 4: Compare with a different architecture

As a final step, we will gather timings from an instance running a different CPU architecture, the [AArch64 architecture](https://en.wikipedia.org/wiki/AArch64), more commonly referred to by the shorter "ARM64" or even just "ARM".

### Amazon's Graviton ARM processor and `g5g.2xlarge` instance

We will use Amazon's implementation of this architecture, called [Graviton](https://aws.amazon.com/ec2/graviton/), in a `g5g.2xlarge` instance. Amazon charges about the same for this instance as the x86 instance used in the previous step and it features the following parameters:

 * ARM64 Graviton2 CPU
 * 8 vCPUs on 8 cores
 * 16 GiB of main memory

 The `lscpu` command provides more detail:

~~~
[ubuntu@suspicious_spence/ec2 ~]$ lscpu
Architecture:                    aarch64
...
CPU(s):                          8
On-line CPU(s) list:             0-7
Thread(s) per core:              1
Core(s) per socket:              8
Socket(s):                       1
...
L1d cache:                       512 KiB
L1i cache:                       512 KiB
L2 cache:                        8 MiB
L3 cache:                        32 MiB
~~~

There are two immediately-visible differences from the x86 instance we used in the preceding step:

1. The ARM instance has four times as many cores as the x86 instance (8 versus 2)
2. The ARM instance has much larger L1 and L2 caches than the x86 instance (16 and 8 times, respectively), though their L3 caches are similar.

Each instance also features a single Nvidia T4G, essentially the same design as the T4 included in the x86 instance we tested in the last section, with minor differences to interface with Graviton processors.

In summary, this instance provides four times as much parallelism available when we train on the CPU but essentially the same parallelism when training on its GPU.

### Start an ARM instance

You start this instance using essentially the same commands as for an x86 instance, with a few differences:

* The package command is `armpkg`.
* The userid is `ubuntu`.
* The conda activation command is `source activate base`.

Here are the commands, where `NAME` is the name assigned to the instance upon its creation:

~~~bash
/home/k8s# armpkg gpu_small
...
/home/k8s# # Wait 1--2 minutes for the instance to completely initialize
/home/k8s# esn NAME ubuntu
...
/home/k8s# esshn NAME ubuntu
... signon messages ...
[ubuntu@suspicious_spence/ec2 ~]$ source activate base
(base) [ubuntu@suspicious_spence/ec2 ~]$ git clone https://github.com/scp-2021-jan-cmpt-756/gpu-assignment
(base) [ubuntu@suspicious_spence/ec2 ~]$ cd gpu-assignment
~~~

### Train *NaiveNet* on the ARM instance

Set up the three windows to this instance in the same configuration as the last one.

Do a single-epoch training of *NaiveNet* on the GPU and ignore the results.  For ARM instances, the timing on the first GPU run includes the GPU setup time, which we do not want to include.

After the initial throwaway run, train *NaiveNet* for the same width values as you did for the x86 instance, recording the values in the ARM columns of Tables&nbsp;2 and 3.  You do not need to do any runs with *RegNetX_200MF*.

## Cleanup and cost control

Once you have gathered all the above timings, terminate the ARM instance.  The surest way to guarantee that you have no instances running in the background is to run the `epurge` command, which terminates *all* your EC2 instances in the current region:

~~~bash
/home/k8s# epurge
...
/home/k8s# eps
i-0735372c21aca9b7e g4dn.xlarge shutting-down 54.191.192.143 fervent_lichterman
~~~

## Is this realistic?  Our future of diverse processors

Cloud providers are moving to a diverse range of processor architectures, with Amazon aggressively leading.  When we ran this exercise just a year ago, in Spring 2021, AWS's machine learning-focused instances only featured x86 CPUS and Nvidia GPUs. Just one year later, Spring 2022, AWS offers two general CPU architectures (x86 and ARM) and *four* architectures featuring specialized forms of massive parallelism (NVidia GPU, Intel Habana Gaudi, AWS Inferentia, and AWS Tranium).

Although other cloud providers are not moving as quickly in this direction, focused on x86 as their general CPU
architecture and Nvidia GPUs for massive parallelism, they are also increasing their architectural diversity.
Google is adding Google TPUs, Apple is converting to the
[ARM-based M1 processor design](https://en.wikipedia.org/wiki/Apple_M1),
and other vendors are adopting the [RISC V](https://en.wikipedia.org/wiki/RISC-V) architecture.  It seems
inevitable that we will see increasing diversity of processors. Effective system design will require matching
your application requirements to the strengths, weaknesses, and costs of the multiple architectural
families&mdash;the skills developed in this assignment.

## Analysis

Our analysis in this assignment is not intended to be definitive or detailed. This sort of test is an initial exploration to determine what factors *might* be worth further investigation and what architectural choices might not apply. We are not looking for regression analyses but simply for general guidelines. In practice, results such as these would guide further, more detailed work, including multiple runs rather than the single runs we made.

Our intent here is just to give some experience of the kinds of issues that arise when matching an application to the machine architectures offered by a given cloud provider.

Answer the questions in BLERG.

## Report submission 

BLERG Use PDF below?

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
