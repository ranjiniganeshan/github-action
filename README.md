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

This workflow shows how environment variable scopes work in GitHub Actions:

* WORKFLOW_VAR is available to all jobs and steps.

* JOB_VAR is only available within job-1.

* STEP_VAR is available only in the specific step where it’s defined.
  
* Each step echoes which variables are visible to show the scope differences.

## Passing data between steps and jobs

1. create a file called passing-data.yaml in github/workflows/

```

name: Passing Variables Between jobs
on:
  workflow_dispatch:

jobs:
  producer:
    runs-on: ubuntu-24.04

    outputs:
      foo: ${{ steps.generate-foo.outputs.foo }}
    steps:
      - name: Generate and export foo
        id: generate-foo
        run: |
          foo=bar

          # 1) Step output (for job output)
          echo "foo=${foo}" >> "$GITHUB_OUTPUT"

          # 2) Job-scoped environment variable
          echo "FOO=${foo}" >> "$GITHUB_ENV"

      - name: Inspect values inside producer
        run: |
          echo "FOO (set via GITHUB_ENV):   $FOO"
          echo "foo (step output):          ${{ steps.generate-foo.outputs.foo }}"

  consumer:
    runs-on: ubuntu-24.04
    needs: producer
    steps:
      - name: Inspect values inside consumer (note FOO is unset)
        run: |
          echo "Value from producer:        ${{ needs.producer.outputs.foo }}"
          echo "FOO in consumer:            ${FOO:-<UNSET>}"

```

This workflow demonstrates how to pass data between jobs in GitHub Actions:

* The producer job creates a variable foo=bar.

* It saves it as a step output ($GITHUB_OUTPUT) so it can be shared with other jobs.

* It also sets it as a job environment variable ($GITHUB_ENV), available only within that job.

* The consumer job runs after the producer (needs: producer) and accesses the foo value through ${{ needs.producer.outputs.foo }}.

* Regular environment variables like FOO don’t persist between jobs.

## Secrets, variables, and environments
```
1. create a file called secrets-and-variables.yaml in github/workflows/

name: Secrets and Environments
on:
  workflow_dispatch:

jobs:
  staging-environment:
    runs-on: ubuntu-24.04
    environment: staging
    env:
      # Inject repository-level secret & variable into the shell
      EXAMPLE_REPOSITORY_SECRET: ${{ secrets.EXAMPLE_REPOSITORY_SECRET }}
      EXAMPLE_REPOSITORY_VARIABLE: ${{ vars.EXAMPLE_REPOSITORY_VARIABLE }}
      # Inject environment-level items into the shell
      EXAMPLE_ENVIRONMENT_SECRET: ${{ secrets.EXAMPLE_ENVIRONMENT_SECRET }}
      EXAMPLE_ENVIRONMENT_VARIABLE: ${{ vars.EXAMPLE_ENVIRONMENT_VARIABLE }}
    steps:
      - name: Inspect values inside job
        run: |
          echo "Repo secret (masked):    $EXAMPLE_REPOSITORY_SECRET"
          echo "Repo variable:           $EXAMPLE_REPOSITORY_VARIABLE"
          echo "Env secret (masked):     $EXAMPLE_ENVIRONMENT_SECRET"
          echo "Env variable:            $EXAMPLE_ENVIRONMENT_VARIABLE"

  production-environment:
    runs-on: ubuntu-24.04
    environment: production
    env:
      # Inject repository-level secret & variable into the shell
      EXAMPLE_REPOSITORY_SECRET: ${{ secrets.EXAMPLE_REPOSITORY_SECRET }}
      EXAMPLE_REPOSITORY_VARIABLE: ${{ vars.EXAMPLE_REPOSITORY_VARIABLE }}
      # Inject environment-level items into the shell
      EXAMPLE_ENVIRONMENT_SECRET: ${{ secrets.EXAMPLE_ENVIRONMENT_SECRET }}
      EXAMPLE_ENVIRONMENT_VARIABLE: ${{ vars.EXAMPLE_ENVIRONMENT_VARIABLE }}
    steps:
      - name: Inspect values inside job
        run: |
          echo "Repo secret (masked):    $EXAMPLE_REPOSITORY_SECRET"
          echo "Repo variable:           $EXAMPLE_REPOSITORY_VARIABLE"
          echo "Env secret (masked):     $EXAMPLE_ENVIRONMENT_SECRET"
          echo "Env variable:            $EXAMPLE_ENVIRONMENT_VARIABLE"
```

