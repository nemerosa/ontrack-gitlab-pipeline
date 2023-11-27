# Ontrack Gitlab Pipeline

This project defines reusable GitLab pipeline tasks which can used by actual jobs in your pipelines.

## Quick start

In your GitLab group or project, define the following variables:

* `ONTRACK_URL` - the URL of the Ontrack instance.
* `ONTRACK_TOKEN` - an API token to connect to the Ontrack instance. The `AUTOMATION` global permission is recommended (instead of administrator).

In your own `.gitlab-ci.yml`, start by importing the Ontrack GitLab pipeline template:

```yaml
include:
  - remote: 'https://gitlab.com/nemerosa/ontrack-gitlab-pipeline/-/raw/main/templates/.ontrack-gitlab-template.yml'
```

Setup the Ontrack project & branch by running:

```yaml
setup:
  stage: .pre
  extends: .ontrack-setup
  script: |
    echo "Setting up Ontrack"
```

> This will use the CI predefined variables to setup your Ontrack project & branch.

As soon as your pipeline knows the _version_ which is built, you can declare the Ontrack build:

```yaml
<job>:
  extends: .ontrack-build
  script: |
    echo "1.0.${CI_PIPELINE_IID}" > version.txt
  variables:
    ONTRACK_BUILD_VERSION_FILE: version.txt
```

Just extend the `.ontrack-build` job and set the `ONTRACK_BUILD_VERSION_FILE` variable to point to a file which contains the _version_.

Note that the setup and the creation of the build can be combined together:

```yaml
setup:
  stage: .pre
  extends:
    - .ontrack-setup
    - .ontrack-build
  script: |
    echo "Setting up Ontrack"
    echo "1.0.${CI_PIPELINE_IID}" > version.txt
  variables:
    ONTRACK_BUILD_VERSION_FILE: version.txt
```

Then, register an Ontrack validation task for every job in your pipeline by setting up:

```yaml
<job>:
  extends: .ontrack-validate
  variables:
    ONTRACK_VALIDATION: lint
```

This validation is either PASSED or FAILED, depending on the result of the job. More use cases (see below) are possible.

## Use cases

## References

* https://docs.gitlab.com/ee/ci/variables/predefined_variables.html - GitLab environment variables
