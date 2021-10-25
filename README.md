# Bioinformatics Template

This is a template Github repository that is preconfigured with extreme bias. Please contact the BI-SRE team to reconfigure this repository for your project or to offer suggestions to what the default configuration should be.


## Master Branch protection

This repository is configured to require pull requests to reviewed by a **CODEOWNER** before merging into the Master branch. Configure this by adding users to the CODEOWNER file in the following format: `@{gh-username}`.

## Mergeable enabled

Mergeable integration is active on this repository. The default configuration is located at `.github/mergeable.yml` and includes the following default behavior:

 - Remove WIP from the title. Recommend using draft PRs instead.
 - Label this PR as a bug, enhancement, feature, or chore.
 - Description should not be empty. Provide detail with **what** was changed, **why** it was changed, and **how** it was changed.
 - Include ticket in title in square brackets (ex: [AAA-123])
 - Assign this PR to at least one person.
 - Isn't 5 more than enough for number of people assigned to review this PR?
 - Change is very huge. Let's agree that you and I are human and we can't really comprehend more than 500 lines of changes.

## Jenkins enabled

Jenkinsfile is included in this repo and the default jenkins webhook is configured. A corresponding Jenkins pipeline will have to be configured in Jenkins to complete the integration. 

The default Jenkinsfile will most likely fail in a valid Jenkins pipeline. Configure this file to suit the needs of your project.

## Dependabot enabled

Dependabot configuration is located at `.github/dependabot.yml` and is configured to check for newer pip packages daily.

## .gitignore included

The default .gitignore is suited for python development.
