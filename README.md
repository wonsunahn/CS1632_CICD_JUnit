- [CS 1632 - Software Quality Assurance](#cs-1632---software-quality-assurance)
  * [Before You Begin](#before-you-begin)
    + [Install Apache Maven](#install-apache-maven)
    + [Install VSCode](#install-vscode)
  * [Description](#description)
  * [Running the Program](#running-the-program)
  * [Running Unit Tests](#running-unit-tests)
    + [The POM Maven build configuration](#the-pom-maven-build-configuration)
  * [Using TDD to Complete the Implementation](#using-tdd-to-complete-the-implementation)
    + [Red Phase](#red-phase)
    + [Green Phase](#green-phase)
    + [Refactor Phase](#refactor-phase)
  * [Verifying the Test Cases](#verifying-the-test-cases)
  * [Measuring Code Coverage](#measuring-code-coverage)
  * [Using VSCode](#using-vscode)
- [Submission](#submission)
- [GradeScope Feedback](#gradescope-feedback)
- [Groupwork Plan](#groupwork-plan)
- [Resources](#resources)

# CS 1632 - Software Quality Assurance
Fall Semester 2022 - Supplementary Exercise 4

* DUE: December 2 (Friday), 2022 11:59 PM

## Description

During the semester, we learned various ways in which we can automate testing.
But all that automation is of no use if your software organization as a whole
does not invoke those automated test scripts diligently.  Preferably, those test
scripts should be run before every single source code change to the repository,
and for good measure, regularly every night or every weekend just in case.  Now,
there are many reasons why this does not happen if left to individual
developers:

1. Developers are human beings so they forget.  Or, they remember to run
   some tests, but not all the test suites that are relevant to the changes
they have made.

1. Developers are sometimes on a tight schedule, so they are tempted to skip
   testing that may delay them, especially if they are not automated.  They
justify their actions by telling themselves that they will fix the failing
tests "as soon as possible", or that the test cases are not testing anything
important, or that failing test cases in modules under the purview of
another team "is not my problem".

In Part 1 of this exercise, we will learn how to build an automated
"pipeline" of events that get triggered automatically under certain
conditions (e.g. a source code push).  A pipeline can automate the entire
process from source code push to software delivery to end users, making sure
that a suite of tests are invooked as part of the process before software is
delivered.  Pipelines that are built for this purpose are called CI/CD
(Continuous Integration / Continuous Delivery) pipelines, because they
enable continuous delivery of software to the end user at at high velocity
while still maintaining high quality.  We will learn how to build a fully
functioning pipeline for the (Rent-A-Cat application)[../exercises/2] that
we tested for Exercise 2 on our GitHub repository.

In Part 2, we will learn how to use dockers to both test and deploy
software as part of a CI/CD pipeline.  Dockers are virtualized execution
environments which can emulate the execution environments in the deployment
sites (OS, libraries, webservers, databases, etc.) so that software can be
tested in situ.  In our case, we will create a docker image out of the
(Rent-A-Cat website)[cs1632.appspot.com] that we tested for Deliverable 3
for testing and deployment.

## Part 1: CI/CD Pipelines

**GitHub Classroom Link:** TBD

In Part 1, you will learn how to create a pipeline from scratch based on the
Rent-A-Cat application for Exercise 2, using the CI/CD support provided by
your GitHub repository through GitHub Actions.  GitHub Actions is just one
example CI/CD framework.  Other widely used CI/CD frameworks include GitLab
Pipelines and Jenkins.  Regardless of which you choose, they work in mostly
similar ways: there is a YAML configuration file that describes actions in
the pipeline and the actions are performed on one or more Runner machines
which are typically Docker containers.  The only thing that differs is the
YAML file syntax.  Hence, by learning GitHub Actions, you will be able to
translate that knowledge to other frameworks as well.  In GitHub Actions
lingo, pipelines are called **workflows**, and the two terms will be used
interchangeably.

### Add Hello World Workflow

Let's first start with a very basic workflow which prints "Hello World"
inside the Runner.  Click on the "Actions" menu on your GitHub repository
webpage (it is near the top).  Then click on the "New workflow" button on
the left hand side.  You will be presented with a plethora of "starter"
workflows for different purposes.  Search for "manual" in the search box and
you should see a single workflow named "Manual workflow" by GitHub Actions
in the search results.  Click on the "Configure" button on the workflow.
Then click on the "Start commit" button once you are done reviewing the
workflow and then commit the file.  Note that this creates a workflow YAML
file ".github/workflows/manual.yml" under your repository.

Please refer to the following tutorial to see exactly where to click:
https://docs.github.com/en/actions/using-workflows/using-starter-workflows

Now let's take a close look at the manual.yml YAML file.  At below are the
file contents:

```
# This is a basic workflow that is manually triggered

name: Manual workflow

# Controls when the action will run. Workflow runs when manually triggered using the UI or API.
on:
  workflow_dispatch:
    # Inputs the workflow accepts.
    inputs:
      name:
        # Friendly description to be shown in the UI instead of 'name'
        description: 'Person to greet'
        # Default value if no value is explicitly provided
        default: 'World'
        # Input has to be provided for the workflow to run
        required: true

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "greet"
  greet:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Runs a single command using the runners shell
    - name: Send greeting
      run: echo "Hello ${{ github.event.inputs.name }}"
```

Please read the comments starting with # carefully.  In essence, a workflow
is composed of one or more jobs.  Jobs are run on individual Runners (in
parallel by default).  A job is composed of one or more sequential steps
which are performed on the same Runner.  The "Manual workflow" is composed
of a single job named "greet" which is in turn composed of a single step
"Send greeting".

Now let's put this workflow into action by executing it on a Runner!  click
on the "Actions" menu again and you should see a new workflow named "Manual
workflow" on your lefthand side.  Click on it.  Then click on the "Run
workflow" button.  You will get a pop up with an option to change "Person to
greet".  Leave everything as=is and click on the green "Run workflow" button
again.  After a couple of seconds, you will see a new "Manual workflow" run
appear on the list of runs with an orange dot and the orange dot will soon
turn into a green checkmark.  The orange dot indicates that the workflow is
under execution and the green checkmark indicates that it completed
successfully.  A failure is indicated by a red crossmark (which we will
encounter later).  Now click on the "Manual workflow" run link to see the
details of the run.  You should see a screen that looks like this:

<img alt="Manual workflow run" src=img/manual_workflow_1.png>

The screen shows an overview of the workflow.  The workflow is composed of a
single "greet" job as configured in the YAML file.  The green checkmark
beside the job indicates success.  Now click on the "greet" job to peek into
the job and you should see the below screen:

<img alt="Manual workflow greet job" src=img/manual_workflow_2.png>

As you see, the job is composed of 3 steps: "Set up job", "Send greeting",
and "Complete job".  The steps "Setup job" and "Complete job" are implicitly
inserted into every job even though they are not specified in the YAML file.
I've expanded the first two steps for viewing.  The purpose of "Set up job"
is to set up the Runner Docker container within which the job is to run.
You can see that the container was created using the ubuntu-20.04 Docker
image, and that is because "run-on: ubuntu-latest" was specified on the YAML
file and 20.04 happened to tbe latest version.  The "Send greeting" step
runs the commandline specified in the "run:" entry in the YAML file.

The reason that we were able to trigger this workflow manually was because
of the following lines on the YAML file:

```
on:
  workflow_dispatch:
```

Typically, workflows are triggered automatically in response to some
repository event but it is useful to be able to sometimes trigger them
manually.

Again, refer to the below tutorial if you are confused on where to click:
https://docs.github.com/en/actions/quickstart

### Add Maven CI Workflow

Now let's get down to business and try writing a CI workflow for our Maven
project.  This time we are going to have two jobs: 1) A "maven_test" job for
invoking "mvn test" on our project and 2) A "update_dependence_graph" job
for updating the package dependence graph in your GitHub repository.  The
dependence graph is used by a GitHub bot called Dependabot to notify the
repository owner of packages used by the repository that have become stale
or have outstanding security vulnerabilities.

Click on the "Actions" menu again and then click on the "New workflow"
button.  Now, instead of choosing a pre-existing "starter" workflow like
before, click on the "set up a workflow yourself" link.  This will take you
to a page for editing ".github/workflows/main.yml" from a blank slate.
Please change the file name to "maven-ci.yml" instead and then paste the
following into the content box before commiting the file:

```
name: Maven CI

# Triggers manually or on push or pull request on the main branch
# (in other words, on a code integration event.)
on:
  workflow_dispatch:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  
  # Runs the Maven test phase on the project
  maven_test:

    runs-on: ubuntu-latest

    permissions:
      contents: read

    steps:

    # The uses: keyword invokes a GitHub action:
    # https://github.com/marketplace/actions/checkout
    - name: Checkout repository
      uses: actions/checkout@v3

    # This invokes the Setup Java JDK GitHub action:
    # https://github.com/marketplace/actions/setup-java-jdk
    - name: Set up JDK 8
      uses: actions/setup-java@v3
      with:
        java-version: '8'
        distribution: 'temurin'
        cache: maven

    - name: Test with Maven
      run: mvn test

    # https://github.com/marketplace/actions/upload-a-build-artifact
    - name: Upload jacoco results as artifact
      uses: actions/upload-artifact@v2
      with:
        name: Jacoco coverage results
        path: target/site/jacoco

  # Uploads dependency graph to GitHub to receive Dependabot alerts 
  update_dependence_graph:

    runs-on: ubuntu-latest

    permissions:
      contents: read

    steps:

    # https://github.com/marketplace/actions/maven-dependency-tree-dependency-submission
    - name: Update dependency graph
      uses: advanced-security/maven-dependency-submission-action@v2

```

# Submission

Please complete Part 1 and Part 2 questions in
[ReportTemplate.docx](ReportTemplate.docx).  These is a [PDF
version](ReportTemplate.pdf) as well.  

Wnen you are done, submit to the "Supplementary Exercise 4 Report" link on
GradeScope.  I want each member in the group to have gone through the
exercise on his/her/their own before submitting.

# Groupwork Plan

I expect each group member to experience CI/CD pipelines.  I created
individual repositories for each of you, so please work on your own
repositories to implement the pipelines.  After both of you are done,
compare the YAML files that each of you wrote.  Discuss, resolve any
differences, and submit the GitHub repository of your choice.

# Resources
