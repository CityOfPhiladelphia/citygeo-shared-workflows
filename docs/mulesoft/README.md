# Mulesoft CI/CD Workflows

## Features

- Uploads code to Exchange whenever the app version changes
- Deploys app first to dev, then on success, deploys to test
- Waits for manual approval before deploying to prod
- Allows for manual deploys to dev environment from dev branches

## Setup

It is recommended to switch to a non-main branch for any changes to the project before merging to main. This allows you to double check the results of the testing pipelines before actually deploying anything, which will happen on merge to main.

### pom.xml

Deployments to Anypoint Platform are orchestrated by Maven, which uses values in `pom.xml`.

Essentially we are combining the Anypoint documentation for [deploying to RTF from Maven](https://docs.mulesoft.com/mule-runtime/latest/deploy-to-rtf) and for [publishing to Exchange from Maven](https://docs.mulesoft.com/exchange/to-publish-assets-maven), plus adding a few extras to make automation easier.

Please refer to the [sample pom.xml](https://github.com/CityOfPhiladelphia/mulesoft-sample-rtf-deploy/blob/main/pom.xml) which has plenty of comments explaining all the parameters you need to add and their values.

Basic overview of what you need to change:

1. Set `groupId` to the business group ID that the app will be deployed to.
1. Set `artifactId` to the desired name of the app in RTF.
1. Make sure `version` is a valid semantic version, like `0.1.2`
1. For the `<plugin>` with `<artifactId>mule-maven-plugin</artifactId>`, copy the entire `<runtimeFabricDeployment>` section from the sample pom.xml and read through the comments to make necessary adjustments.
1. In the `<repositories>` section, add the repository with name `<name>Private Exchange repository</name>` from the sample pom.xml and do not change it.
1. Add the entire `<distributionManagement>` section from the sample pom.xml and do not change it.

### Github Repo

### Github Repo Settings

1. Add OIT-Mulesoft to "Collaborators and teams" with at least write permissions.
1. Add an environment for "dev", "test", and "prod", spelled that way exactly
   1. Leave "dev" and "test" at default values.
   1. For "prod" environment, enable "Required reviewers" and select `CityOfPhiladelphia/oit-mulesoft`
   1. For "prod" environment, change "Deployment branches and tags" to "Selected branches and tags", then add "main" as the branch in the section below.
1. Add Keeper Secret
   1. Navigate to secrets and variables -> Actions
   1. Select "New repository secret"
   1. For name, insert exactly "KSM_CONFIG"
   1. For secret, insert the Keeper key. If you don't have this, ask Ryan or Roland.

### Github Repo Folder Structure

1. At the root of your git project, create a folder called `.github`, then a subfolder called `.github/workflows`
1. Copy these two files into that folder: [deploy.yaml](https://github.com/CityOfPhiladelphia/mulesoft-sample-rtf-deploy/blob/main/.github/workflows/deploy.yaml) and [pr.yaml](https://github.com/CityOfPhiladelphia/mulesoft-sample-rtf-deploy/blob/main/.github/workflows/pr.yaml). Ideally these files would never change once this is stable, but you may be instructed to make adjustments in the future.
1. **Important**: If you are deploying an existing / already deployed app, comment out entirely (using the '#' character) the `deploy-test` and `deploy-prod` jobs in "deploy.yaml", this ensures you don't accidentally overwrite prod or test while first setting up this pipeline. Then, once dev seems to work well, you can uncomment those lines.
