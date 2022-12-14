# GitHub Actions

> Cours Repository: <https://github.com/academind/github-actions-course-resources>

> Plugin GitHub Actions for VSCode: <https://marketplace.visualstudio.com/items?itemName=cschleiden.vscode-github-actions>

- `GitHub Actions` is a workflow automation service: automate all kinds of repository-related processes & actions
- `Code Deployment`: automate code testing, building & deployment
  - `Continuous Integration` (CI): code changes are automatically built, tested & merged with existing code
  - `Continuous Delivery` (CD): after integration, new app or package versions are published automatically
- `Code & Repository Management`: automate code reviews, issue management etc.

## GitHub

- a company: a cloud `Git` repository & services provider to store and manage `Git repositories`
- Cloud Git repository storage (`push`, `pull`): public, private, team management etc.
- Code management & collaborative development: via `issues`, `projects`, `pull requests` etc.
- Automation & CI/CD: via `GitHub Actions`, GitHub Apps etc.

## Key Elements

- `Workflows`
  - are attached to GitHub repositories
  - contain one or more `jobs`
  - are triggered when certain `events` are fired
  - by default, `workflows` get cancelled if `jobs` fail -> but you can cancel also manually directly in GitHub
  - by default, all matching `events` start a `workflow` -> exceptions for `push` and `pull_request`
    - there you can skip with proper commit message: `git commit -m "message [skip ci]"`
    - all skip keywords: <https://docs.github.com/en/actions/managing-workflow-runs/skipping-workflow-runs>
- `Jobs`
  - define a `runner` (i.e. execution environment)
  - contain one or more `steps`
  - if you have multiple `jobs`, they can run in parallel (default) or sequential
  - can run for certain conditions
  - by default, `jobs` fail if at least one `step` fails
- `Steps`
  - execute a `shell script` or an `action`
  - can use custom or third-party actions
  - steps are executed in order
  - can run for certain conditions

![](/01_slides/01_workflows-jobs-steps.png)

## Events (Workflow Triggers)

> List of Events: <https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows>

- `repository-related`

  - `push`: pushing a commit
  - `pull_request`: pull request action (opened, closed, ...)
    - by default, `pull requests` based on forks do NOT trigger a workflow -> to avoid malicious workflow runs & high costs for all these runs
  - `create`: a branch or tag was created
  - `fork`: repository was forked
  - `issues`: an issue was opened, deleted, ...
  - `issue_comment`: issue or pull request comment action
  - `watch`: repository was starred
  - `discussion`: discussion action (created, deleted, ...)
  - ... and more

- `other events`
  - `workflow_dispatch`: manually trigger workflow
  - `respository_dispatch`: REST API request triggers workflow
  - `schedule`: workflow is scheduled
  - `workflow_call`: can be called by other workflows

### Event Activity Types & Filters

> List of Events with their available `activity types` and `filters`: <https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows>

> Filter Patterns Cheat Sheet: <https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#patterns-to-match-branches-and-tags>

- some events habe `activity types`: allows more control over when a workflow will be triggered
  - e.g. `pull_request` event: `opened`, `closed`, `edited` ...
- some events have `filters`: allows more control over when a workflow will be triggered
  - e.g. `push` event: filter based on target branch

```yml
name: Events Demo 1
on:
  # Events with more control about when they are triggered
  pull_request:
    # activity types
    # - opened -> this notation also works
    types: [opened, synchronize, edited]
    # target branches of a pull request
    branches:
      - main
      - 'dev-*' # all branches starting with 'dev-' and characters behind (except slashes)
      - 'feature/**' # all branches starting with 'feature/' and any combination of characters and slashes after
  # notice: colon is mandatory, even if no more details defined below
  workflow_dispatch:
  push:
    # filters
    branches:
      - main
      - 'dev-*'
      - 'feature/**'
    branches-ignore:
      - 'foo/**'
    # filters based on file paths: runs workflow if commit changes file(s) in specific path(s) or except specific path(s)
    paths-ignore:
      - '.github/workflows/*' # triggers only if NO file changed in this path
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Output event data
        run: echo "${{ toJSON(github.event) }}"
      - name: Get code
        uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Test code
        run: npm run test
      - name: Build code
        run: npm run build
      - name: Deploy project
        run: echo "Deploying ..."
```

