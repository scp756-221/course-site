How many files are contained in the following commit:

```sh
$ mkdir my-repo
$ cd my-repo
$ cp ~/path/to/prewritten/text.txt README.txt
$ git init
$ git add .
$ git commit -m "Lorem ipsum"
$ echo "*.txt" > .gitignore
```

One
Two
None
===


How many files are contained in the following commit:

```sh
$ mkdir my-repo
$ cd my-repo
$ cp ~/path/to/prewritten/text.txt README.txt
$ git init
$ touch .gitignore
$ git add .
$ git commit -m "Lorem ipsum"
```

Two
One
None
===





Consider the life of project that proceeded roughly as follows:

```sh
$ mkdir cool-project
$ cd cool-project
$ git init
$ scp buddysbox:/remote/path/to/friends/code/ .
$ git add .
$ git commit -m "Sed ut perspiciatis unde omnis"
$ git checkout -b buddys-idea
$ cp ~/my/own/ideas .
$ cp ~/templates/.gitignore .
$ git add .
$ git tag -a my-starting-point -m "Initial starting point"
$ git commit -m "Lorem ipsum".
$ git checkout -b my-direction
# do a bunch of test
$ git add .
$ git commit -m "Quis autem vel eum iure"
$ git checkout buddys-idea
$ scp buddysbox:/another/path/to/more/friends/code/ .
$ git add .
$ git commit -m "Excepteur sint occaecat cupidatat"
$ scp buddysbox:/another/path/to/more/friends/doc/*.txt .
$ git add .
$ git commit -m "Quod erat demonstrandum"

```

How many commits are there in this repo altogether?

Five
Four
Three
Two


How many branches are there in this repo altogether?

Three
Two
One


How many commits are there in your own branch (my-direction)?

Two
One 
Three

How many commits are there in your buddy's branch (buddys-idea)?

Three
Two
One 
