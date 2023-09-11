# Ontrack Gitlab Pipeline

This project defines reusable GitLab pipeline tasks which can used by actual jobs in your pipelines.

## Quick start

In your GitLab group or project, define the following variables:

* `ONTRACK_URL` - the URL of the Ontrack instance.
* `ONTRACK_TOKEN` - an API token to connect to the Ontrack instance. The `AUTOMATION` global permission is recommended (instead of administrator).

In your own `.gitlab-ci.yml`, start by importing the Ontrack GitLab pipeline template:

```yaml
include:
  - project: 'nemerosa/ontrack-gitlab-pipeline'
    file: '/templates/.ontrack-gitlab-template.yml'
```

Setup the Ontrack project & branch by running:

```yaml
setup:
  stage: .pre
  extends: .ontrack_setup
  script: |
    echo "Setting up Ontrack"
```

> This will use the CI predefined variables to setup your Ontrack project & branch.

Then, register an Ontrack validation task for every job in your pipeline by setting up:

```yaml
default:
    extends: .ontrack_job
```

> Other options are possible at job level for some specific cases.
