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
- `Jobs`
  - define a `runner` (i.e. execution environment)
  - contain one or more `steps`
  - if you have multiple `jobs`, they can run in parallel (default) or sequential
  - can run for certain conditions
- `Steps`
  - execute a `shell script` or an `action`
  - can use custom or third-party actions
  - steps are executed in order
  - can run for certain conditions

![](/01_slides/01_workflows-jobs-steps.png)

## Practice Basic Workflow

- `GitHub` -> `Actions`
- workflow configuration should be stored in `<repo-name>/.github/workflows/<config-file.yml>`
- after defining `yml` config file, go to `Actions` where workflows are listed
  - there you can start and evaluate result of workflow

```yml
# name of workflow
name: First Workflow
# event(s) that triggers workflow
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
