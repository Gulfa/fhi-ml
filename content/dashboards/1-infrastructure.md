---
title: General Infrastructure
author: Richard White
date: '2019-01-01'
slug: infrastructure 
toc: true
categories:
tags: 
- test
---

# Executive summary

The dashboards project is a project at FHI concerned with running automated analyses on data.

In principle, the dashboards project is split up into two parts:

1. The umbrella infrastructure (i.e. Docker containers, continuous integration, chron jobs, etc.)
2. The R package for each automated analysis

## What is an automated analysis?

An automated analysis is any analysis that:

1. Will be repeated multiple times in the future
2. Always has an input dataset with consistent file structure
3. Always has the same expected output (e.g. tables, graphs, reports)

## Why not have one project for each automated analysis?

Automated analyses have a lot of code and infrastructure in common.

Automated analyses:

1. Need their code to be tested via unit testing to ensure the results are correct
2. Need their code to be tested via integration testing to ensure everything runs
3. Need to be run at certain times
4. Need to be able to send emails notifying people that the analyses have finished running
5. Need to make their results accessible to the relevant people

By combining them all in one umbrella project we can force everyone to use the same infrastructure and coding principles, so we:

1. Only need to solve a problem once
2. Only need to maintain one system
3. Can easily work on multiple projects, as we all speak the same language

## Important repositories 

### Infrastructure

https://github.com/raubreywhite/dashboards_control/ (private)

This contains the Dockerfiles, cronfiles, all bash scripts, etc.

http://github.com/raubreywhite/docker/

This contains the base analysis Dockerfile

https://rocker-project.org

https://folkehelseinstituttet.github.io/fhi/

This is an R package that contains helper functions.

### Automated analyses R packages

https://www.github.com/folkehelseinstituttet/dashboards_sykdomspuls/

https://www.github.com/folkehelseinstituttet/dashboards_normomo/

https://www.github.com/folkehelseinstituttet/dashboards_sykdomspulspdf/

https://www.github.com/folkehelseinstituttet/dashboards_noispiah/

https://www.github.com/folkehelseinstituttet/dashboards_sykdomspulslog/

# Physical Hardware and Subscriptions

