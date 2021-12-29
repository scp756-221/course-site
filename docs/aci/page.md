# Assignment 7&mdash;Continuous integration, Git branches, and testing

## EARLY DRAFT

This assignment demonstrates the combined use of several distinct but related tools:

* **Git branches**, which support development along multiple independent  paths, by one or several developers;
* **Static checking**, which ensures that a system's code conforms to syntax, style, and safety requirements;
* **Testing frameworks**, which support accumulating a consistent and broad test suite;
* **Development platforms**, such as GitHub, which provide a central site to merge branches; and
* **Continuous integration**, which reduce the risk that merged changes introduce defects.

This list is incomplete—each of these tools supports additional use cases and outcomes, while other tools are also used to achieve the above goals.

`flake8` Syntax and style changing.


## Step 1: Add your changes

~~~bash
/home/k8s # cd ci/v1.1
/home/k8s/ci/v1.1 # mv a7_music.py music.py
/home/k8s/ci/v1.1 # mv a7_test_music.py test_music.py
/home/k8s/ci/v1.1 # cd ../../s2/v1.1
/home/k8s/s2/v1.1 # mv a7_app.py app.py
~~~

* Run it locally
* Commit and push and watch

## Step 2:  Add `other_dev`'s changes

~~~bash
/home/k8s/ci/v1.1 # git checkout -b other_dev HEAD~
/home/k8s/ci/v1.1 # ls
~~~

You've rolled back time!

Do the same operations, but with `a7_other_dev_*` files.

~~~bash
/home/k8s/ci/v1.1 # mv a7_other_dev_music.py music.py
/home/k8s/ci/v1.1 # mv a7_other_dev_test_music.py test_music.py
/home/k8s/ci/v1.1 # cd ../../s2/v1.1
/home/k8s/s2/v1.1 # mv a7_other_dev_app.py app.py
~~~

* Run it locally
* Commit and push and watch

## Interlude: The levels of the CI tests

This is where we talk about structure, right?

* `runci-local.sh v1.1` (bash) Top-level script
  * `docker` (Go) Client
    * `dockerd` (Go) Server managing container images
  * `runci.sh` (bash) Mid-level script
    * `flake8` (Python) Static code checker
    * `docker-compose` (Go) Manage services and networks
      * `ci_test` (Python) Application running `pytest` framework
        * `test_music` (Python)) Test collection
          * `music.py` (Python) Client library for Music service
            * [via HTTP] `ci_s2` (Python) Music service
              * [via HTTP] `ci_db` (Python) Database interface layer
                * [via HTTP] `DynamoDB` (Java) Data storage layer

But note that we only have to attend to our layer, the one above it, and the one below.

## Step 3: Merge your changes with `other_dev`'s changes

~~~bash
/home/k8s # git checkout master
/home/k8s # git merge other_dev
~~~

Do the drill:

* Run it locally
* Commit and push and watch

Anything more?  See any possible problems?  Anything missing?

## The solution

Seriously&mdash;think about this!

## Step 4: Write a test that calls both

Use a song with multiple previous artists.

~~~python
def test_full_cycle(mserv):
    # `mserv` is an instance of the `Music` class

    # Performance at 2010 Vancouver Winter Olympics
    song = ('k. d. lang', 'Hallelujah')
    # Soundtrack of first Shrek film (2001)
    orig_artist = 'Rufus Wainwright'
    # Original recording from album "Various Positions" (1984)
    orig_orig_artist = 'Leonard Cohen'

    # Create a music record and save its id in the variable `m_id`
    # ... Fill in the test ...

    # The last statement of the test
    mserv.delete(m_id)
~~~

And ... we do the drill again:

* Run it locally
* Commit and push and watch

## Summary

The summary goes here

## Submission

BLERG


And here I go, aiming for inspiration. Fatigued, not doing things.

The layers.  The purpose of this assignment:

* Show the connection between

GitHub is the tool where

* Continuous integration integrates and does and is
* Testing, that thing, occurs
* Branches are integrated. The work of multiple developers are
  integrated and tested.
* GitHub is where the action is