## Runners

> Docu runner environments: <https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources>

> List of preinstalled software of specific runner: <https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#preinstalled-software>

- servers (machines) that execute the jobs
- pre-defined `runners` (with different operating systems) exist
- you can also create custom `runners`

## Actions

- `command` (-> `run` key) is a (typically simple) shell command that's defined by you

- `action` (-> `uses` key)
  - is a (custom) application that performs a (typically complex) frequently repeated task
  - you can build you own `Actions` but you can also use official or community `Actions`
  - `GitHub marketplace` to find `Actions`: <https://github.com/marketplace>

### Custom Actions

#### Why Custom Actions?

- simplify workflow `steps`
  - instead of writing multiple (possibly very complex) `Step` definitions, build and use a single `custom Action`
  - multiple `Steps` can be grouped into a single `custom Action` and re-used across multiple jobs
- no existing (community) action
  - existing, public `Actions` might not solve the specific problem you have in your `workflow`
  - `custom Actions` can contain any logic you need to solve your specific `workflow` problems

#### Types of Custom Actions

- `JavaScript Actions`
  - execute a JS file
  - use JS (NodeJS) and any packages of your choice
  - straightforward (if you know JS)
- `Docker Actions`
  - create a Dockerfile with your required configuration
  - perform any task(s) of your choice with any language
  - lots of flexibility but requires Docker knowledge
- `Composite Actions`
  - combine multiple workflow steps in one single Action
  - combine `run` (commands) and `uses` (Actions)
  - allow for reusing shared steps (without extra skills)

#### Create Custom Actions

> Documentation YAML syntax for custom actions: <https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions>

1. create an `actions` folder (name is up to you) in `/.github` folder to create a `custom Action` that is available in the repo

   - create a subfolder for each action with an `action.yml`

1. create a standalone repo with your `custom Action` to make it available everywhere

   - Create a new, local Git repo project folder which contains your `action.yml` file (in root level) + all the code belonging to the action
   - Add a tag via `git tag -a -m "My action release" v1`
   - Push your local code to the remote GitHub repository (via `git push --follow tags`)
   - Use your custom Action in any other Workflow (in any other project and repository) by referencing the repository which contains your action (e.g., `my-account/my-action@v1`)
   - if your custom Action is stored in a public repository, it can also be published to the GitHub Actions Marketplace as described here: <https://docs.github.com/en/actions/creating-actions/publishing-actions-in-github-marketplace#publishing-an-action>

#### Example Composite Action

```yml
# ./.github/actions/cached-deps/action.yml
name: 'Get & Cache Dependencies'
description: 'Get the dependencies (via npm) and cache them'
# configure action inputs
inputs:
  # custom identifier
  caching:
    description: 'Whether to cache dependencies or not'
    required: false
    default: 'true'
  # ... add more inputs if needed
# configure action outputs
outputs:
  used-cache:
    description: 'Whether the cache was used'
    value: ${{ steps.install.outputs.cache }}

runs:
  # define that's a custom composite action
  using: 'composite'
  steps:
    - name: Cache dependencies
      # use inputs context object to insert configured values above
      if: inputs.caching == 'true'
      id: cache
      uses: actions/cache@v3
      with:
        path: node_modules
        key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
    - name: Install dependencies
      id: install
      # execute npm installation if cache is staled OR if NO caching was set
      if: steps.cache.outputs.cache-hit != 'true' || inputs.caching != 'true'
      # only to show how output works: output is set to input.caching value
      run: |
        npm ci
        echo "cache=${{ inputs.caching }}" >> $GITHUB_OUTPUT
      # if run key is used, then shell definition is needed in custom action
      shell: bash
```

