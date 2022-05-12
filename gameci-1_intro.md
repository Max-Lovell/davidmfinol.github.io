# GameCI 1: Intro to GitHub Actions for Unity

As a software engineer, I believe every modern software project should have a CI/CD pipeline.
The reasons for such are many and better explained [all](https://www.synopsys.com/glossary/what-is-cicd.html)
 [over](https://www.redhat.com/en/topics/devops/what-is-ci-cd) [the](https://www.infoworld.com/article/3271126/what-is-cicd-continuous-integration-and-continuous-delivery-explained.html)
 [internet](https://en.wikipedia.org/wiki/CI/CD), so I won't be making a case for why you should be using CI/CD.
Instead, I write this series of articles with the goal of helping Game Developers with their own pipelines.
In particular, I belive that [Unity](https://unity.com/) game projects don't have as many resources for CI/CD as they should.
Hopefully, this guide to how I built the CI/CD pipeline for my Unity project will help you with yours.

## My Workflow
A picture is worth a thousand words, so take a look at the visualization graph for my workflow:
![Test, Build, and Deploy with GameCI](assets/img/cgs-workflow.png)

Nowadays, developers have a lot of CI options (Unity Cloud Build, CircleCI, GitLab CI, and Jenkins are just a few examples).
I chose [GitHub Actions](https://github.com/features/actions) because it is tightly integrated to the GitHub repo where I already keep my open-source project,
 and GitHub provides free CI minutes for open-source projects like mine.
Plus, it makes this nice visualization graph.

## Pieces of the Workflow
You may have noticed that there are a lot of different jobs in the workflow, but only some of them ran, while others did not run.
Later, I'll explain all the different jobs and what they do/how they work, but the first thing to examine is why some jobs run while others do not.
If you look at the names of the jobs that didn't run, you may be able to guess the reason: The jobs that deploy to production (the CD part of CI/CD) were not part of this workflow run.

Depending on how you think of it, it could be argued that my workflow is actually multiple workflows in one.
For every push to my develop branch, I run the test and build jobs, to confirm that that commit didn't break anything.
When I want to actually deploy to production, I [create a release in GitHub](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository#creating-a-release), which will trigger the workflow to run with all the production deployment jobs.
I can also [manually run the workflow](https://docs.github.com/en/actions/managing-workflow-runs/manually-running-a-workflow), with the option to input which jobs should run.

The modularity of my workflow is enabled by setting up different triggers and blocking certain jobs with `if`s.
You may refer to the [complete workflow on GitHub](https://github.com/finol-digital/Card-Game-Simulator/blob/develop/.github/workflows/main.yml), but going forward, I will show the relevant excerpts here.
For example, here's the basic setup:
```yml
# .github/workflows/main.yml
name: Test, Build, and Deploy with GameCI
on:
  push:
    branches:
      - develop
    paths:
      - 'Assets/**'
      - 'Packages/**'
      - 'ProjectSettings/**'
  pull_request:
    types:
      - opened
    branches:
      - main
    paths:
      - 'Assets/**'
      - 'Packages/**'
      - 'ProjectSettings/**'
  release:
    types:
      - published
  workflow_dispatch:
    inputs:
      workflow_mode:
        description: '[release] [Android, iOS, StandaloneLinux64, WebGL, StandaloneWindows, StandaloneWindows64, WSAPlayer, StandaloneOSX, Steam]'
        required: false
        default: ''
jobs:
```

## Trigger On
As you can see in the above code, there are 4 triggers on: `push` to develop, `pull_request` to main, `release` published (with GitHub Release), and `workflow_dispatch`.

The `push` and `pull_request` triggers should run checks to validate each commit, along with a final validation before merging changes to `main`.
Runs on `push` and `pull_request` should only trigger if a file has changed in the actual Unity project, ie in `Assets/`, `Packages/`, or `ProjectSettings/`.
Any changes outside of these folders would not cause our Unity project to change, so they can be ignored.
For example, if we modify the README.md for our project, we wouldn't want the workflow to run, since it wouldn't actually be testing any relevant change to the Unity project.

Runs on `release` will trigger whenever a release is created in the GitHub UI, and we can use `if: github.event.action == 'published'` to detect this scenario.

The triggers for `workflow_dispatch` are the most interesting, since that's where we have setup the most control over the workflow.
For example, if we wanted to create 2 Windows .exe's to download and run, we can simply pass `StandaloneWindows StandaloneWindows64` as the input.
This would create both 32-bit and 64-bit Windows executables and upload them to GitHub, where they could then be downloaded for testing.

Taking it a step further, we could input `release Steam` to run the entire production deployment pipeline for Steam.
My Steam depots involve 4 artifacts: 1) Windows 32-bit, 2) Windows 64-bit, 3) Linux, and 4) Mac.
Building and deploying these artifacts is split across multiple jobs, so it would now be good to get a high-level overview of all the jobs.

## The Jobs
Going into detail on each job would make this article far too long.
Furthermore, a reader who only wants to publish Android builds on Google Play would be interested in a different set of jobs than a reader who wants to publish PC games on Steam.
Therefore, here is a quick overview of each job, with links to more info, if more info is desired.
You may read these overviews and then pick and choose to read only that which is relevant to you.

### Test Code Quality
I consider this job to be the most fundamental in any CI pipeline, as it is the one responsible for actually running your [unit tests](https://docs.unity3d.com/Manual/testing-editortestsrunner.html).
Furthermore, I also set up SonarQube quality checks and code coverage.
See [GameCI 2: Testing](gameci-2_testing.html).

### Build with Linux

### Build with Mac

### Deploy to the App Store

### Deploy to the Google Play Store

### Deploy to the Web via GitHub Pages

### Deploy to the Mac App Store

### Build with Windows

### Deploy to the Microsoft Store

### Deploy to the Steam Marketplace

### Announce Release to Social Media

## Continue
If you have decided that you would not like to read about all the jobs in order, I'd recommend continuing with [GameCI 2: Testing](gameci-2_testing.html)