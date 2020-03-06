

# Gitlab-CI pour JavaScript
 
> WebPack, VueJS



<!-- TOC depthFrom:2 orderedList:true -->

1. [WebPack](#webpack)
    1. [gitlab-ci.yml](#gitlab-ciyml)
2. [VueJS](#vuejs)
    1. [gitlab-ci.yml](#gitlab-ciyml-1)
    2. [Github Action](#github-action)

<!-- /TOC -->

## WebPack
### gitlab-ci.yml
```yml
image: node:latest

pages:
  stage: deploy
  before_script:
    - npm install
  script:
    - npm run build
    - mkdir public
    - cp -r dist/ public/
  artifacts:
    paths:
      - public

```

## VueJS
### gitlab-ci.yml
```yml
image: node:latest

pages:
  stage: deploy
  before_script:
    - npm install
  script:
    - npm run build
    - mkdir public
    - cp -r dist/ public/
  artifacts:
    paths:
      - public

```

### Github Action
> https://github.com/idesys-dev/jeh-maker


github/workflows/main.yml
```yml
name: CI

on: [push]


jobs:
  linter:
    name: Linter
    runs-on: 'ubuntu-latest'
    steps:
    - uses: actions/checkout@v1
    - uses: actions/cache@v1
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    - run: npm ci
    - name: Linter
      run: npm run lint

  test-unit:
    name: Test Unit
    runs-on: 'ubuntu-latest'
    steps:
    - uses: actions/checkout@v1
    - uses: actions/cache@v1
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    - run: npm ci
    - name: Unit Test
      run: npm run test:unit
    - uses: codecov/codecov-action@v1
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
    - name: Upload artifact
      uses: actions/upload-artifact@v1.0.0
      with:
        # Artifact name
        name: coverage
        # Directory containing files to upload
        path: coverage

  test-e2e:
    name: Test e2e
    runs-on: 'ubuntu-latest'
    steps:
    - uses: actions/checkout@v1
    - uses: actions/cache@v1
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    - run: npm ci
    - name: Test e2e
      run: npm run test:e2e:firefox

  build:
    name: Build
    runs-on: 'ubuntu-latest'
    steps:
    - uses: actions/checkout@v1
    - uses: actions/cache@v1
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    - run: npm ci
    - name: Build
      run: npm run build
    - name: Upload artifact
      uses: actions/upload-artifact@v1.0.0
      with:
        # Artifact name
        name: build
        # Directory containing files to upload
        path: dist

  storybook:
    name: Storybook
    runs-on: 'ubuntu-latest'
    steps:
    - uses: actions/checkout@v1
    - uses: actions/cache@v1
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    - run: npm ci
    - name: Build
      run: npm run build-storybook
    - name: Upload artifact
      uses: actions/upload-artifact@v1.0.0
      with:
        # Artifact name
        name: storybook
        # Directory containing files to upload
        path: storybook-static

  chromatic:
    name: Chromatic
    runs-on: 'ubuntu-latest'
    steps:
    - uses: actions/checkout@v1
    - uses: actions/cache@v1
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    - run: npm ci
    - uses: chromaui/action@v1
      with:
        appCode: ${{ secrets.CHROMATIC_APP_CODE }}
        token: ${{ secrets.GITHUB_TOKEN }}



```
github/workflows/nightwatch.conf.js
```js

module.exports = {
    src_folders: ['tests/e2e/specs'],
  output_folder: 'tests/e2e/reports',
  page_objects_path: 'tests/e2e/page-objects',
  custom_assertions_path: 'tests/e2e/custom-assertions',
  custom_commands_path: 'tests/e2e/custom-commands',
    test_settings: {
        default: {
            launch_url: '${VUE_DEV_SERVER_URL}',
            
        },
        selenium: {
          selenium: {
            start_process: true,
            port: 4444,
            server_path: require('selenium-server').path,
            cli_args: {
              'webdriver.gecko.driver': require('geckodriver').path,
              'webdriver.chrome.driver': require('chromedriver').path
            }
          },
          webdriver: {
            start_process: false
          }
          },
          'selenium.chrome': {
            extends: 'selenium',
            desiredCapabilities: {
              browserName: 'chrome',
              chromeOptions : {
                w3c: false
              }
            }
          },
      
          'selenium.firefox': {
            extends: 'selenium',
            desiredCapabilities: {
              browserName: 'firefox'
            }
          }
    }
}
```