```yml
# use custom action
name: Deployment
on:
  push:
    branches:
      - main
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Load & cache dependencies
        # a) if custom action stored in standalone repo
        # uses: MatchuPitchu/my-custom-action-repo-name
        # b) if custom action in same repo: relative path starts at root level of repo
        uses: ./.github/actions/cached-deps
        with:
          # set custom action input value
          caching: 'false'
      - name: Output custom action
        run: echo "Cache used? ${{ steps.cache-deps.outputs.used-cache }}"
      - name: Lint code
        run: npm run lint
    # ...
```

#### Example JavaScript Action

> Documentation GitHub Actions Toolkit for JavaScript that provides a set of npm packages: <https://github.com/actions/toolkit>

- notice: `node_modules` in `./.github/actions/YOUR_CUSTOM_JS_ACTION_FOLDER` should NOT be ignored
  - also ALL folders and files inside `node_modules` -> check in general `package.json` of whole repo, if e.g. `dist` folder is ignored

```yml
# ./.github/actions/deploy-s3-javascript/action.yml
name: 'Deploy to AWS S3'
description: 'Deploy a static website via AWS S3'
inputs:
  bucket:
    description: 'S3 bucket name'
    required: true
  bucket-region:
    description: 'Region of the S3 bucket'
    required: false
    default: 'us-east-1'
  dist-folder:
    description: 'Folder containing the deployable files'
    required: true
outputs:
  # register an output that is set from inside the js file
  website-url:
    description: 'URL of the website'
runs:
  # define the node environment for javascript code
  using: 'node16'
  # references to js files
  # pre: 'setup.js' # if setup needed
  main: 'main.js'
  # post: 'cleanup.js' # if cleanup needed
```

```javascript
// main.js
import * as core from '@actions/core';
import * as github from '@actions/github';
import * as exec from '@actions/exec';

const run = () => {
  // prints message to workflow log
  core.notice('Hello from my custom JS action');

  // 1) Get input values (defined in action.yml) to build a generic custom action
  const bucket = core.getInput('bucket', { required: true });
  const bucketRegion = core.getInput('bucket-region', { required: true }); // true since default will be provided in case
  const distFolder = core.getInput('dist-folder', { required: true });

  // 2) Upload files
  // exec can run shell command in js code
  // here use AWS CLI -> it's pre-installed on ubuntu-latest runner machine
  const s3Uri = `s3://${bucket}`;
  exec.exec(`aws s3 sync ${distFolder} ${s3Uri} --region ${bucketRegion}`);

  // 3) Generate output value
  const websiteUrl = `http://${bucket}.s3-website-${bucketRegion}.amazonaws.com`;
  core.setOutput('website-url', websiteUrl); // key is defined in action.yml
};

run();
```

```yml
# use custom action
name: Deployment
on:
  push:
    branches:
      - main
jobs:
  # ... other steps (lint, test, build)
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Get build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist-files
          path: ./dist
      - name: Output contents
        run: ls
      - name: Deploy site
        id: deploy-step
        uses: ./.github/actions/deploy-s3-javascript
        # set needed AWS keys as env variables for authenticated access to your AWS
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        with:
          bucket: dummy_name
          dist-folder: ./dist
          bucket-region: use-east-2
      - name: Output information
        run: |
          echo "Live URL: ${{ steps.deploy-step.outputs.website-url }}"
  information:
    runs-on: ubuntu-latest
    steps:
      # use custom JavaScript action
      # have to checkout the repo code to make it available - if custom action is stored in same repo
      # checkout NOT needed, if it comes from another repo
      - name: Get code
        uses: actions/checkout@v3
      - name: Run custom JS action
        uses: ./.github/actions/deploy-s3-javascript
    # ...
