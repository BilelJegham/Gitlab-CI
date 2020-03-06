
# Gitlab-CI pour C 
> MakeFile, gcc

Utilisation de Makefile pour compiler et lancer un projet en C.           
3 phases(stages) sont lancés par la CI :
  1. build
  2. test
  3. run


## Les fichiers



### Makefile
```makefile
run: compile
	./run
compile: 
	gcc **.c -ldl -o run
test:
	cppcheck .
install:
	apt-get -qq update && apt-get -qq install -y cppcheck

```
### .gitlab-ci.yml

```yml
image: gcc

stages:
  - build
  - test
  - run

compile:
  stage: build
  script:
    - make compile
    
run:
  stage: run
  script:
   - make run

test: 
  stage: test
  before_script:
    - make install
  script:
    - make test

```