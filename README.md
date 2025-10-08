# citygeo-shared-workflows

This projects combines some shared Github actions workflows for CityGeo

## Purpose

The purpose of this project is to avoid having to update every project individually in the event that one of the common actions needs to be updated, and to ensure consistency

## Workflows

### Linting / Formatters

#### [shellcheck](.github/workflows/shellcheck.yaml)

Runs [shellcheck](https://github.com/koalaman/shellcheck) on a file or set of files

#### [terraform_fmt](.github/workflows/terraform_fmt.yaml)

Checks that terraform file are formatted according to `terraform fmt` standards

#### [tflint](.github/workflows/tflint.yaml)

Runs [tflint](https://github.com/terraform-linters/tflint) on terraform files

### Testing

#### [terraform_plan](.github/workflows/terraform_plan.yaml)

Executes `terraform plan` and comments on the pull request with the output

### Maintenance

#### [tf_ami_updater.yaml](.github/workflows/tf_ami_updater.yaml)

Scheduled job that checks for new AWS AMIs and creates a pull request with them

### Deployment

#### [terraform_apply](.github/workflows/terraform_apply.yaml)

Executes `terraform apply`