### How to Add Secrets and Variables ** 

* Go to your GitHub repository → click on Settings.

* On the left panel, under Security, select Secrets and variables → Actions.

* You’ll see two tabs:

* Secrets → For sensitive data (like API keys, passwords).

* Variables → For non-sensitive config values (like environment names, versions).

* Add Repository-Level Secrets/Variables:

* Click New repository secret or New repository variable.

### Example:

* Secret name: EXAMPLE_REPOSITORY_SECRET

* Variable name: EXAMPLE_REPOSITORY_VARIABLE

* Add Environment-Level Secrets/Variables: 

* Under Environments → create environments like staging and production.

* Inside each environment, add:

* Secret: EXAMPLE_ENVIRONMENT_SECRET

* Variable: EXAMPLE_ENVIRONMENT_VARIABLE

** How the Workflow Works ** 

## The workflow has two jobs: 

* staging-environment (uses environment staging)

* production-environment (uses environment production)

Each job defines environment variables (env:) that reference secrets and variables using:

${{ secrets.SECRET_NAME }}
${{ vars.VARIABLE_NAME }}


During runtime:

GitHub injects the appropriate repository-level and environment-level secrets/variables into the job’s shell.

The secrets’ actual values are masked in logs (you’ll see *** instead of the real value).

Variables are printed normally.

Finally, each job prints the injected values to show the difference between repository- and environment-level configurations.

## three main types of GitHub Action runners

* GitHub-Hosted Runner

1. create a workflow github_hosted_runner.yaml file in .github/workflows
```
name: GitHub Hosted Runner Demo
on: workflow_dispatch

jobs:
  github-hosted:
    runs-on: ubuntu-latest
    steps:
      - name: Show environment info
        run: |
          echo "Running on a GitHub-hosted Ubuntu machine"
          uname -a
          df -h
```

* Self-Hosted Runner

These are your own machines (e.g., EC2, on-prem, or VM) that you connect to GitHub.
You install the runner software and register it to your repo/org.
1. create a file called self-hosted-runner.yaml within .github/workflows
```
name: Self Hosted Runner Demo
on: workflow_dispatch

jobs:
  self-hosted:
    runs-on: self-hosted
    steps:
      - run: echo "Running on a self-hosted runner (my own machine)"
```
#### Steps to add the self hosted runner

*  Launch an EC2 Instance
* Connect to the EC2 Instance and install the below tools
```
sudo apt update
sudo apt install -y curl tar
```
* Register the Runner with GitHub
Go to your repository in GitHub.

Navigate to:
Settings → Actions → Runners → New self-hosted runner → New runner

Choose Linux and copy the commands GitHub provides — they look like this:


<img width="794" height="541" alt="Screenshot 2025-10-08 at 8 57 06 PM" src="https://github.com/user-attachments/assets/6297cb37-8324-41e3-a9a3-19d9f15d4998" />





<img width="1133" height="553" alt="Screenshot 2025-10-08 at 8 59 51 PM" src="https://github.com/user-attachments/assets/66057a00-e8dd-40e1-867c-372272424784" />

<img width="466" height="403" alt="Screenshot 2025-10-08 at 9 00 24 PM" src="https://github.com/user-attachments/assets/6fdbd4f1-dba6-4fa7-a82c-0950066a495a" />


## Persisting build outputs with artifacts

Artifacts are files shared between jobs or stored after workflow completion.

They’re uploaded using the actions/upload-artifact action and downloaded using actions/download-artifact.


1. create a file build-deploy-artifact.yaml

```
name: Artifacts Demo
on: workflow_dispatch

jobs:
  # Job 1: Create and upload an artifact
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Create a sample file
        run: |
          echo "Build output created on $(date)" > build-output.txt
          echo "Artifact contains build results" >> build-output.txt

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: build-artifact
          path: build-output.txt

  # Job 2: Download and use the artifact
  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: build-artifact

      - name: Display artifact content
        run: cat build-output.txt

  # Job 3: Demonstrate artifact persistence after workflow
  deploy:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - name: Download artifact again
        uses: actions/download-artifact@v4
        with:
          name: build-artifact

      - name: Use artifact data
        run: |
          echo "Deploying using data from artifact..."
          cat build-output.txt
```
##### How It Works

