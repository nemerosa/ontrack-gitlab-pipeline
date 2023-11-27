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

### Setup of validations and promotion levels

If you need to setup auto promotions (which I highly recommend), you can ask your pipeline to set them up:

```yaml
setup:
  stage: .pre
  extends:
    - ...
    - .ontrack-promotions
    - ...
```

The `.ontrack-promotions` expects a `.ontrack/promotions.yaml` file to be present with a content like:

```yaml
validations:
  - name: unit-tests
    description: Unit tests
    tests:
      warningIfSkipped: false
promotions:
  - name: BRONZE
    validations:
      - unit-tests
      - lint
  - name: SILVER
    promotions:
      - BRONZE
    validations:
      - deploy
```

> Using this file, you can define _typed validatons_ and relationships between promotions, validations & other promotions.

### JUnit validations

To create a validation based on the JUnit XML result files:

```yaml
build-job:
  stage: build
  script:
    - ./gradlew build --stacktrace --parallel --console plain
  extends:
    - .ontrack-validate-tests
  variables:
    ONTRACK_VALIDATION_TESTS: build/test-results/test/*.xml
```

Use the `.ontrack-validate-tests` job and the `ONTRACK_VALIDATION_TESTS` to specify a pattern to look for the JUnit XML results.

> Important: your validation must be setup for unit tests (see example (above)[#setup-of-validations-and-promotion-levels])

## References

* https://docs.gitlab.com/ee/ci/variables/predefined_variables.html - GitLab environment variables
