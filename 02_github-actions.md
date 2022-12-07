# GitHub Actions

> Cours Repository: <https://github.com/academind/github-actions-course-resources>

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
- pre-defined `runners`(with different operating systems) exist
- you can also create custom `runners`

## Actions

- `command` (-> `run`) is a (typically simple) shell command that's defined by you

- `action`
  - is a (custom) application that performs a (typically complex) frequently repeated task
  - you can build you own `Actions` but you can also use official or community `Actions`
  - `GitHub marketplace` to find `Actions`: <https://github.com/marketplace>

## Jobs in Parallel vs Sequential

- `jobs` run in parallel by default, add `needs` keywords to make this job sequential (-> depending on another)

```yml
# ...
jobs:
  deploy:
    # this job waits until 'test' job is successfully finished
    needs: test # multiple jobs: [test, job2]
```

## Examples of Workflows

- `GitHub` -> `Actions` -> there you find all workflows listed
- workflow configuration should be stored in `<repo-name>/.github/workflows/<config-file>.yml`
- workflos are executed when their events are triggered
- `GitHub` provides detailed insights into job execution and logs

### Basic Workflow

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

### Workflow to Run Tests of React App

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

### Workflow to Run Tests and Deploy React App

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

### Workflow with Basic Expression and Context Integration

- Docu of available contexts (e.g. `github`): <https://docs.github.com/en/actions/learn-github-actions/contexts>
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

### Workflow triggered by something Issue-related in Repository

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
