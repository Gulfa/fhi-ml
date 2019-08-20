---
title: 2b. Infrastructure (Scripts)
author: Richard White
date: '2019-08-20'
slug: this-is-a-test
categories:
tags: 
- test

---

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

