PL/SQL Selenium
====

> Sql, utplsql, Selenium

https://gitlab.com/akeed/adf-structure/-/tree/master

```yml
stages:
  - tests-unit
  - tests-fonc
  - deploy

variables:
  COMPOSE_FILE: "docker-compose.yml:docker-compose.gitlab-ci.yml"  

utplsql:
  tags:
    - ubuntu
  stage : tests-unit
  before_script:    
    - docker login -u gitlab-ci-token -p $CI_BUILD_TOKEN registry.gitlab.com
    - docker-compose pull oracle || true
    - docker-compose up -d --force-recreate oracle  
    - sleep 1m
    - docker-compose exec -T oracle sh akeed/.gitlab/database/scriptinstall.sh
    - cd database && sh run.sh && cd ..
    - git clone https://gitlab-ci-token:$CI_BUILD_TOKEN@gitlab.com/akeed/cumcumber-utplsql.git
    - cp database/test/* cumcumber-utplsql/src/test/resources/utPLSQLCucumber/
    - cd cumcumber-utplsql && mvn test 
    - cd ..
    - docker-compose exec -T oracle sh akeed/.gitlab/database/install_test.sh
  script:
    - sh  .gitlab/java/scriptjava.sh
    - cat coverage.html | grep "lines covered at"
  after_script:
    - docker-compose down 
  artifacts:
    paths :
      - coverage.html
      - coverage.html_assets/
    expire_in: 1 week
    
selenium:
  tags:
    - ubuntu
  stage : tests-fonc
  before_script:    
    - git clone https://gitlab-ci-token:$CI_BUILD_TOKEN@gitlab.com/akeed/cucumber_selenium.git
    - docker run -d -p 4444:4444 -v /dev/shm:/dev/shm --name ${CI_PROJECT_NAME}_selenium-standalone --rm selenium/standalone-chrome:3.141.59-titanium 
    - cp application/test/* cucumber_selenium/src/test/resources/cucumber_selenium/
  script:
    - cd cucumber_selenium && mvn test 
  after_script:
    - docker stop ${CI_PROJECT_NAME}_selenium-standalone
  artifacts:
    paths:
      - cucumber_selenium/target/cucumber-html-report/
      - cucumber_selenium/screenshots/


pages:
  stage: deploy
  script:
    - mkdir public
    - mkdir public/coverage
    - cp coverage.html public/coverage/index.html
    - mkdir public/coverage/coverage.html_assets
    - cp -R coverage.html_assets/* public/coverage/coverage.html_assets/
    - mkdir public/selenium
    - mkdir public/selenium/screenshots
    - cp -R cucumber_selenium/screenshots/* public/selenium/screenshots/
    - mkdir public/selenium/cucumber-html-report
    - cp -R cucumber_selenium/target/cucumber-html-report/* public/selenium/cucumber-html-report/
  artifacts:
    paths:
      - public
  only:
    - master




```