- build job

  * Creates a text file (build-output.txt).

  * Uploads it as an artifact named build-artifact.

-  test job

   * Waits for build (needs: build).

   * Downloads the same artifact and prints its contents.

- deploy job

  * Waits for test (needs: test).

  * Downloads the artifact again and uses it (simulating a deployment).

After the workflow completes, you’ll see the uploaded artifact under
GitHub → Actions → Run → Artifacts → build-artifact.zip
— you can download it manually anytime.


## Reusing work with caches

simple caching

1. create a workflow file called simple-caching.yaml in .github/workflows

```
name: Cache Demo
on: workflow_dispatch

jobs:
  actions-cache:
    name: Prime demo cache
    runs-on: ubuntu-24.04
    steps:
      #  Try to restore existing cache
      - name: Restore cache
        id: demo-cache
        uses: actions/cache@v4
        with:
          path: demo-cache
          key: demo-cache-v1

      # If cache not found, create and populate directory
      - name: Populate cache directory
        if: steps.demo-cache.outputs.cache-hit != 'true'
        run: |
          echo "Cache miss – generating contents"
          mkdir -p demo-cache
          date > demo-cache/timestamp.txt

      #  Save the cache for future runs
      - name: Save cache
        if: steps.demo-cache.outputs.cache-hit != 'true'
        uses: actions/cache@v4
        with:
          path: demo-cache
          key: demo-cache-v1

      #  Show cache status
      - name: Verify cache status
        run: |
          echo "cache-hit? -> ${{ steps.demo-cache.outputs.cache-hit }}"
          cat demo-cache/timestamp.txt
```
details of the worflow

Step 1: The workflow tries to restore a cache folder named demo-cache using the key demo-cache-v1. → First run = cache miss, next runs = cache hit.

Step 2: If no cache exists, it creates a folder and writes the current date.

Step 3: The new data is saved to cache with the same key (demo-cache-v1).

Step 4: On future runs, the cached data (timestamp) is restored instantly — showing reuse of previous work.

## Controlling GitHub permissions
1. create a file github-permissions.yaml in .github/workflows directory


```
name: GitHub Actions Permissions 

on:
  workflow_dispatch:    # allows manual trigger from Actions tab
  pull_request:         # still allows running on PR events

permissions:
  contents: read
  pull-requests: write

jobs:
  comment:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Add comment to PR
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: "Thanks for your pull request! The workflow ran successfully."
            })

```
2. create a branch name test and checkin a test file.
3. create a PR for main and test branch. Merge the PR.
```
permissions:
  contents: read
  pull-requests: write

```
* GitHub Actions runs with a GITHUB_TOKEN, which can have fine-grained permissions.
    - This block defines what that token can do inside this workflow:

        * contents: read → allows the workflow to read repository contents (needed for checkout and accessing files).

        * pull-requests: write → allows the workflow to write to pull requests — such as posting comments or updating statuses.

Without this, your script wouldn’t be able to create a PR comment.


## Matrix strategies, conditionals, and concurrency controls
1. create a file matrix-condition-concurrency.yaml in .github/workflows

```
name: Matrix + Conditional + Concurrency Demo

on:
  push:
    branches:
      - main
      - dev
  workflow_dispatch:    # allows manual runs

# Concurrency control – only one workflow per branch at a time
concurrency:
  group: build-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build-and-test:
    name: Build & Test (${{ matrix.os }}, Node ${{ matrix.node }})
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node: [18, 20]

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}

      - name: Dummy build
        run: echo "Simulating build on ${{ matrix.os }} with Node ${{ matrix.node }}"

      - name: Dummy test
        run: echo "Simulating tests on ${{ matrix.os }} with Node ${{ matrix.node }}"

  deploy:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build-and-test
    #  Conditional: only deploy from main branch
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Deploy app
        run: echo " Deploying application from main branch..."

```

#### How It Works

* Matrix Strategy

* build-and-test runs 4 parallel jobs (Ubuntu + Windows × Node 18 + 20).

- Each job prints which OS & Node it is simulating.

* Conditionals

  - deploy runs only if branch is main.

  - Skips on dev or other branches.

* Concurrency Control

  - If multiple pushes happen quickly on the same branch, older runs are cancelled.

* Prevents duplicate deployments.

   - Dummy Steps

   - echo replaces npm install/build/test, so the workflow runs successfully even without a Node project.