- One Github organization (http://github.com/folkehelseinstituttet/)
- One Github team (https://github.com/orgs/folkehelseinstituttet/teams/dashboards)
- One [drat](https://github.com/eddelbuettel/drat) repository (https://folkehelseinstituttet.github.io/drat/)
- One travis-ci.com account (http://travis-ci.com/folkehelseinstituttet)
- One travis-ci.org account (http://travis-ci.org/folkehelseinstituttet)
- One Docker hub account (http://hub.docker.com/u/raw996/)
- At least three computers:
    1. Production linux computer `smhb`
    2. Integration testing linux computer `linux`
    3. Development linux computers (1 per person)

## Production Computer - smhb

### Explanation

`smhb` is designed to be extremely simple. It has two jobs:

1. Updating [raw996/dashboards_r:production](https://hub.docker.com/r/raw996/dashboards_r/), [raw996/dashboards_shiny:production](https://hub.docker.com/r/raw996/dashboards_shiny/), and [raw996/dashboards_nginx:production](https://hub.docker.com/r/raw996/dashboards_nginx/)
2. Making sure that [docker-compose-prod.yml](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/docker-compose-prod.yml) is running

We purposefully minimize all integration with the base machine, because this enables us to be deploy-environment independent and to move our deployment anywhere with minimal hassle.

### Requirements

- Git
- Docker Engine - Community (https://www.docker.com/products/docker-engine)

### File Structure

```
/home/raw996/git/dashboards_control/
```

### crontab

```
# m h  dom mon dow   command
5 1 * * * /home/raw996/git/dashboards_control/bin/prod_update.sh
@reboot /home/raw996/git/dashboards_control/bin/prod_update.sh
```

## Integration Testing - linux

### Explanation

`linux` has three jobs:

1. Building [raw996/dhadley](https://github.com/raubreywhite/docker/blob/master/dhadley/Dockerfile
2. Integration testing for the automated analyses
3. Pushing [raw996/dashboards_r:production](https://hub.docker.com/r/raw996/dashboards_r/), [raw996/dashboards_shiny:production](https://hub.docker.com/r/raw996/dashboards_shiny/), and [raw996/dashboards_nginx:production](https://hub.docker.com/r/raw996/dashboards_nginx/) to Docker hub after successful integration testing

Integration testing happens by Jenkins running:

- [test_noispiah.sh](https://github.com/raubreywhite/dashboards_control/blob/master/bin/test_noispiah.sh)
- [test_normomo.sh](https://github.com/raubreywhite/dashboards_control/blob/master/bin/test_normomo.sh)
- [test_sykdomspuls.sh](https://github.com/raubreywhite/dashboards_control/blob/master/bin/test_sykdomspuls.sh)
- [test_sykdomspulslog.sh](https://github.com/raubreywhite/dashboards_control/blob/master/bin/test_sykdomspulslog.sh)
- [test_sykdomspulspdf.sh](https://github.com/raubreywhite/dashboards_control/blob/master/bin/test_sykdomspulspdf.sh)

### Requirements
- Git
- Docker Engine - Community (https://www.docker.com/products/docker-engine)
- Jenkins installed via a Docker container (http://jenkins.io)

### File Structure

```
/home/raw996/
  |-- docker-compose-jenkins/
    |-- docker-compose.yml
  |-- data/
    |-- data_app/
      |-- normomo/
      |-- sykdomspuls/
    |-- data_clean/
      |-- normomo/
      |-- sykdomspuls/
    |-- data_raw/
      |-- normomo/
        |-- FHIDOD2_20170425.txt
      |-- sykdomspuls/
        |-- partially_formatted_2017_08_01.txt
    |-- data_results/
      |-- normomo/
      |-- sykdomspuls/
  |-- jenkins/
```

## Development Computer - dev

### Requirements

- Git
- Docker Engine - Community (https://www.docker.com/products/docker-engine)

## Unit Testing - travis-ci {#travis}

[Travis-ci](http://travis-ci.org/folkehelseinstituttet) is used for  [unit testing of packages](http://r-pkgs.had.co.nz/check.html#travis). If the R package passes all tests, then we use [drat](https://github.com/eddelbuettel/drat) to deploy a built version of the package to Folkehelseinstituttet's R repository: https://folkehelseinstituttet.github.io/drat/.

# Analysis Docker Image

## Images

Our analysis Docker images are based off the [rocker](https://rocker-project.org) images. More specifically, the [rocker/verse:3.5.0](https://hub.docker.com/r/rocker/verse/) image.

This Docker image is then expanded upon by a separate Dockerfile [raw996/dhadley](https://github.com/raubreywhite/docker/blob/master/dhadley/Dockerfile). This Docker image is automatically rebuilt by `Jenkins` on `linux` whenever the repository is updated. The resultant Docker image is pushed to [raw996/dhadley:3.5.0](https://hub.docker.com/r/raw996/dhadley/). This image is a general-purpose analysis image, with no sensitive information in it.

This Docker image is then expanded upon by a separate Dockerfile [raw996/dashboards_r](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/Dockerfile). This Docker image is automatically rebuilt by `Jenkins` on `linux` whenever the repository is updated. The resultant Docker image is locally tagged as `raw996/dashboards_r:test` and then a number of integration tests are performed on it. If the integration tests are passed, then the Docker image is retagged and pushed to [raw996/dashboards_r:production](https://hub.docker.com/r/raw996/dashboards_r/). This image is private as it contains passwords and email addresses.

## File Structure

Inside `raw996/dashboards_r` we have the following file structure:

```
/data_raw/
  |-- normomo/
  |-- noispiah/
  |-- sykdomspuls/
  |-- sykdomspulspdf/
  |-- sykdomspulslog/
/data_clean/
  |-- normomo/
  |-- noispiah/
  |-- sykdomspuls/
  |-- sykdomspulspdf/
  |-- sykdomspulslog/
/data_app/
  |-- normomo/
  |-- noispiah/
  |-- sykdomspuls/
  |-- sykdomspulspdf/
  |-- sykdomspulslog/
/results/
  |-- normomo/
  |-- noispiah/
  |-- sykdomspuls/
  |-- sykdomspulspdf/
  |-- sykdomspulslog/
/usr/local/lib/R/site-library/  <soft linked to /r>
  |-- <OTHER R PACKAGES INSTALLED HERE>/
  |-- fhi/
  |-- normomo/
  |-- noispiah/
  |-- sykdomspuls/
  |-- sykdomspulspdf/
  |-- sykdomspulslog/
  |-- <OTHER R PACKAGES INSTALLED HERE>/

/tmp/computer <computer name>

/etc/secret.sh

/etc/gmailr/mail.json
/etc/gmailr/.httr-oauth
/etc/gmailr/emails.xlsx
/etc/gmailr/emails_test.xlsx
/etc/gmailr/emails_sykdomspuls_alert.xlsx
/etc/gmailr/emails_sykdomspuls_alert_test.xlsx

/etc/cron.d/simple-cron
/etc/auto.mounts

/etc/cont-init.d/add_cron
/etc/cont-init.d/add_autofs
```

Note that we have a soft link between `/r` and `/usr/local/lib/R/site-library/`.


## cron

We use [cron](https://en.wikipedia.org/wiki/Cron) to schedule the analyses. The schedule is specified in [crontab](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/crontab).

The cronjobs are only activated when the environmental variable `ADD=cron` is defined. Cronjobs are then activated through [add_cron.sh](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/add_cron.sh).

In principle, cronjobs should only be activated on `smhb`.

## autofs

We use [autofs](https://help.ubuntu.com/community/Autofs) to connect to the F network. The network locations, username, and password are specified in [auto.mounts](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/auto.mounts).

Autofs is only activated when the environmental variable `ADD_AUTOFS=yes` is defined. Autofs is then activated through [add_autofs.sh](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/add_autofs.sh).

In principle, autofs should only be activated on `smhb`.

## Emails

[emails.xlsx](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/emails.xlsx) and [emails_test.xlsx](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/emails_test.xlsx) contain email lists.

[emails_sykdomspuls_alert.xlsx](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/emails_sykdomspuls_alert.xlsx) and [emails_sykdomspuls_alert_test.xlsx](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/emails_sykdomspuls_alert_test.xlsx) contain email lists.

[httr-oauth_2017_09_17](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/httr-oauth_2017_09_17) contains the authentication credientials for the gmail account. 

## Passwords and Keys

[repo-key](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/repo-key) repo key that gives access to the private [raubreywhite/dashboards_control](https://github.com/raubreywhite/dashboards_control) repository.

[secret.sh](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/secret.sh) contains passwords.

# Reverse Proxy Docker Image

We use nginx as a reverse proxy to make rstudio server available to the developers.

The relevant Dockerfile is [here](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_nginx/Dockerfile) and is pushed to [raw996/dashboards_nginx:production](https://hub.docker.com/r/raw996/dashboards_nginx/) after integration testing is passed.

# Docker Compose {#dockercompose}

[Docker compose](https://docs.docker.com/compose/) is used to integrate these Docker images into the local filesystem. We have multiple docker-compose files for different reasons:

- For [production](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/docker-compose-prod.yml) on `smhb`
- For [testing](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/docker-compose-test.yml) on `linux`
- For [development](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/docker-compose-dev.yml) on a dev computer

# Scripts

# Overview

Most of the bash scripts, Docker files, passwords, etc. are all hosted at the private Github repository [raubreywhite/dashboards_control](https://github.com/raubreywhite/dashboards_control).

```
- $DASHBOARDS_FOLDER/dashboards_control/
  |-- bin/
  |  |-- dev_down.sh
  |  |-- dev_up.sh
  |  |-- docker_build.sh
  |  |-- docker_login.sh
  |  |-- docker_push_test_to_prod.sh
  |  |-- prod_down.sh
  |  |-- prod_up.sh
  |  |-- prod_update.sh
  |  |-- public.sh
  |  |-- test_noispiah.sh
  |  |-- test_normomo.sh
  |  |-- test_sykdomspuls_log.sh
  |  |-- test_sykdomspuls_pdf.sh
  |  |-- test_sykdomspuls.sh
  |-- docker-compose-api.yml
  |-- docker-compose-dev.yml
  |-- docker-compose-prod.yml
  |-- docker-compose-test.yml
```

# Bash scripts

## $DASHBOARDS_FOLDER/dashboards_control/bin/dev_down.sh

Dev script to shut down docker-compose

## $DASHBOARDS_FOLDER/dashboards_control/bin/dev_up.sh

Dev script to start docker-compose

## $DASHBOARDS_FOLDER/dashboards_control/bin/docker_build.sh

Builds all relevant Docker containers

## $DASHBOARDS_FOLDER/dashboards_control/bin/docker_login.sh

Logs in to docker-hub

## $DASHBOARDS_FOLDER/dashboards_control/bin/docker_push_test_to_prod.sh 

Retags 'test' Docker containers to 'production' and pushs them to docker-hub

## $DASHBOARDS_FOLDER/dashboards_control/bin/prod_down.sh

Only to be used on the production computer `smhb`. This bash script stops the docker-compose.

Note: This script is not used.

## $DASHBOARDS_FOLDER/dashboards_control/bin/prod_up.sh

Only to be used on the production computer `smhb`. This bash script starts up the docker-compose.

Note: This script is not used.
    
## $DASHBOARDS_FOLDER/dashboards_control/bin/prod_update.sh

Only to be used on the production computer `smhb`. This bash script:

1. Removes unused Docker container/images (important so we don't run out of space)
2. Runs some scripts required for network access
3. Pulls the latest version of [raubreywhite/dashboards_control](https://github.com/raubreywhite/dashboards_control)
4. Stops the [docker-compose-prod.yml](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/docker-compose-prod.yml)
5. Pulls the latest production images for:
    - [raw996/dashboards_r:production](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/Dockerfile)
    - [raw996/dashboards_nginx:production](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_nginx/Dockerfile)
    - [raw996/dashboards_shiny:production](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_shiny/Dockerfile)
6. Starts the [docker-compose-prod.yml](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/docker-compose-prod.yml)

## $DASHBOARDS_FOLDER/dashboards_control/bin/public.sh

Lists a bunch of environmental variables

## $DASHBOARDS_FOLDER/dashboards_control/bin/test_noispiah.sh

Runs `/r/noispiah/src/RunTest.R` inside [raw996/dashboards_r:test](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/Dockerfile)

Note: This script is generally only run by `Jenkins` on `linux` as the integration testing for [noispiah](https://folkehelseinstituttet.github.io/dashboards_noispiah/).

## $DASHBOARDS_FOLDER/dashboards_control/bin/test_normomo.sh

Runs `/r/normomo/src/RunTest.R` inside [raw996/dashboards_r:test](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/Dockerfile)

Note: This script is generally only run by `Jenkins` on `linux` as the integration testing for [normomo](https://folkehelseinstituttet.github.io/dashboards_normomo/).

### $DASHBOARDS_FOLDER/dashboards_control/bin/test_sykdomspuls_log.sh

Runs `/r/sykdomspulslog/src/RunTest.R` inside [raw996/dashboards_r:test](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/Dockerfile)

Note: This script is generally only run by `Jenkins` on `linux` as the integration testing for [sykdomspuls_log](https://folkehelseinstituttet.github.io/dashboards_sykdomspuls_log/).

## $DASHBOARDS_FOLDER/dashboards_control/bin/test_sykdomspuls_pdf.sh

Runs `/r/sykdomspulspdf/src/RunTest.R` inside [raw996/dashboards_r:test](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/Dockerfile)

Note: This script is generally only run by `Jenkins` on `linux` as the integration testing for [sykdomspuls_pdf](https://folkehelseinstituttet.github.io/dashboards_sykdomspuls_pdf/).

## $DASHBOARDS_FOLDER/dashboards_control/bin/test_sykdomspuls.sh

Runs `/r/sykdomspuls/src/RunTest.R` inside [raw996/dashboards_r:test](https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/Dockerfile)

Note: This script is generally only run by `Jenkins` on `linux` as the integration testing for [sykdomspuls](https://folkehelseinstituttet.github.io/dashboards_sykdomspuls/).


# R Packages

## Overview

Each automated analysis has its own R package:

- [sykdomspuls](https://folkehelseinstituttet.github.io/dashboards_sykdomspuls/)
- [normomo](https://folkehelseinstituttet.github.io/dashboards_normomo/)
- [noispiah](https://folkehelseinstituttet.github.io/dashboards_noispiah/)
- [sykdomspulspdf](https://folkehelseinstituttet.github.io/dashboards_sykdomspulspdf/)
- [sykdomspulslog](https://folkehelseinstituttet.github.io/dashboards_sykdomspulslog/)

Each R package contains all of the code necessary for that automated analysis. Typical examples are:

- Data cleaning
- Signal analysis
- Graph generation
- Report generation

## Requirements

The R packages should be developed using unit testing as implemented in the [testthat](http://r-pkgs.had.co.nz/tests.html) package.

Furthermore, the R package should operate (and be able to be tested) independently from the real datasets on the system. This is because the real datasets cannot be shared publically or uploaded to github. To circumvent this issue, each package will need to develop functions that can generate fake data. [GenFakeDataRaw](https://folkehelseinstituttet.github.io/dashboards_sykdomspuls/reference/GenFakeDataRaw.html) is one example from [sykdomspuls](https://folkehelseinstituttet.github.io/dashboards_sykdomspuls/).

We also require that unit tests are created to test the formatting/structure of results. [ValidateAnalysisResults](https://folkehelseinstituttet.github.io/dashboards_sykdomspuls/reference/ValidateAnalysisResults.html) is one example from [sykdomspuls](https://folkehelseinstituttet.github.io/dashboards_sykdomspuls/), where the names of the data.table are checked against reference values to ensure that the structure of the results are not accidentally changed.

## Deployment via travis-ci and drat

Unit testing is then automatically run using [travis-ci](http://r-pkgs.had.co.nz/check.html#travis). If the R package passes all tests, then we use [drat](https://github.com/eddelbuettel/drat) to deploy a built version of the package to Folkehelseinstituttet's R repository: https://folkehelseinstituttet.github.io/drat/.

## Integration with the local file system

We assume that the local file system follows [this file structure](#internalfilestructure), and this is provided via Docker-compose from the [umbrella infrastructure](#umbrella).

Referencing the `data_raw`, `data_clean`, `data_app`, and `results` folders are done through the the [fhi](https://folkehelseinstituttet.github.io/fhi/articles/dashboardbasics.html) package.

## inst/src/RunProcess.R {#RunProcess}

An automated analysis needs to:

1. Know the location of the data/results folders.
2. Check for new data in these folders. If no new data - then quit.
3. Load in the data.
4. Load in the analysis functions.
5. Run the analyses.
6. Save the results.

`RunProcess.R` is responsible for these tasks.

We can think of it as an extremely short and extremely high-level script that implements the analysis scripts.

Depending on the automated analysis `RunProcess.R` can be run every two minutes (constantly checking for new data), or once a week (when we know that data will only be available on a certain day/time).

## inst/src/RunTest.R

This file is the brains of the integrated testing. This file will be run by the [$DASHBOARDS_FOLDER/dashboards_control/bin/test_*.sh](#test_noispiah) files.

## inst/bin/0_run.sh

This file will be run by cron

This file is used to:

1. Do any pre-analysis steps (e.g. download new data from an SFTP server)
2. Run [inst/src/RunProcess.R](#RunProcess)

```
/usr/local/bin/Rscript /r/ANALYSIS/src/RunProcess.R
```

3. Do any post-analysis steps (e.g. upload results to an SFTP server)

# Contributing

## Quickstart

1. Create a project folder for your code (let us say `~/git/dashboards`). This folder will be accessible from inside your Docker container as `/dashboards/`
2. Create a project folder for your data (let us say `~/data/`).
3. Clone the following repos inside `~/git/dashboards`:
    - https://github.com/raubreywhite/dashboards_control/ (private)
4. Fork the following repos, and then clone the forks to your computer inside `~/git/dashboards` (depending on which project you are working on):
    - https://github.com/folkehelseinstituttet/dashboards_sykdomspuls/
    - https://github.com/folkehelseinstituttet/dashboards_normomo/
    - https://github.com/folkehelseinstituttet/dashboards_sykdomspulspdf/
    - https://github.com/folkehelseinstituttet/dashboards_noispiah/
    - https://github.com/folkehelseinstituttet/dashboards_sykdomspulslog/
5. For each repo, add the folkehelseinstitutet repository as your upstream:

```
git remote add upstream https://github.com/folkehelseinstituttet/ORIGINAL_REPOSITORY.git
```

6. In your `~/.profile` add the following three lines:

```
export DASHBOARDS_DATA=~/data/
export DASHBOARDS_FOLDER=~/git/dashboards/
export PATH=$PATH:$DASHBOARDS_FOLDER/dashboards_control/bin/
```

7. Build your Dockerfiles:

```
docker_build.sh
```

8. A number of folders will have been automatically created (see below). Please put in your development datafiles into the appropriate `data_raw/` folder:

```
- $DASHBOARDS_DATA/
  |-- data_app/
    |-- sykdomspuls/
    |-- normomo/
    |-- sykdomspulslog/
    |-- sykdomspulspdf/
  |-- data_clean/
    |-- sykdomspuls/
    |-- normomo/
    |-- sykdomspulslog/
    |-- sykdomspulspdf/
  |-- data_raw/
    |-- sykdomspuls/
    |-- normomo/
    |-- sykdomspulslog/
    |-- sykdomspulspdf/
  |-- results/
    |-- sykdomspuls/
    |-- normomo/
    |-- sykdomspulslog/
    |-- sykdomspulspdf/
```

9 Run your docker-compose:

```
dev_up.sh
```

10. Open a browser and go to http://localhost:8788/
11. Login using `username=rstudio` and `password=rstudio1`
12. Using the project menu, open the project corresponding to the automated analysis you are interested in. This should be located at `/dashboards/`.

## New Automated Analysis {#newautomatedanalysis}

This will walk you through the creation of a new automated analysis called "yyyy".

1. Create a new repository on [folkehelseinstituttet](https://github.com/folkehelseinstituttet) with the name `dashboards_yyyy`.
2. Fork this repository to your Github account
3. Clone the forked repository to `~/git/dashboards/dashboards_xxxx`
4. Download [dashboards_template](https://github.com/folkehelseinstituttet/dashboards_xxxx) and copy all of its contents into `~/git/dashboards/dashboards_yyyy`
5. Edit the following files:
    - `~/git/dashboards/dashboards_yyyy/DESCRIPTION` (`Package: xxxx` -> `Package: yyyy`)
    - `~/git/dashboards/dashboards_yyyy/tests/testthat.R` (`library(xxxx)`->`library(yyyy)`)
    - `~/git/dashboards/dashboards_yyyy/.travis.yml` (`xxxx`->`yyyy`)
    - `~/git/dashboards/dashboards_yyyy/inst/bin/0_run.sh` (`xxxx`->`yyyy`)
    - `~/git/dashboards/dashboards_yyyy/inst/src/RunProcess.sh` (`xxxx`->`yyyy`)
    - `~/git/dashboards/dashboards_yyyy/inst/src/RunTest.sh` (`xxxx`->`yyyy`)
    
6. Ask Richard to log into travis-ci.com and add the `GITHUB_PAT` environmental variable to activate continuous integration for `yyyy`
7. Submit pull requests and ensure that:
    - Everything works
    - The package is successfully build and included in https://github.com/folkehelseinstituttet/drat/tree/gh-pages/src/contrib

**You cannot proceed any further until Step 7 works properly (i.e. the package is included in the drat repository).**

8. Add your package under the secton "## DRAT PACKAGES FROM FHI" at https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/Dockerfile
9. Create the 4 folders `data_raw`, `data_clean`, `results`, `data_app` at https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/Dockerfile
10. Add your package's [0_run.sh](#Runsh) to https://github.com/raubreywhite/dashboards_control/blob/master/infrastructure/dashboards_r/crontab
11. Add your package's [test_yyyy.sh](#integrationtesting) to `Jenkins` on `Linux`
12. Hope it works!

## Development guidelines

We try to follow the [GitHub flow](https://guides.github.com/introduction/flow/) for development.

1. If you have forked and cloned the project before and it has been a while since you worked on it, merge changes from the original repo to your clone by using:
```
git fetch upstream
git merge upstream/master
```
2. Open the RStudio project file (`.Rproj`).
3. Make your changes:
    * Write your code.
    * Test your code (bonus points for adding unit tests).
    * Document your code (see function documentation above).
    * Do an `R CMD check` using `devtools::check()` and aim for 0 errors and warnings.
    * Commit your changes locally
    * Merge changes from the original repo (again)
    * Do an `R CMD check` using `devtools::check()` and aim for 0 errors and warnings.
4. Commit and push your changes.
5. Submit a [pull request](https://guides.github.com/activities/forking/#making-a-pull-request).
6. If you are reviewing the pull request, wait until the [travis-ci](www.travis-ci.org) unit tests have finished

```{r echo=FALSE, message=FALSE, warning=FALSE}
knitr::include_graphics("images/pull_request_before_checks.png")
```

7. Please make sure that the unit tests `PASS` before merging in!!

```{r echo=FALSE, message=FALSE, warning=FALSE}
knitr::include_graphics("images/pull_request_checks_failed.png")
```

## Code style

- Reference [Hadley's style code](http://adv-r.had.co.nz/Style.html)
- <- is preferred over = for assignment
- Indentation is with two spaces, not two or a tab. There should be no tabs in code files.
- if () {} else {} constructions should always use full curly braces even when usage seems unnecessary from a clarity perspective.
- TODO statements should be opened as GitHub issues with links to specific code files and code lines, rather than written inline.
- Follow Hadley’s suggestion for aligning long functions with many arguments:
```
 long_function_name <- function(a = "a long argument", 
                                b = "another argument",
                                c = "another long argument") {
   # As usual code is indented by two spaces.
 }
```
- Never use print() to send text to the console. Instead use fhi::DashboardMsg() as appropriate.
- Use environment variables, not options(), to store global arguments that are used by many or all functions.

# Manual actions

## Sykdomspulsen

### Sykdomspulsen Weekly update

The weekly update needs to be done every Tuesday morning. 
New data has arrived from Helsedirektoratet to FHI during Monday evening and night, and the further statistical analysis and updating of the interactive webpage for Sykdomspulsen is done on Tuesday. 
For more information about Sykdomspulsen setup:

https://folkehelseinstituttet.github.io/dashboards/

https://folkehelseinstituttet.github.io/dashboards_sykdomspuls/

1. Log in to «Sikker sone»
2. Click on «Sikkersone statistikk»
3. Open R or R studio by clicking on the four squares in the bottom left corner, then on the arrow pointing downwards approximately in the same area and search for R in the upper right corner
4. When R is open, click on “file” in the upper left corner, and then “open script”
5. Open G:/Helseregistre/MSIS/Sykdomspulsen/Gry/FormattingWithinSikkersone/  
SykdomspulsenWeeklyReport
6. Slide the R-code so that it is beside the R console (so that you can see the whole R console)
7. Put the marker within the R-code and click Ctrl+A, then click Ctrl+R
8. After a few minutes, you will see a progress bar on the R-console, which shows how much of the code is finished. When the bar shows 100%, the whole code has run, and the results are finished (this takes about 1 hour, you can do other things while waiting for this)
9. When the progress bar shows 100% you should look at the data that is shown in the console and see if the dates are from this week. If not you should contact the IT department by Cathrine Slorbak (Cathrine.Slorbak@fhi.no) and Gry Grøneng (GryMarysol.Groneng@fhi.no). There is probably something wrong with receiving the data from the Health directorate, hence the IT department needs to solve it. Do not continue until the problem is solved.
10. If the dates look ok, you can close R (by clicking on x in the upper right corner). 
11. Go to “Windows explorer” by clicking on the four squares in the lower left corner (still in “sikker sone”), and go to: G:/Helseregistre/MSIS/Sykdomspulsen/Gry/  
FormattingWithinSikkersone, and find a text file which is called “partially_formatted_  
todays date.txt” (todays date is displayed 2017_06_13). You should look if the file is larger than last weeks file, it should be larger since we take out all the data every week, and more days have been added since last week. If the file is smaller than last week, contact the IT department by Cathrine Slorbak (Cathrine.Slorbak@fhi.no) and Gry Grøneng (GryMarysol.Groneng@fhi.no). Do not continue until the problem is solved.
12. Copy and paste the file into the folder “Filer til ordinær sone (J:)”
13. Log out of “sikker sone” and open the windows explorer in the ordinary zone and go to “Filer overført fra sikker sone (K:)” and find the file.
14. Copy and paste the file into:
  + F:/Prosjekter/sMSIS/WeeklyUpdates_Sykdomspulsen AND
  + F:/Prosjekter/Dashboards/data_raw/sykdomspuls
15. You have now done the weekly update of Sykdomspulsen!
