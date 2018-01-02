+++
title = "Setting Up Sadatrocks 🌏"
date = 2017-12-14T16:04:18+06:00
description = "Building a pipeline for SadatRocks"
bref = "This is how I build a pipeline for SadatRocks"
tags = ["hugo","travis-ci","pipeline"]
categories = ["devops"]
weight = 20
draft = false
+++

# Prologue

SadatRocks is my platform for writing blogs and manage my cookbooks. I love 💝
[hugo](http://gohugo.io/), which is The world’s fastest framework for building
websites. So I used [kube](https://github.com/jeblister/kube) as a theme to
develop SadatRocks with hugo.

Then I need to come up with a plan for building a pipeline for SadatRocks.

# The plan

I am going to use User/Organization Page strategy here for hosting my static
files with github. So I have to put my files into a branch named other than
master, which in my case happens to be `sources`.

On my code push to `sources` will trigger a build with [Travis CI](travis-ci.org),
which will then build my static files and place it into fresh `master`.

Last I need to copy my .travis.yml and README.md files to my `master` branch.

# Deployment

I am deploying a directory as a git branch. So I am going to use a script developed
by [dan smith](https://github.com/X1011) for this purpose.

To get the script and change it's permissions we need to run the following in project
root directory

```shell
$ wget https://github.com/X1011/git-directory-deploy/raw/master/deploy.sh && chmod +x deploy.sh
```

# Token generation and Encryption

Next we need to generate a token for our github repo with `public_repo` or `repo`
scope and Token description as `<ORGANIZATION>.github.io` from [here](https://github.com/settings/tokens).

Now that we have our token, we need to encrypt it with travis-ci. For that purpose
we need travis on our workstation. I am using `ruby 2.4.1p111 (2017-03-22 revision 58053) [x64-mingw32]`

To install Travis

```shell
$ gem install travis
$ travis version
```

I am using `1.8.8`. To encrypt the token and add it your project .travis.yml
file, run the following command in the project root directory

```shell
$ travis encrypt GIT_DEPLOY_REPO=https://<TOKEN>@github.com/<USERNAME>/<REPO>.git --add
```

# Setup .travis.yml

Setup your .travis.yml file with the following code and change where appropriate

```sh
env:  
  global:
    - secure: "..."                     # replace this with the travis encryption output!
    - GIT_DEPLOY_DIR=public             # default output dir of Hugo (change it, when you use configured it!)
    - GIT_DEPLOY_BRANCH=master          # target branch, replace by "gh-pages" for Github Project Pages
    - GIT_DEPLOY_USERNAME="Travis CI"   # dummy name
    - GIT_DEPLOY_EMAIL=user@example.com # replace by your email

branches:
  only:
    - sources                           # for Github Project Pages replace with "master"

install:  
  - rm -rf public || exit 0             # cleanup previous run
script:  
  - binaries/hugo                       # Generate
after_success:  
  - cp .travis.yml public               # all branches need this file
  - cp README.md public                 # copy readme
  - bash deploy.sh                      # run the deploy
```

# Configuring Travis-CI

Now we have to tell Travis which repository should be build. Login to Travis-CI and
change our repo general settings tab as Build only if .travis.yml present, build
pushes and Build pull requests as <b>ON</b>. Put Limit concurrent jobs as <b>OFF</b>.

# Hugo binary

One could simply advise Travis CI to install Hugo during the website build. But
a much more elegant way is to cross compile the Hugo binary ourselves and then
deploy it in the `sources` branch. This will also speedup the build.

Run the following command on project root directory. I am using go version
`go1.9.2 windows/amd64`.

```shell
$ mkdir binaries && cd binaries
$ env GOPATH="`pwd`" go get -v github.com/spf13/hugo
$ env GOPATH="`pwd`" GOOS=linux GOARCH=amd64 go build -v github.com/spf13/hugo
```

Now we can delete 💀 other source packages from `binaries` folder and we may
need to change our permission for the hugo binary as executable with

```shell
$ git update-index --add --chmod=+x binaries/hugo
```

# Wrapping up

Final folder structure should be as follows

```sh
├── [archetypes]
├── [binaries]
    └── hugo
├── [content]
├── [data]
├── [layouts]
├── [static]
├── [themes]
├── config.toml
├── deploy.sh
├── .travis.yml
└── README.md
```
Now with my every push to `sources` Travis-CI should run and generate static
HTML and push it back to our `master` branch. The results should be visible
on our GitHub Page.

So that's how I build the pipeline for SadatRocks. ✌️
