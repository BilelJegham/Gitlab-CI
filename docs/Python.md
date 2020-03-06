# Gitlab-CI pour Python
> python, pep8, enregistrements d'artifacts

Cette CI a été utilisé dans le cadre du GoogleHashCode 2017.    
Elle permettait de tester la syntaxe (Pep8) dans un premier temps.    
De générer les fichiers de sortie issue de nos algorithmes dans un second temps.   
 Ces fichiers peuvent ensuite être téléchargés depuis l'onglet CI de Gitlab.

3 phases(stages) sont lancées par le CI ;
  1. test
  1. build


## Les fichiers

<!-- TOC depthFrom:3 orderedList:true -->autoauto1. [.gitlab-ci.yml](#gitlab-ciyml)autoauto<!-- /TOC -->


### .gitlab-ci.yml

```yml
before_script:
  - apt-get -qq update && apt-get -qq install -y python3
  - apt-get -qq install -y python-pip
  - pip install flake8 pep8-naming
  - apt-get -qq update


stages:
  - test
  - build

pep-8:
  stage: test
  script:
  - flake8 src/python/**.py

charleston_road:
  stage: build
  script:
    - python3 src/python/main.py  src/res/in/charleston_road
  artifacts:
    paths:
    - src/res/out/charleston_road.out

rue_de_londres:
  stage: build
  script:
    - python3 src/python/main.py  src/res/in/rue_de_londres
  artifacts:
    paths:
    - src/res/out/rue_de_londres.out

opera:
  stage: build
  script:
    - python3 src/python/main.py  src/res/in/opera
  artifacts:
    paths:
    - src/res/out/opera.out

lets_go_higher:
  stage: build
  script:
    - python3 src/python/main.py  src/res/in/lets_go_higher
  artifacts:
    paths:
    - src/res/out/lets_go_higher.out

```