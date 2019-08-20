---
title: 1. Physical Hardware and Subscriptions
author: Richard White
date: '2019-08-20'
slug: this-is-a-test
categories:
tags: 
- test

---


## Physical Hardware and Subscriptions

- One Github organization (http://github.com/folkehelseinstituttet/)
- One Github team (https://github.com/orgs/folkehelseinstituttet/teams/dashboards)
- One [drat](https://github.com/eddelbuettel/drat) repository (https://folkehelseinstituttet.github.io/drat/)
- One travis-ci.org account (http://travis-ci.org/folkehelseinstituttet)
- One travis-ci.com account (http://travis-ci.com/folkehelseinstituttet)
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