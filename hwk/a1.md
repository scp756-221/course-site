---
title: Assignment 1/2
style:
  background: red
---

## Introduction

This assignment provides practice working with approximate set
cover. It comprises several steps:

1. Identify an example of the set cover problem in a domain of your
   choice.
2. Implement the greedy set cover algorithm in Python.
3. Test your implementation on representative datasets that we provide.
4. Compare the results of your approximate algorithm to their optimal
   results.

On completing this assignment, you will be able to connect this
module's material on approximation algorithms to problems in a real-world setting.

## 1. Identify an example of the set cover problem

The purpose of this part of the assignment is to provide an opportunity to see
how an abstract problem (minimal set cover) might arise in a real-world context.

Identify an example of the set cover problem in a domain of your choice.
The dataset for this does not need to exist already. In fact, we do not want you
to reuse/describe proprietary datasets that you may have used. If you want
to use such a domain, design a comparable dataset that captures the idea in spirit
but not the details.

Your dataset does not have to actually exist but must be something that
could realistically exist.  (A dataset of the speed records for all
[starships named in Ian M. Banks novels](https://concord.fandom.com/wiki/Iain_M._Banks_Culture_Universe_Spaceship_Names)
is *not* acceptable.)

The size of your dataset should have at least 200 items in
its universe and at least 1000 subsets of that universe from which the
cover will be constructed.  Your dataset can be substantially larger
if you like---this requirement is simply to rule out "toy" problems
readily solvable by the naive optimal algorithm.

### Describing the dataset

Your description should have the following structure. You are welcome
to use the titles as section headings in your description:

#### 1. Domain

What is the application domain in which your dataset would be used?
Banking?  Fraud detection?  Drug design?  This section will
typically be a sentence or two.

#### 2. Universe

The basic "items" of your universe.  These will be distinct instances of a
common type.  In real estate, they might be addresses or postal
codes. In fraud detection, they might be credit card accounts. In drug
design, they might be potential binding sites.

You may want to use a table to write out example items of your universe.

#### 3. Collection of subsets

The subsets from which the cover will be computed. These subsets could
be lists of postal codes, binding sites, and so forth. But set
covering problems may also arise in terms of tags or attributes.
Suppose that there are 1,000 tags that might potentially be assigned
to an individual postal code but each tag will only be assigned to at
most 5 codes. In this case, each tag identifies a subset of the postal
codes.

Again, a table may be useful to showcase a few representative subsets of your collection.

#### 4. Application of minimal set cover

Given the universe and collection of subsets you described, what use could
be made in the application domain for minimal set cover? For example,
the smallest set of tags such that every postal code in the dataset is
covered by at least one tag. If tags indicate marketing pitches that
succeeded in a given postal code, minimal set cover would tell you the
smallest number of successful pitches that would cover
every postal code.

Again, your application doesn't need to be super-realistic or
represent an actual application. It does need to be reasonable.

Aim for a description here of 3-500 words.


## 2. Implement the greedy algorithm for approximate set cover

Implement the greedy algorithm for unweighted set cover (presented in class, also summarized in
[the Wikipedia page on set cover](https://en.wikipedia.org/wiki/Set_cover_problem#Greedy_algorithm))
in Python.


We use GitHub Classroom for this assignment. Navigate to the following URL
from a browser to join the CMPT 756 assignment classroom and access the assignment template:

~~~
https://classroom.github.com/a/wqGAHnYZ
~~~

The template includes a module for reading test data sets from
[Beasley's OR Library](http://people.brunel.ac.uk/~mastjjb/jeb/orlib/scpinfo.html),
a main routine, command-line argument parsing, and some printing
utilities.

Implement the greedy set cover algorithm as the body of function
`set_cover()`. See the `README.md` file for the details of programming
and running the code.

We encourage you to implement the algorithm to emphasize conciseness
rather than raw execution efficiency. Your algorithm only needs to
process datasets that fit entirely in memory. Feel free to use any
Python features or standard Python libraries (but *not* libraries that
require installation) to make your function as straightforward as
possible.

Test your implementation on any of the datasets provided in the
template repository.

## 3. Run your implementation on the sample data sets

The repo we provided includes 16 data sets named `worst-N.txt`, where
`N` is an integer. Run your implementation of greedy set cover on each
one. None of them should take very long to process---typically seconds
on a recent laptop. Only print the time and number of sets in the
solution. For example, for the `worst-8.txt` dataset:

~~~
$ ./set-cover-approx.py --check --skip_print worst-8.txt
~~~

Save the printed values (the time and solution size) in a file for entry
in the table you create in the next step.

## 4. Compute the degree of approximation

The following table presents the sizes of the optimal
solutions. **Note:** If your implementation found smaller solutions,
it has a bug.

Fill in the missing values in this table.
We have ran the optimal solution on these files on your behalf, saving you the tedium
of waiting for its long runs to complete. Fill in the corresponding
values for your greedy algorithm.  The times will only be broadly
comparable because the optimal and greedy runs were on different CPUs
but they will be good enough for our purposes. See the Appendix for a
description of the machine on which the optimal timings were obtained.

**Note:** The values for the three left columns are also in file
`table.csv` in the assignment repository. You can use this as a
starting point for your own table, if you wish.

<table>
  <thead>
    <tr>
      <th style="text-align: left">Filename</th>
      <th style="text-align: right">Time, Optimal alg. (s)</th>
      <th style="text-align: right">Size, Optimal alg. (sets)</th>
      <th style="text-align: right">Time, Greedy alg. (s)</th>
      <th style="text-align: right">Size, Greedy alg. (sets)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: left">worst-3</td>
      <td style="text-align: right">0.001</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-4</td>
      <td style="text-align: right">0.001</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-5</td>
      <td style="text-align: right">0.001</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-6</td>
      <td style="text-align: right">0.001</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-7</td>
      <td style="text-align: right">0.002</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-8</td>
      <td style="text-align: right">0.005</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-9</td>
      <td style="text-align: right">0.011</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-10</td>
      <td style="text-align: right">0.063</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-11</td>
      <td style="text-align: right">0.222</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-12</td>
      <td style="text-align: right">0.936</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-13</td>
      <td style="text-align: right">2.602</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-14</td>
      <td style="text-align: right">12.227</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-15</td>
      <td style="text-align: right">56.141</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-16</td>
      <td style="text-align: right">230.723</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-17</td>
      <td style="text-align: right">1050.842</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
    <tr>
      <td style="text-align: left">worst-18</td>
      <td style="text-align: right">4051.983</td>
      <td style="text-align: right">2</td>
      <td style="text-align: right"> </td>
      <td style="text-align: right"> </td>
    </tr>
  </tbody>
</table>

Write a brief comparison of your results versus the optimal
solutions. Include a table with the columns, "Dataset", "Optimum solution
size", "Solution size of approximation", and "Theoretical bound".  The "theoretical
bound" column should list the bound on the size of the approximate
solution for the dataset. Compute the theoretical bound using the
bound presented in class.

## Submission

There is only one explicit submission for this assignment: your report. The code will be submitted (collected) via GitHub Education. This Canvas assignment is used for the report.

### Code submission

Your code will be automatically collected when the assignment is due.

### Report submission

### Create a PDF

Make a copy of the [submission template](https://docs.google.com/document/d/1pQb7GRvvacvXyrGi1D0avu2jbENzaWvJSvcDVn0hnN0/edit?usp=sharing)(GDoc format).

Fill in:

a. The header box at the top of the document.

b. Description of an application for minimal set cover (the
   first part of this assignment)

c. Summary of the results of running your application (the latter part of this
   assignment).

Generate a PDF when you are done.

You must name your PDF according to the pattern: **SFU-student-no**`-a1-submission.pdf` where **SFU-student-no** is the  numeric id (typicall 30...) assigned to you upon entering SFU. Unfortunately, you will be penalized for incorrect filename because of cascading dependencies for a large class.

**A penalty will be assessed for failure to name your submission appropriately.**


#### Canvas submission

Navigate to this assignment and upload the generated PDF.


## Appendix

The machine on which timings for the optimal algorithm were obtained:

[Intel Xeon Ivy Bridge 2.4&nbsp;GHz](https://en.wikipedia.org/wiki/Xeon)(specifically
a [2695 V2](https://www.intel.ca/content/www/ca/en/products/processors/xeon/e5-processors/e5-2695-v2.html))

Only a single core was used for the runs and it was dedicated to
running the algorithm.
