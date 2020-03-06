
Electron JS
===

> JS, Electron


Génération d'un exécutable electron et publication sur Gitlab Page :

<img src="img/pageElectron.png" alt="pageElectron" style="width:550px;text-align:center"/>  


```yml

image: node:latest

stages:
 - tests
 - build
 - deploy



lint:
  stage: tests
  before_script:
    - npm install
  script:
    - npm run lint

# test:
#   tags: 
#     - ubuntu
#   stage: tests
#   script:
#     - npm run test
#   artifacts:
#     paths:
#       - test/unit/coverage/lcov-report/


build:
  image: electronuserland/electron-builder:wine
  stage: build
  before_script:
    - npm install
    - git clone https://gitlab-ci-token:$CI_BUILD_TOKEN@gitlab.com/akeed/cucumber_selenium.git
    - git clone https://gitlab-ci-token:$CI_BUILD_TOKEN@gitlab.com/akeed/cumcumber-utplsql.git
  script:
    - npm run build:win32
    - npm run build:linux
    - cp -r cucumber_selenium/ build/akeed-testing-electron-linux-x64/
    - cp -r cumcumber-utplsql/ build/akeed-testing-electron-linux-x64/
    - cp -r cucumber_selenium/ build/akeed-testing-electron-win32-x64/
    - cp -r cumcumber-utplsql/ build/akeed-testing-electron-win32-x64/
    - cp run_test.bat build/akeed-testing-electron-win32-x64/run_test.bat
    - cp run_test.sh build/akeed-testing-electron-linux-x64/run_test.sh
    - cp -r utPLSQL-cli/ build/akeed-testing-electron-win32-x64/
    - cp -r utPLSQL-cli/ build/akeed-testing-electron-linux-x64/
  artifacts:
    paths:
      - build/
  only:
    - tags
    - master
    
pages:
  stage: deploy
  script:
    - apt-get update && apt-get -qq install -y zip unzip
    - mkdir public
    - zip -r public/windows.zip build/akeed-testing-electron-win32-x64
    - zip -r public/linux.zip build/akeed-testing-electron-linux-x64
    - cp -r .gitlab/site/* public/
    - sed -i "s/VERSION/${CI_COMMIT_TAG}/g" public/index.html
  artifacts:
    paths:
      - public
  only:
    - tags


```