-   <a href="#introduction" id="toc-introduction">1 Introduction</a>
-   <a href="#the-environment---docker" id="toc-the-environment---docker">2
    The environment - Docker</a>
-   <a href="#the-platform---github-actions"
    id="toc-the-platform---github-actions">3 The platform - Github
    Actions</a>

# 1 Introduction

Workflow automation is key to making a forecast that is run every day,
takes in new observations, updates parameters and intiial conditions and
produces a new forecast. And the automation means all of this is done
without you needing to click a button every day.

# 2 The environment - Docker

To automate your forecast, the workflow needs to be fully reproducible.
The environment/set up, packages, file paths need to be set up in a way
that can be reproduced every day in the same way. As part of this
reprodcibility we will use a Docker container:

> A container is a standard unit of software that packages up code and
> all its dependencies so the application runs quickly and reliably from
> one computing environment to another. Containers isolate software from
> its environment and ensure that it works uniformly despite differences
> for instance between development and staging.

We will utilise a container from the `RockerProject` a Docker container
that has R installed as well as some pre-installed packages. The NEON
forecast Challenge has a container available which has the neon4cast
package (plus tidyverse and other commonly used packages) already
installed.

# 3 The platform - Github Actions

There are a few ways that the running of a script can be automated but
we will be using the Github Actions tools. Actions allow you to run a
workflow based on a trigger, which in our case will be a time (but could
be when you push or pull to a directory). Read more about [Github
Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions).
To start off with Github actions you need a worflow yaml file. Yaml
files are computer readable ‘instructions’ that essentially say what the
Action needs to do. Every time an action is triggered to start it will
open a Linux machine environment and from this we can give it a series
of instructions to get to our forecast submission. Below is an example
of what your yaml file might look like to run an automated forecast.

A basic description of a Github action

> You can configure a GitHub Actions *workflow* to be triggered when an
> event occurs in your repository, such as a pull request being opened
> or a timer. Your workflow contains one or more *jobs* which can run in
> sequential order or in parallel. Each job will run inside its own
> virtual machine or container, and has one or more *steps* that either
> run a script that you define or run an action.

-   `on` tells you what triggers the workflow - here we use a `schedule`
    to determine the action based on a `cron` schedule (i.e. a timer) to
    run a 12 (UTC), everyday. You can update this to run on a different
    schedule based on timing codes found in <https://crontab.guru>.
-   `jobs` this is what you are telling the machine to do. You can see
    that within the job we have other instructions that tell the machine
    what `container` to use and the various `steps` in the job.
    -   The first is to `checkout repo` which uses a premade action
        `checkout` to get a copy of the Github repo.
    -   Next, within the container, we run the R script
        `run_forecast.R` - this is your forecast code that generates a
        forecast file and has code to submit the saved forecast to the
        Challenge.

Because of the workflow_dispatch this will run everyday, submitting your
forecast to the Challenge. As long as your run_forecast.R has all the
code in to do this!

    on:
      workflow_dispatch:
      schedule:
      - cron: "0 12 * * *"

    jobs:
      run_forecast:
        runs-on: ubuntu-latest
        container:
          image: eco4cast/rocker-neon4cast
        steps:
          - name: Checkout repo
            uses: actions/checkout@v2
            with:
              fetch-depth: 0
              
          - name: Run automatic prediction file
            run: Rscript run_forecasts.R 

Once all the instructions/steps are run the container will close. When a
container closes all data created (like the forecast_file.csv) will be
lost. If you need to retrieve anything from an automated Github Action
it needs to be pushed back to Github or to a remote location (e.g. cloud
storage).

This workflow file should be saved into a sub-directory called
`.github/workflows` in the parent directory.