```

#### Example Docker Action

- is similar to JavaScript action: look at example in specific GitHub practice repository

## Examples of Basic Workflows

- `GitHub` -> `Actions` -> there you find all workflows listed
- workflow configuration should be stored in `<repo-name>/.github/workflows/<config-file>.yml`
- workflos are executed when their events are triggered
- `GitHub` provides detailed insights into job execution and logs

### Workflow 1

```yml
# name of workflow
name: First Workflow
# event(s) that triggers workflow (event = workflow trigger)
# "workflow_dispatch" means that workflow is started manually
on: workflow_dispatch
# define jobs
jobs:
  # custom name of job
  first-job:
    # define runner environment: https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources
    runs-on: ubuntu-latest
    # define steps
    steps:
      - name: Print greeting
        run: echo "Hello World!"
      - name: Print goodbye
        # run multiple or multi-line shell commands with | symbol
        run: |
          echo "Done"
          echo "Bye"
```

### Workflow 2: to run tests of react app

```yml
name: Test Project
on: push
jobs:
  test:
    # list of preinstalled software of specific runner: https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#preinstalled-software
    # Node is already installed
    runs-on: ubuntu-latest
    steps:
      # steps run on machines/servers in a specific environment, NOT inside the repo
      # so you need to make code available for server
      - name: Get code
        # Action to checkout the repo: https://github.com/actions/checkout
        # use 'uses' keyword for an action, 'run' for a command
        uses: actions/checkout@v3
        # configure action with 'with' keyword
        # with: -> here not needed since default config is fine
      # install Node NOT needed here, but to show like it would be installed on server
      - name: Install NodeJS
        uses: actions/setup-node@v3
        with:
          node-version: 18
      - name: Install dependencies
        # 'npm ci' install all dependencies locked into package-lock.json
        # 'npm i' install all dependencies in package.json
        run: npm ci
      - name: Run tests
        run: npm test
```

### Workflow 3: to run tests and deploy react app

```yml
name: Deploy Project
on: [push, workflow_dispatch] # add multiple events that trigger workflow
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install NodeJS
        uses: actions/setup-node@v3
        with:
          node-version: 18
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test
  deploy:
    # this job waits until 'test' job is successfully finished
    needs: test # multiple jobs: [test, job2]
    # every job gets its own runner - its own virtual machine that's totally isolated from other machines and jobs
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install NodeJS
        uses: actions/setup-node@v3
        with:
          node-version: 18
      - name: Install dependencies
        run: npm ci
      - name: Build project
        run: npm run build
      - name: Dploy
        # normally you need to deploy on real hosting server
        run: echo "Deploying ..."
```

### Workflow 4: with basic expression and context integration

- Docu of available contexts (`github`, `steps`, `needs`, `secrets`, `env` etc.): <https://docs.github.com/en/actions/learn-github-actions/contexts>
- Docu of available expressions (e.g. `${{ toJSON(github) }}`): <https://docs.github.com/en/actions/learn-github-actions/expressions>

```yml
name: Output information
on: workflow_dispatch
jobs:
  info:
    runs-on: ubuntu-latest
    steps:
      - name: Output GitHub context
        # output inserted values of a context: here github key (-> stands for the context object)
        run: echo "${{ toJSON(github) }}" # outputs github context
```

### Workflow 5: triggered by something Issue-related in repository

```yml
name: Handle issues
# workflow is triggered when something issue-related happens in repo
on: issues
jobs:
  output-info:
    runs-on: ubuntu-latest
    steps:
      - name: Output event details
        # access event property of github context object
        # in real case: you could use event data to do something with it
        run: echo "${{ toJSON(github.event) }}"
```

## Jobs in Parallel vs Sequential

- `jobs` run in parallel by default, add `needs` keywords to make this job sequential (-> depending on another)

```yml
# ...
jobs:
  deploy:
    # this job waits until 'test' job is successfully finished
    needs: test # multiple jobs: [test, job2]
```

## Jobs Artifacts & Outputs

- `Job Artifacts`: are output asset(s) (i.e. files, folders) of a job (e.g. App binary, website files, log files etc.)
  - download artifacts and use them manually (via `GitHub UI` or `REST API`)
  - download artifacts and use them in other `jobs` (via `Action`)

```yml
name: Deploy example
on:
  push:
    branches:
      - main
