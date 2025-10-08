## Github Actions

GitHub Actions is a continuous integration and continuous delivery (CI/CD) platform that allows you to automate your build, test, and deployment pipeline. You can create workflows that run tests whenever you push a change to your repository, or that deploy merged pull requests to production.

## Advantages of using github actions
* Strengths that keep teams on Actions
* Low friction for GitHub-hosted code: Pipelines live alongside your code; creating a workflow is as simple as committing a YAML file under .github/workflows/.
* Vibrant marketplace: Thousands of community-maintained actions help you automate everything from dependency caching to cloud deployments without re-writing common logic.

## Workflow

### Creating an example workflow
GitHub Actions uses YAML syntax to define the workflow. Each workflow is stored as a separate YAML file in your code repository, in a directory named .github/workflows.

You can create an example workflow in your repository that automatically triggers a series of commands whenever code is pushed. In this workflow, GitHub Actions checks out the pushed code, installs the bats testing framework, and runs a basic command to output the bats version: bats -v.

In your repository, create the .github/workflows/ directory to store your workflow files.

In the .github/workflows/ directory, create a new file called learn-github-actions.yml and add the following code.
```
name: learn-github-actions
run-name: ${{ github.actor }} is learning GitHub Actions
on: [push]
jobs:
  check-bats-version:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm install -g bats
      - run: bats -v

```


Commit these changes and push them to your GitHub repository.

Your new GitHub Actions workflow file is now installed in your repository and will run automatically each time someone pushes a change to the repository



## Understanding the workflow file


To help you understand how YAML syntax is used to create a workflow file, this section explains each line of the introduction's example:


name: learn-github-actions
Optional - The name of the workflow as it will appear in the "Actions" tab of the GitHub repository. If this field is omitted, the name of the workflow file will be used instead.

run-name: ${{ github.actor }} is learning GitHub Actions
Optional - The name for workflow runs generated from the workflow, which will appear in the list of workflow runs on your repository's "Actions" tab. This example uses an expression with the github context to display the username of the actor that triggered the workflow run. For more information, see Workflow syntax for GitHub Actions.

on: [push]
Specifies the trigger for this workflow. This example uses the push event, so a workflow run is triggered every time someone pushes a change to the repository or merges a pull request. This is triggered by a push to every branch; for examples of syntax that runs only on pushes to specific branches, paths, or tags, see Workflow syntax for GitHub Actions.

jobs:
Groups together all the jobs that run in the learn-github-actions workflow.

  check-bats-version:
Defines a job named check-bats-version. The child keys will define properties of the job.

    runs-on: ubuntu-latest
Configures the job to run on the latest version of an Ubuntu Linux runner. This means that the job will execute on a fresh virtual machine hosted by GitHub. For syntax examples using other runners, see Workflow syntax for GitHub Actions

    steps:
Groups together all the steps that run in the check-bats-version job. Each item nested under this section is a separate action or shell script.

      - uses: actions/checkout@v5
The uses keyword specifies that this step will run v4 of the actions/checkout action. This is an action that checks out your repository onto the runner, allowing you to run scripts or other actions against your code (such as build and test tools). You should use the checkout action any time your workflow will use the repository's code.

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
This step uses the actions/setup-node@v4 action to install the specified version of the Node.js. (This example uses version 20.) This puts both the node and npm commands in your PATH.

      - run: npm install -g bats
The run keyword tells the job to execute a command on the runner. In this case, you are using npm to install the bats software testing package.

      - run: bats -v
Finally, you'll run the bats command with a parameter that outputs the software version.

## Jobs, dependencies, and triggers

### cordinating multiple jobs

1. create a file called  multiple-workflows-jobs-steps.yaml in folder github/workflows/

```
name: Workflows, Jobs, and Steps

on:
  workflow_dispatch:
jobs:
  job-1:
    runs-on: ubuntu-24.04
    steps:
      - run: echo "A job consists of"
      - run: echo "one or more steps"
      - run: echo "which run sequentially"
      - run: echo "within the same compute environment"
  job-2:
    runs-on: ubuntu-24.04
    steps:
      - run: echo "Multiple jobs can run in parallel"
  job-3:
    runs-on: ubuntu-24.04
    needs:
      - job-1
      - job-2
    steps:
      - run: echo "They can also depend on one another..."
  job-4:
    runs-on: ubuntu-24.04
    needs:
      - job-2
      - job-3
    steps:
      - run: echo "...to form a directed acyclic graph (DAG)"
```

The workflow runs manually (workflow_dispatch).

* Each job runs on an Ubuntu runner (runs-on: ubuntu-24.04).

* Steps are shell commands (run:) executed sequentially within a job.

* Jobs (job-1, job-2, job-3, job-4) can run in parallel or depend on others using needs.

* The dependencies (needs) form a directed acyclic graph (DAG) showing execution order.

## Controlling when workflows run

1. create a file called  triggers-and-filters.yaml in folder github/workflows/


```
name: Triggering Events
# https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows

on:
  ### https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#push
  push:
    branches:
      # Filter patterns cheat sheet:
      #   https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#filter-pattern-cheat-sheet
      - "example-branch/*"

  ### https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#pull_request
  pull_request:
    ## The default is to trigger on the following types of events:
    types:
      - opened
      - synchronize
      - reopened
    paths:
      # include markdown files
      - "03-core-features/filters/*.md"
      # Exclude txt files
      - "!03-core-features/filters/*.txt"

  ### https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#schedule
  # schedule:
  #   - cron: "0 0 * * *" # Midnight UTC

  ## Manual Trigger
  workflow_dispatch:
jobs:
  echo-event:
    runs-on: ubuntu-24.04
    steps:
      - name: Dump full event JSON
        run: |
          echo "=== $GITHUB_EVENT_NAME payload ==="
          cat "$GITHUB_EVENT_PATH"


```

This workflow runs automatically based on certain GitHub events:

* push → triggers only when code is pushed to branches matching example-branch/*.

* pull_request → runs when a PR is opened, updated, or reopened — but only if it changes .md files (and skips .txt files).

* workflow_dispatch → allows manual execution from the GitHub Actions UI.

* When triggered, it runs one job (echo-event) that simply prints the event details (JSON payload) to the console.

## Environment Variables and Scope
1. create a file called environment-variables.yaml in github/workflows/

```
name: Environment Variables
on:
  workflow_dispatch:
env:
  WORKFLOW_VAR: I_AM_WORKFLOW_SCOPED

jobs:
  job-1:
    runs-on: ubuntu-24.04
    env:
      JOB_VAR: I_AM_JOB_1_SCOPED
    steps:
      - name: Inspect scopes job 1 step 1
        env:
          STEP_VAR: I_AM_STEP_SCOPED
        run: |
          echo "WORKFLOW_VAR: $WORKFLOW_VAR"   # visible
          echo "JOB_VAR:      $JOB_VAR"        # visible
          echo "STEP_VAR:     $STEP_VAR"       # visible only here

      - name: Inspect scopes job 1 step 2
        run: |
          echo "WORKFLOW_VAR: $WORKFLOW_VAR"         # visible
          echo "JOB_VAR:      $JOB_VAR"              # visible
          echo "STEP_VAR:     ${STEP_VAR:-<UNSET>}"  # not set here

  job-2:
    runs-on: ubuntu-24.04
    steps:
      - name: Inspect scopes job 2 step 2
        run: |
          echo "WORKFLOW_VAR: $WORKFLOW_VAR"                 # still visible
          echo "JOB_VAR:      ${FOO:-<UNSET>}"               # not set here
          echo "STEP_VAR:     ${STEP_VAR:-<UNSET>}"          # not set here

```



