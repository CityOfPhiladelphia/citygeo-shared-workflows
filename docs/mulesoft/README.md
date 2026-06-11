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

## Advanced Features

### Passing Secrets into pom.xml

Your mule app may need a secret provided to it, such as an API key, database password, etc.

1. Add the secret to Github Actions secrets
   1. Navigate to Settings --> Secrets and variables -> Actions
   1. Select "New repository secret"
   1. For name, pick a name of your choosing
   1. Insert the secret value
1. Insert the name of the secret (from step 1.3) in the `secretsToEnv` workflow variable
   1. Navigate to the workflow file in your repo (.github/workflows/deploy.yaml)
   1. Underneath every deploy job, add an input called `secretsToEnv:`, and set its value to a comma separated list of all the secret names (from step 1.3). See example below
1. Insert each secret name into the pom.xml as environment variables within the `<secureProperties>` section
   1. Navigate to pom.xml
   1. Add the secret names. See example below

#### Example: You added a secret named "PHILA_API_KEY" and one named "PHILA_DB_PASSWORD"

`deploy.yaml` snippet

```yaml
# Dev only showed for brevity, but add the input to prod and test as well
deploy-dev:
  name: Deploy to Dev
  needs: deploy-exchange
  if: ${{ (github.event_name == 'push' && github.ref == 'refs/heads/main') || (github.event_name == 'workflow_dispatch' && inputs.environment == 'dev') }}
  uses: CityOfPhiladelphia/citygeo-shared-workflows/.github/workflows/mulesoft_deploy_rtf.yaml@main
  secrets: inherit
  with:
    MS_ENV: DEV
    secretsToEnv: "PHILA_API_KEY, PHILA_DB_PASSWORD"
```

`pom.xml` snippet

```xml
<runtimeFabricDeployment>
  <secureProperties>
    <PHILA_API_KEY>${env.PHILA_API_KEY}</PHILA_API_KEY>
    <PHILA_DB_PASSWORD>${env.PHILA_DB_PASSWORD}</PHILA_DB_PASSWORD>
  </secureProperties>
</runtimeFabricDeployment>
```

### Additional Environment Specific Parameters in pom.xml

If your pom.xml is set up properly from earlier, a lot of environment (dev, test, prod) specific configuration is handled automatically. However, you may have specific configuration that needs to be different for each environment.

1. Insert any custom non-secret variables in the `extraEnv` workflow variable as a JSON
   1. Navigate to the workflow file in your repo (.github/workflows/deploy.yaml)
   1. Underneath every deploy job, add an input called `extraEnv:`, and set its value to a JSON of all the variables and their values. See example below
1. Insert each variable into the pom.xml as environment variables within the `<properties>` section
   1. Navigate to pom.xml
   1. Add the variables. See example below

#### Example: Dev and Test environment reach out to a service at dev-api.phila.gov, but prod reaches out to prod-api.phila.gov

`deploy.yaml` snippet

```yaml
deploy-dev:
  name: Deploy to Dev
  needs: deploy-exchange
  if: ${{ (github.event_name == 'push' && github.ref == 'refs/heads/main') || (github.event_name == 'workflow_dispatch' && inputs.environment == 'dev') }}
  uses: CityOfPhiladelphia/citygeo-shared-workflows/.github/workflows/mulesoft_deploy_rtf.yaml@main
  secrets: inherit
  with:
    MS_ENV: DEV
    extraEnv: '{"PHILA_API_URL": "https://dev-api.phila.gov"}'

deploy-test:
  name: Deploy to Test
  needs: deploy-dev
  if: ${{ (github.event_name == 'push' && github.ref == 'refs/heads/main') }}
  uses: CityOfPhiladelphia/citygeo-shared-workflows/.github/workflows/mulesoft_deploy_rtf.yaml@main
  secrets: inherit
  with:
    MS_ENV: TEST
    extraEnv: '{"PHILA_API_URL": "https://dev-api.phila.gov"}'

deploy-prod:
  name: Deploy to Prod
  needs: deploy-test
  if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
  uses: CityOfPhiladelphia/citygeo-shared-workflows/.github/workflows/mulesoft_deploy_rtf.yaml@main
  secrets: inherit
  with:
    MS_ENV: PROD
    extraEnv: '{"PHILA_API_URL": "https://prod-api.phila.gov"}'
```

`pom.xml` snippet

```xml
<runtimeFabricDeployment>
  <properties>
    <phila-api-url>${env.PHILA_API_URL}</custom-value>
  </properties>
</runtimeFabricDeployment>
```