jobs:
  # test:
  # ... not included here
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Build website
        run: npm run build
      # Store output (-> job artifact) using available upload-artifact action
      # https://github.com/actions/upload-artifact
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          # give artifact a name -> can be downloaded manually in GitHub -> Actions ...
          name: dist-files
          # define path of your repo to be stored
          path: dist
          # define multiple path (-> use | for multiline instructions) to be stored
          # path: |
          #   dist
          #   package.json
  deploy:
    # need to run after build step that artifact is ready to use
    needs: build
    runs-on: ubuntu-latest
    steps:
      # take artifacts of previous job
      # actions/download-artifact
      - name: Get build artifacts
        uses: actions/download-artifact@v3
        with:
          # identifier for which artifact to download, unzip and directly navigate into this folder
          name: dist-files
      - name: Output contents
        # only to show that you are directly navigated into dist folder of artifact
        run: ls
      - name: Deploy
        run: echo "Deploying..."
```

- `Job Outputs`: simple values
  - is typically used for re-using a value in different jobs
  - example: name of a file generated in a previous build step
  - to use in command: `echo '{output-name}={output-value}' >> $GITHUB_OUTPUT`

```yml
name: Deploy example
on:
  push:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    # define that this job will have an output (i.e. value(s))
    outputs:
      # custom identifier key, using special 'steps' context object, use your defined id (look below)
      # to access your value
      script-file: ${{ steps.publish.outputs.foo }}
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Install dependencies
        run: npm ci
      - name: Build website
        run: npm run build
      - name: Publish JS filename
        id: publish
        # Linux Command to find name of JS file in assets folder and print it
        # to store this file name, add "foo=" before output value (here "{}")
        # and add >> $GITHUB_OUTPUT after value (targets special file that GitHub creats in your environment
        # where output key value pair is written to)
        run: find dist/assets/*.js -type f -execdir echo 'foo={}' >> $GITHUB_OUTPUT ';'
  output:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Output filename
        # output stored js filename of build job; needs obj contains all outputs of current jobs
        run: echo "${{ needs.build.outputs.script-file }}"
```

## Caching

> GitHub Action for Caching with Examples: <https://github.com/actions/cache>

> Documentation for Cache dependencies: <https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows>

- helps speed up repeated, slow `steps`
- typical use-case is caching dependencies, but any files and folders can be cached
- data is cached once and is available for other `jobs` or across `workflow` executions
  - cache `action` automatically stores and updates cache values (based on the `cache key`)
- do NOT use caching for `artifacts`

```yml
name: Deploy example
on:
  push:
    branches:
      - main
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      # Cache dependencies: needs action before dependencies installation
      # AND inserted in every job that needs the same centrally cached data
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          # define path(s) that includes folder(s) that should be stored in some place
          # in GitHub cloud across different jobs
          path: ~/.npm
          # define key to re-use cached data OR to discard data
          # hashFiles() creates unique hash value based on a file path passed as argument
          # idea: whenever package-lock.json was updated, new hash would be different and key would become invalid
          key: deps-node-modules-${{ hashFiles('**/package-lock.json')}}
      - name: Install dependencies
        run: npm ci
      # ...
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json')}}
      - name: Install dependencies
        run: npm ci
      # ...
