---
title: "Passing Variables Through GitLab Pipelines" # Title of the blog post.
date: 2022-04-29 # Date of post creation.
description: "Passing variable gitlab pipeline" # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Continuous Integration
  - GitLab
tags:
  - automation
  - pipeline
  - gitlab
# comment: false # Disable comment if false.
---

During working with GitLab multi-project pipelines and parent-child pipelines, I have encountered the problem how to pass variables through these pipelines.
The [GitLab documentation](https://docs.gitlab.com/13.12/ee/ci/multi_project_pipelines.html#passing-cicd-variables-to-a-downstream-pipeline) describes very well how to pass variables to a downstream pipeline.
My challenge is how to pass variables from child to parent pipeline and how the parent pipeline can pass these variables to a downstream pipeline, that it describes in another GitLab project.
Let's start, how to publish the variable that are defined in a child pipeline.

## Publishing Variables of a Child Pipeline

Assume that we have a GitLab project with the following structure for the pipelines.

```text
.
├── pipelines
│   └── child-pipeline.yml
└── .gitlab-ci.yml
```

The parent pipeline, defined in `.gitlab-ci.yml`, triggers the child pipeline, that is defined in `pipelines/child-pipeline.yml`.

```yaml
# .gitlab-ci.yml
stages:
  - a

trigger-child-pipeline-job:
  stage: a
  trigger:
    include: pipelines/child-pipeline.yml
    strategy: depend
```

The child pipeline `pipelines/child-pipeline.yml` defines the variables and publishes them via the [report artifact dotenv](https://docs.gitlab.com/ee/ci/yaml/artifacts_reports.html#artifactsreportsdotenv).

```yaml
stages:
  - define-env

define-env-job:
  stage: define-env
  script:
    - echo "MODULE_A_VERSION=1.0.0" >> .env
  artifacts:
    reports:
      dotenv: .env
```

Dotenv is a standardized way to handle environment variables.
Following the dotenv concept, the environment variables are stored in a file that have the following structure.


```shell
# .env
VARIABLE_NAME=variable-value
MODULE_A_VERSION=1.0.0
```

For more information, please visit the [dotenv homepage](https://dotenv.org/).

Let's go to the next step, how to consume this variable in the parent pipeline.

## Consuming Variables From a Child Pipeline in a Parent Pipeline

The first challenge is how the parent pipeline can consume the variable, that is defined in the child pipeline (in our sample, it is the variable `MODULE_A_VERSION`).
The child pipeline publishes its variable via a report artifact.
This artifact can be used by the parent pipeline via the `needs` keyword.
Unfortunately, it is not enough to reference the job name of the child pipeline that creates the report artifact.
You also have to add a reference to the project that contains the parent and the child pipeline.
Now, the parent pipeline can use the variable that is stored in the report artifact.

```yaml
stages:
  - a
  - b

trigger-child-pipeline-job:
  stage: a
  trigger:
    include: pipelines/child-pipeline.yml
    strategy: depend

consume-env-from-child-pipeline-job:
  stage: b
  script:
    - "echo $MODULE_A_VERSION"
  needs:
    - project: sparsick/gitlab-ci-passing-variable-pipeline
      job: define-env-job
      ref: main
      artifacts: true
```

The next challenge is to consume this variable in a downstream pipeline that is defined in another project.

## Consuming Variable From a Child Pipeline in a Downstream Pipeline of the Parent Pipeline

It exists two ways how a downstream pipeline can consume a variable from a child pipeline of its upstream pipeline.

The first way works similarly that I described in the above section. Assume, that we have the following parent pipeline that triggered a child pipeline and a downstream pipeline in another project.

```yaml
# .gitlab-ci.yaml
stages:
  - a
  - b

trigger-child-pipeline-job:
  stage: a
  trigger:
    include: pipelines/child-pipeline.yml
    strategy: depend

trigger-another-pipeline-job:
  stage: b
  trigger: sparsick/gitlab-ci-passing-variable-downstream-pipeline
```

The variable `MODULE_A_VERSION` is defined in the child pipeline like I described in the above section.
The variable can be consumed by the downstream pipeline in the same way as the parent pipeline, that I described in the above section.

```yaml
# .gitlab-ci.yaml of the downstream pipeline
stages:
  - print-env

print-env-from-a-child-pipeline-of-the-upstream-job:
  stage: print-env
  script:
    - echo $MODULE_A_VERSION
  needs:
    - project: sparsick/gitlab-ci-passing-variable-pipeline
      job: define-env-job
      ref: main
      artifacts: true
```

This approach has a big disadvantage.
If the job/variable/project/branch of the upstream pipeline changes its name, the downstream pipeline doesn't recognize this change automatically, and it couldn't work anymore as expected.

A second way solves this disadvantage.
Here, the variable value is passed via a new variable to the downstream pipeline.
Assume, that we have the following parent pipeline that triggered a child pipeline and a downstream pipeline in another project and pass a variable to the downstream pipeline.

```yaml
# .gitlab-ci.yaml
stages:
  - a
  - b

trigger-child-pipeline-job:
  stage: a
  trigger:
    include: pipelines/child-pipeline.yml
    strategy: depend

trigger-another-pipeline-with-var-job:
  stage: b
  variables:
    ARTIFACT_VERSION: "1.0.0"
  trigger: sparsick/gitlab-ci-passing-variable-downstream-pipeline

```
The downstream pipeline can use the `ARTIFACT_VERSION` variable in the common way.


```yaml
# .gitlab-ci.yaml of the downstream pipeline
stages:
  - print-env

print-env-from-upstream-job:
  stage: print-env
  script:
    - echo $ARTIFACT_VERSION
```

Now, I want, that the value of the variable `MODULE_A_VERSION` of the child pipeline is pass to the downstream pipeline.
My first idea was to add with `needs` a dependency like I used it above in the `consume-env-from-child-pipeline-job` job.
But this is invalid because `trigger` and `needs` with a reference to a project can't be used together in the same job.


```yaml
# invalid job definition
trigger-another-pipeline-with-var-job:
  stage: b
  variables:
    ARTIFACT_VERSION: "$MODULE_A_VERSION"
  trigger: sparsick/gitlab-ci-test-downstream
  needs:
    - project: sparsick/gitlab-ci-test
      job: define-env-job
      ref: main
      artifacts: true

```

Therefore, I have to take a detour via a new job that read the variable from the child and create a new dotenv report artifact.

```yaml
stages:
  - a
  - b
  - c

trigger-child-pipeline-job:
  stage: a
  trigger:
    include: pipelines/child-pipeline.yml
    strategy: depend

consume-env-from-child-pipeline-job:
  stage: b
  script:
    - echo "MODULE_A_VERSION=$MODULE_A_VERSION" >> .env
  needs:
    - project: sparsick/gitlab-ci-test
      job: define-env-job
      ref: main
      artifacts: true
  artifacts:
    reports:
      dotenv: .env

trigger-another-pipeline-with-var-job:
  stage: c
  variables:
    ARTIFACT_VERSION: "$MODULE_A_VERSION"
  needs:
    - job: consume-env-from-child-pipeline-job
      artifacts: true
  trigger: sparsick/gitlab-ci-passing-variable-downstream-pipeline
```

You can find the whole example on [GitLab](https://gitlab.com/sparsick/gitlab-ci-passing-variable-pipeline).

When you have another or better approach how to solve this described problem, let me know and please write a comment.


## Links
- [GitLab Documation about passing CI/CD variables to a downstream pipeline](https://docs.gitlab.com/13.12/ee/ci/multi_project_pipelines.html#passing-cicd-variables-to-a-downstream-pipeline)
- [GitLab Documentation about job artifact dotenv](https://docs.gitlab.com/ee/ci/yaml/artifacts_reports.html#artifactsreportsdotenv)
- [Dotenv Homepage](https://dotenv.org/)
- [GitLab Documation about job dependencies via `need`](https://docs.gitlab.com/ee/ci/yaml/#needs)
- [GitLab Repository with the whole example](https://gitlab.com/sparsick/gitlab-ci-passing-variable-pipeline)
