

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

<!-- /TOC -->

### .gitlab-ci.yml
```yml

before_script:
  - cp ./GitlabCI/config.php ./application/config/config.php
  - cp ./GitlabCI/config.php ./application/config/database.php


deploy-dev:
  script:
  - gem install dpl
  - dpl --provider=heroku --app=idesys-egc-dev --api-key=$HEROKU_STAGING_API_KEY
  only:
  - dev

deloy-master:
  script:
  - gem install dpl
  - dpl --provider=heroku --app=idesys-egc --api-key=$HEROKU_STAGING_API_KEY
  only:
  - master

```