```

## Environment Variables & Secrets

> Docu of GitHub Actions: <https://docs.github.com/en/actions/learn-github-actions/environment-variables>

> Default environment variables that GitHub sets automatically and that are available to every `step` in a `workflow`: <https://docs.github.com/en/actions/learn-github-actions/environment-variables#default-environment-variables>

```javascript
// credential variables can differ depending on the current environment (e.g. testing and production)
const dbPassword = process.env.DB_PASSWORD;
```

- use `secrets` to avoid that some env variable values are exposed (e.g. database password, API keys)

  - go to `GitHub` repo -> `Settings` -> `Secrets` -> `Actions`
  - there you can set env variables that will be available in all workflows
  - these variables can NOT be viewed again by a person having access to your repo

- GitHub `Environments`: e.g. set a `testing`, `staging`, or `production` environment for a specific repo
  - go to `GitHub` repo -> `Settings` -> `Environments` (available in public or private paid repositories)
  - `workflow jobs` can target `Environments` with key `environment`: so you can provide
    - different configurations like `different secrets` for different environments (e.g. different database passwords, different database addresses)
    - special `protection rules`:
      - only specified reviewer(s) should approve workflow runs
      - only events related to certain branches can use an environment (e.g. only pushes from `main` branch can trigger this environment)

```yml
name: Deployment
on:
  push:
    branches:
      - main
      - dev
# defining env variable on workflow level -> they will be available in all jobs
env:
  # use your defined variable name of process.env.NAME as key and set the value
  MONGODB_DB_NAME: dummy-name
jobs:
  test:
    # set an environment configured in GitHub Repo -> Settings -> Environments
    # env variables are pulled from this environment
    environment: testing
    # set env variable on job level (-> also possible on step level)
    env:
      MONGODB_CLUSTER_ADDRESS: dummy_cluster_address
      # reference to secrets stored in GitHub repo with secret object context
      MONGODB_USERNAME: ${{ secrets.MONGODB_USERNAME}}
      MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD}}
      PORT: 3000
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: npm-deps-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Run server
        # a) inject env variable into command (-> format for Linux Bash Shell used on ubuntu runner: $NAME)
        # you could also change the default shell to Windows Powershell etc.: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsshell
        run: npm start & npx wait-on http://127.0.0.1:$env:PORT
      - name: Output information
        # b) output env variable with GitHub env object context
        # 'env.MONGODB_USERNAME' is automatically hidden by GitHub because it's set as a secret
        run: |
          echo "MONGODB_USERNAME: ${{ env.MONGODB_DB_NAME }}"
          echo "MONGODB_USERNAME: ${{ env.MONGODB_USERNAME }}"
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        run: |
          echo "MONGODB_DB_NAME: ${{ env.MONGODB_DB_NAME }}"
```

## Controlling Execution Flow

- default behavior: workflow fails if one step failed
- `if`: you can add `conditions` to `jobs` and `steps` via `if` field
  - job continues if condition in steps is fullfilled, but failed step is also treated as failed and so following workflow is by default stopped
- `continue-on-error`: `steps` does have a `continue-on-error` field to ignore errors
  - job should continue execution even if step fails
  - treats `step` as succeeded despite it is technically failing
  - Documentation for `steps context object`: <https://docs.github.com/en/actions/learn-github-actions/contexts#steps-context>
- evaluate `conditions` via `expressions`

- `special condition functions`:
  - `failure()`: returns `true` when any prev `step` or `job` failed
  - `success()`: returns `true` when none of prev `steps` failed
  - `always()`: causes the `step` to always execute, even when cancelled
  - `cancelled()`: returns `true` if `workflow` has been cancelled

```yml
# Example with if condition and special condition functions
name: Website Deployment
on:
  push:
    branches:
      - main
jobs:
  # ...
  # ...
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Cache dependencies
        id: cache
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        # check if dependencies installation is needed, otherwise skip 'npm ci'
        # check if cached dependencies matches the needed deependencies
        # use "id: cache" vie steps object context to access output ('true'/'false') of actions/cache@v3
        if: ${{ steps.cache.outputs.cache-hit != 'true'}}
        run: npm ci
      - name: Test code
        # id can be added to every step to identify it
        id: run-tests
        run: npm run test
      # upload should be executed if test fails, but by default
      # all following steps AND all jobs based on this job are stopped
      - name: Upload test report
        # add condition with a dynamically evaluated expression with steps object context
        # here: you can also omit surrounding ${{ }}
        # conditional functions: failure() needed to change default behavior of stopping execution if step fails
        # operators: https://docs.github.com/en/actions/learn-github-actions/expressions#operators
        if: ${{ failure() && steps.run-tests.outcome == 'failure'}}
        uses: actions/upload-artifact@v3
        with:
          name: test-report
          # test.json summary is produced by script (-> package.json)
          path: test.json
  # ...
  # ...
  report:
    # only starts after 'lint' and 'deploy' (which depends on 'build' and 'test') finished
    needs: [lint, deploy]
    # only executed if any other job or step failed
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        run: |
          echo "Something went wrong"
          echo "${{ toJSON(github) }}"
