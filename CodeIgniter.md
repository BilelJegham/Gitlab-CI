

# Gitlab-CI pour CodeIgniter
 
> CodeIgniter, Heroku, PHP

Déploiement du projet sur Heroku. 2 espaces ont été créer sur heroku un pour la branche ``master`` un autre pour ``dev``. Note la procédure est assez longue d'une part il faut gérer les clés API de heroku, pour gérer la BD heroku. 

2 liens utiles : 
-  https://docs.gitlab.com/ee/ci/examples/test-and-deploy-ruby-application-to-heroku.html
- https://www.youtube.com/watch?v=mBCH9OTVaGw

3 phases (stages) sont lancés par la CI :
  1. deploy

## Les fichiers

<!-- TOC depthFrom:3 orderedList:true -->

1. [.gitlab-ci.yml](#gitlab-ciyml)
2. [mysql](#mysql)

<!-- /TOC -->

### .gitlab-ci.yml
```yml
stages:
  - deploy

before_script:
  - cp ./GitlabCI/config.php ./application/config/config.php
  - cp ./GitlabCI/config.php ./application/config/database.php


deploy-dev:
  stage: deploy
  script:
  - gem install dpl
  - dpl --provider=heroku --app=name --api-key=$HEROKU_STAGING_API_KEY
  only:
  - dev

deloy-master:
  stage: deploy
  script:
  - gem install dpl
  - dpl --provider=heroku --app=name --api-key=$HEROKU_STAGING_API_KEY
  only:
  - master

```

### mysql

```yml

stages:
  - build
  - backup

variables:
  MYSQL_DATABASE: <heroku>
  MYSQL_HOST: <clear-db.net>
  MYSQL_PORT: <3306>
  MYSQL_USERNAME: <username>
  MYSQL_PASSWORD: <password>


SQL:
  stage : build
  before_script:
    - apt-get -qq update && apt-get install -qq -y mysql-client #image mysql cmd mysqldump ne marche pas
  script:
   - mkdir sql/tmp
   - mysqldump --host=$MYSQL_HOST --port=$MYSQL_PORT --user=$MYSQL_USERNAME --password="$MYSQL_PASSWORD" $MYSQL_DATABASE > sql/tmp/dumpBefore.sql
   - mysqldump --no-create-info --complete-insert --host=$MYSQL_HOST --port=$MYSQL_PORT --user=$MYSQL_USERNAME --password="$MYSQL_PASSWORD" $MYSQL_DATABASE > sql/tmp/dataOnly.sql   
   - cat sql/*.sql > sql/tmp/newSchema.sql
   - mysql --host=$MYSQL_HOST --port=$MYSQL_PORT --user=$MYSQL_USERNAME --password="$MYSQL_PASSWORD" $MYSQL_DATABASE < sql/tmp/newSchema.sql
   - mysql --host=$MYSQL_HOST --port=$MYSQL_PORT --user=$MYSQL_USERNAME --password="$MYSQL_PASSWORD" $MYSQL_DATABASE < sql/tmp/dataOnly.sql
   - mysqldump --host=$MYSQL_HOST --port=$MYSQL_PORT --user=$MYSQL_USERNAME --password="$MYSQL_PASSWORD" $MYSQL_DATABASE > sql/tmp/dumpAfter.sql
  artifacts:
    paths: 
     - sql/tmp/
  cache:
    key: sql
    paths:
      - sql/tmp/
    policy: push
  only:
    - dev

backup-sql:
  stage: backup
  before_script:
    - apt-get -qq update && apt-get install -qq -y mysql-client
  script:
  - mysql --host=$MYSQL_HOST --port=$MYSQL_PORT --user=$MYSQL_USERNAME --password="$MYSQL_PASSWORD" $MYSQL_DATABASE < sql/tmp/dumpBefore.sql
  cache:
    key: sql
    paths:
      - sql/tmp/
    policy: pull
  only:
    - dev
  when: on_failure
```
