# Github Certification Exam Notes

## Actions Overview

- 4 sections in exam:
  1. Author and maintain workflows
  2. Consume workflows
  3. Author and maintain actions
  4. Manage github actions in the enterprise

### Great resources

- Walking through this learn collection should teach about 90% of everything that is needed: <https://docs.microsoft.com/users/githubtraining/collections/n5p4a5z7keznp5>
  - If last minute, I would focus mostly on the modules of:
    - Automate development tasks by using GitHub Actions
    - Build continuous integration (CI) workflows by using GitHub Actions
    - Create and publish custom GitHub actions
    - Manage GitHub Actions in the enterprise

### Two types of actions

- Container actions:
  - With container actions, the environment is part of the action's code. These actions can only be run in a Linux environment that GitHub hosts. Container actions support many different languages.
- JavaScript actions:
  - JavaScript actions don't include the environment in the code. You'll have to specify the environment to execute these actions. You can run in a VM in the cloud or on-premises. JavaScript actions support Linux, macOS, and Windows environments.
- Composite actions (technically a third but not included in this list of need to know):
  - Less used.
  - Composite run steps actions enable you to reuse actions using shell scripts. You can even mix multiple shell languages within the same action. If you have a lot of shell scripts to automate several tasks, now you can easily turn them into an action and reuse them for different workflows. Sometimes it's easier to just write a shell script than using JavaScript or wrapping your code in a Docker container.

### Actions Flow

<img width="517" alt="image" src="https://user-images.githubusercontent.com/22186029/175856662-74c2d884-6da0-425a-8e82-20f6d14b2985.png">

- Workflows
  - A workflow is an automated process that you add to your repository. A workflow needs to have at least one job and can be triggered by different events. You can use it to build, test, package, release, or deploy your repository's project on GitHub.
  - Workflows can be run by either a specific activity that occurs on GitHub, when an event outside of GitHub happens, or at a scheduled time.
- Jobs
  - The job is the first major component within the workflow. A job is a section of the workflow that will be associated with a runner. A runner can be GitHub-hosted or self-hosted, and the job can run on a machine or in a container. You'll specify the runner with the runs-on: attribute.
- Steps
  - A step is an individual task that can run commands in a job. In our example above, the step uses the action actions/checkout@v2 to check out the repository. What's interesting is the using: ./action-a value. This is the path to the container action that you'll build in an action.yml file. A step is a set of tasks performed by a job. Steps can run commands or actions.
- Actions
  - The actions inside your workflow are the standalone commands that are executed. These standalone commands can reference GitHub actions such as using your own custom actions, or community actions like the one we use above, actions/checkout@v2. You can also run commands such as `run: npm install -g bats` to execute a command on the runner.

## Referencing and running actions

### How to reference an action

- You can either reference an action by it's commit hash, its version number or its branch

```yml
steps:    
  # Reference a specific commit
  - uses: actions/setup-node@c46424eee26de4078d34105d3de3cc4992202b1e
  # Reference the major version of a release
  - uses: actions/setup-node@v1
  # Reference a minor version of a release
  - uses: actions/setup-node@v1.2
  # Reference a branch
  - uses: actions/setup-node@main
```

### Environment Variables

To use a default environment variable, specify $ followed by the environment variable's name.
You can use defined variables as contexts. The $ tells GitHub to evaluate the statement as an expression instead of just a string.
Notice default environment variables are all uppercase where context variables are all lowercase.
You can also use custom environment variables in your workflow file. Also Default variables are case sensitive.

```yml
# This example is using the github.ref context to check the branch that triggered the workflow. If the branch is main, the runner is executed and prints out "Deploying to production server on branch $GITHUB_REF". The default environment variable $GITHUB_REF is used in the runner to refer to the branch. To create a custom variable (ex: First_Name), you need to define it in your workflow file using the env context.
name: CI
on: push
jobs:
  prod-check:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Nice work, $First_Name. Deploying to production server on branch $GITHUB_REF"
        env:
          First_Name: Mona
```

### Passing artifact data between jobs

Using "upload-artifact" and "download-artifact" works well.

```yml
name: Share data between jobs
on: push
jobs:
  job_1:
    name: Upload File
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello World" > file.txt
      - uses: actions/upload-artifact@v2
        with:
          name: file
          path: file.txt

  job_2:
    name: Download File
    runs-on: ubuntu-latest
    needs: job_1
    steps:
      - uses: actions/download-artifact@v2
        with:
          name: file
      - run: cat file.txt
```

### Workflow Triggers (Events to run an action)

Workflows can be configured to run:

- Using events from GitHub
- At a scheduled time
  - Actions can be configured to run on a schedule. Configure it by using: *****. Each star stands for a different component of time. Minute/Hour/Day of the month/month/Day of the week.
- When an event outside of GitHub occurs
  - You can trigger a webhook event called repository_dispatch. You will trigger the workflow by making a POST request to the GitHub endpoint: /repos/{owner}/{repo}/dispatches with the event names in the request body.
- Manually
  - You can manually trigger a workflow by using the workflow_dispatch command.

## Creating Actions

Metadata and syntax needed to create an action in a action.yml file:

Parameter | Description | Required
----------|--------------|-------
Name | The name of your action. Helps visually identify the action in a job. | yes
Description |A summary of what your action does. |yes
Inputs |Input parameters enable you to specify data that the action expects to use during runtime. These parameters become environment variables in the runner. |no
Outputs |Output parameters enable you to specify data that subsequent actions can use later in the workflow after the action that defines these outputs has run. |no
Runs |The command to run when the action executes. |yes
Branding |Color and Feather icon to use to create a badge to personalize and distinguish your action in GitHub Marketplace. |no

## Manage github actions in the enterprise

Learning Module: <https://docs.microsoft.com/learn/modules/manage-github-actions-enterprise>

- To control the sync of public actions in the enterprise,  Configure a GitHub Actions use policy to configure which actions are available.
- Users with write access to an organization's .github repository can create workflow templates that will be available for use to the other organization's members with the same write access.
  - Two Files are needed
    - a yml workflow file
    - a json metadata file to describe how the template should be presented to users when they are creating a workflow.
    - Note: The metadata file must have the same name as the workflow file. Instead of the .yml extension, it must be appended with .properties.json. For example, a file named octo-organization-ci.properties.json contains the metadata for the workflow file named octo-organization-ci.yml.

### Self-Hosted vs GitHub-hosted runners

- Self hosted runners are advantageous because they have additional customization.
  - The most common customization to the self-hosted runners is:
    - Labels
      - Has 3 default labels:
        - Default label:
          - self-hosted: Default label applied to all self-hosted runners
          - linux, windows, or macOS: Applied depending on the runner's operating system.
          - x64 , ARM, or ARM64: Applied depending on the runner's hardware architecture.
    - Proxy servers
    - IP allowlists