```

```yml
# Example with continue-on-error
name: Continue Website Deployment
on:
  push:
    branches:
      - main
jobs:
  # ...
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Cache dependencies
        id: cache
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        if: ${{ steps.cache.outputs.cache-hit != 'true'}}
        run: npm ci
      - name: Test code
        # job should continue execution even if step fails
        # treats step as succeeded despite it is technically failing
        # Docu for steps object context: https://docs.github.com/en/actions/learn-github-actions/contexts#steps-context
        continue-on-error: true
        id: run-tests
        run: npm run test
      - name: Upload test report
        uses: actions/upload-artifact@v3
        with:
          name: test-report
          path: test.json
  # ...
```

## Matrix Workflow

- with a `matrix` you can run the same job for different configurations (e.g. `runners`, `node versions`)

```yml
name: Matrix Demo
on: push
jobs:
  build:
    continue-on-error: true
    strategy:
      matrix:
        node-version: [14, 16]
        operating-system: [ubuntu-latest, windows-latest]
        # include specific standalone combination as job environment
        include:
          - node-version: 18
            operating-system: ubuntu-latest
            # ... you can also add more keys that are NOT defined for matrix above
        # exclude specific standalone combination
        exclude:
          - node-version: 14
            operating-system: windows-latest
    # use matric context object to access defined matrix values
    # job will then run multiple times in parallel: once per matrix value
    # AND for ALL matrix value combinations, if you have multiple
    runs-on: ${{ matrix.operating-system }}
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Install NodeJS
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - name: Install Dependencies
        run: npm ci
      - name: Build project
        run: npm run build
```

## Reusable Workflows

- re-use `workflow A` inside `workflow B`
- use `workflow_call` event
- work with `inputs`, `outputs` and `secrets` as required

```yml
# 09_reusable-workflow.yml
name: Reusable Deploy
# make workflow callable for other workflows
on:
  workflow_call:
    # add inputs this workflow needs to execute somewhere else
    inputs:
      artifact-name:
        description: The name of the deployable artifact files
        # if required to succeed workflow
        required: false
        # if 'required: false' -> you can set default value for 'artifact-name' key
        default: dist
        type: string # or number, boolean
      # ... other keys as you like and need
    # set outputs of this workflow
    outputs:
      result:
        description: The result olf the deployment operation
        # use jobs context object
        value: ${{ jobs.deploy.outputs.outcome }}

    # add secrets this workflow needs to execute
    # secrets:
    #   some-secret:
    #     required: false
jobs:
  deploy:
    # define outputs of this job
    outputs:
      outcome: ${{ steps.set-result.outputs.step-result }}
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/download-artifact@v3
        with:
          # use inputs context object
          name: ${{ inputs.artifact-name }}
      - name: List files
        run: ls
      - name: Output information
        run: echo "Deploying and uploading ..."
      - name: Set result output
        id: set-result
        run: echo "step-result=success" >> $GITHUB_OUTPUT
```

```yml
# 10_use-reusable-workflow.yml
name: Using Reusable Workflow
on:
  push:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Cache dependencies
        id: cache
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: dependencies-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: List the state of node modules
        if: ${{ steps.cache.outputs.cache-hit != 'true'}}
        run: npm list
      - name: Build website
        id: build-website
        run: npm run build
      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist-files
          path: dist
  deploy:
    needs: build
    # to re-use workflow, point to path of re-usable workflow
    # here: relative path inside current repo
    # also possible: absolute path to another rep
    uses: ./.github/workflows/09_reusable-workflow.yml
    with:
      # provide needed input values to execute re-usable workflow
      artifact-name: dist-files
    # pass a secret to workflow
    # secrets:
    #   some-secret: ${{ secrets.some-secret }}
  print-deploy-result:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: Print deploy output
        # needs context object gives access to job you are depending on
        # properties 'outputs.results' -> look at reusable-workflow.yml
        run: echo "${{ needs.deploy.outputs.result }}"
```

## Jobs & Docker Containers

- `docker container`:
  - packages that contain `code` and its `execution environment`
  - reproducible and isolated execution environment and results
  - example: same environment for testing + production
  - you build `images` based on a `Dockerfile` definition
- with `GitHub Actions` you can run your workflows
  - just on the `Runner` machine
    - choose frome list of pre-defined environments incl. installed software (e.g. `ubuntu-latest`)
  - or in a `container`
    - full control over environment and installed software
    - a containerized job is hosted by the `Runner` and the `Steps` execute inside the container
    - build your own container `images` or use public `images`
    - you can also create `Services`: Utility containers used by your `Steps` (e.g. testing database)
- `Service Containers` (`Services`)
  - example: run tests
  - problem: tests should not manipulate the production database
  - solution: use a testing database
    - `service container` hosts a testing database
    - runs inside a container (hosted by the `Runner`)
    - job `steps` can communicate with service containers (and the services exposed by them)

### Docker Container Example with Service Container in Workflow

```yml
name: Deployment (Container)
on:
  push:
    branches:
      - main
      - dev
env:
  CACHE_KEY: node-deps
  MONGODB_DB_NAME: gha-demo
jobs:
  test:
    environment: testing
    # Runner machine will now only host container below
    runs-on: ubuntu-latest
    container:
      # use your own docker container
      # OR available images on docker hub: https://hub.docker.com/search?q=&type=image
      image: node:16
      # you can add env variables that are needed by image (e.g. look at image docu)
      # env:
    env:
      # V1: without services testing database
      # Data does NOT work - only dummy data, workflow fails
      # MONGODB_CONNECTION_PROTOCOL: mongodb+srv
      # MONGODB_CLUSTER_ADDRESS: cluster0.15pwqcc.mongodb.net
      # MONGODB_USERNAME: ${{ secrets.MONGODB_USERNAME }}
      # MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD }}

      # V2: with services testing database
      # protocol adjustement as written in image docu
      MONGODB_CONNECTION_PROTOCOL: mongodb
      # a) if job runs in a container: use services label below to establish a network connection
      MONGODB_CLUSTER_ADDRESS: mongodb
      # b) if job runs without a container: use localhost IP address
      # MONGODB_CLUSTER_ADDRESS: 127.0.0.1:27017

      # you have to use dummy credentials (look below)
      MONGODB_USERNAME: root
      MONGODB_PASSWORD: example
      PORT: 8080
    # add a container service (e.g. to host a testing database) in a job
    services:
      # custom identifier
      mongodb:
        # a service always runs into an image (here: 'mongo' image of docker hub)
        image: mongo

        # if V2 b) job runs without a container: open port of localhost IP address
        # ports:
        #   # internal port 27017 should be forwarded to port on Runner machine
        #   - 27017:27017
        # insert needed credentials for mongo db
        # notice: this database is only running as long as the workflow is running
        # so it can NOT be used by anybody else and exposing dummy credentials is possible

        env:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: example
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ env.CACHE_KEY }}-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Run server
        run: npm start & npx wait-on http://127.0.0.1:$PORT # requires MongoDB Atlas to accept requests from anywhere!
      - name: Run tests
        run: npm test
      - name: Output information
        run: |
          echo "MONGODB_USERNAME: $MONGODB_USERNAME"
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        env:
          PORT: 3000
        run: |
          echo "MONGODB_DB_NAME: $MONGODB_DB_NAME"
          echo "MONGODB_USERNAME: $MONGODB_USERNAME"
          echo "${{ env.PORT }}"
```
