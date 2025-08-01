---
title: Migrating from Bamboo with GitHub Actions Importer
intro: 'Learn how to use {% data variables.product.prodname_actions_importer %} to automate the migration of your Bamboo pipelines to {% data variables.product.prodname_actions %}.'
versions:
  fpt: '*'
  ghec: '*'
  ghes: '*'
type: tutorial
topics:
  - Migration
  - CI
  - CD
shortTitle: Bamboo migration
redirect_from:
  - /actions/migrating-to-github-actions/automated-migrations/migrating-from-bamboo-with-github-actions-importer
  - /actions/migrating-to-github-actions/using-github-actions-importer-to-automate-migrations/migrating-from-bamboo-with-github-actions-importer
  - /actions/how-tos/migrating-to-github-actions/using-github-actions-importer-to-automate-migrations/migrating-from-bamboo-with-github-actions-importer
  - /actions/tutorials/migrating-to-github-actions/using-github-actions-importer-to-automate-migrations/migrating-from-bamboo-with-github-actions-importer
---

## About migrating from Bamboo with GitHub Actions Importer

The instructions below will guide you through configuring your environment to use {% data variables.product.prodname_actions_importer %} to migrate Bamboo pipelines to {% data variables.product.prodname_actions %}.

### Prerequisites

* A Bamboo account or organization with projects and pipelines that you want to convert to {% data variables.product.prodname_actions %} workflows.
* Bamboo version of 7.1.1 or greater.
* Access to create a Bamboo {% data variables.product.pat_generic %} for your account or organization.
{% data reusables.actions.actions-importer-prerequisites %}

### Limitations

There are some limitations when migrating from Bamboo to {% data variables.product.prodname_actions %} with {% data variables.product.prodname_actions_importer %}:

* {% data variables.product.prodname_actions_importer %} relies on the YAML specification generated by the Bamboo Server to perform migrations. When Bamboo does not support exporting something to YAML, the missing information is not migrated.
* Trigger conditions are unsupported. When {% data variables.product.prodname_actions_importer %} encounters a trigger with a condition, the condition is surfaced as a comment and the trigger is transformed without it.
* Bamboo Plans with customized settings for storing artifacts are not transformed. Instead, artifacts are stored and retrieved using the [`upload-artifact`](https://github.com/actions/upload-artifact) and [`download-artifact`](https://github.com/actions/download-artifact) actions.
* Disabled plans must be disabled manually in the GitHub UI. For more information, see [AUTOTITLE](/actions/using-workflows/disabling-and-enabling-a-workflow).
* Disabled jobs are transformed with a `if: false` condition which prevents it from running. You must remove this to re-enable the job.
* Disabled tasks are not transformed because they are not included in the exported plan when using the Bamboo API.
* Bamboo provides options to clean up build workspaces after a build is complete. These are not transformed because it is assumed GitHub-hosted runners or ephemeral self-hosted runners will automatically handle this.
* The hanging build detection options are not transformed because there is no equivalent in {% data variables.product.prodname_actions %}. The closest option is `timeout-minutes` on a job, which can be used to set the maximum number of minutes to let a job run. For more information, see [AUTOTITLE](/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idtimeout-minutes).
* Pattern match labeling is not transformed because there is no equivalent in {% data variables.product.prodname_actions %}.
* All artifacts are transformed into an `actions/upload-artifact`, regardless of whether they are `shared` or not, so they can be downloaded from any job in the workflow.
* Permissions are not transformed because there is no suitable equivalent in {% data variables.product.prodname_actions %}.
* If the Bamboo version is between 7.1.1 and 8.1.1, project and plan variables will not be migrated.

#### Manual tasks

Certain Bamboo constructs must be migrated manually. These include:

* Masked variables
* Artifact expiry settings

## Installing the {% data variables.product.prodname_actions_importer %} CLI extension

{% data reusables.actions.installing-actions-importer %}

## Configuring credentials

The `configure` CLI command is used to set required credentials and options for {% data variables.product.prodname_actions_importer %} when working with Bamboo and {% data variables.product.prodname_dotcom %}.

1. Create a {% data variables.product.prodname_dotcom %} {% data variables.product.pat_v1 %}. For more information, see [AUTOTITLE](/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic).

   Your token must have the `workflow` scope.

   After creating the token, copy it and save it in a safe location for later use.
1. Create a Bamboo {% data variables.product.pat_generic %}. For more information, see [{% data variables.product.pat_generic_title_case_plural %}](https://confluence.atlassian.com/bamboo/personal-access-tokens-976779873.html) in the Bamboo documentation.

   Your token must have the following permissions, depending on which resources will be transformed.

   Resource Type | View | View Configuration | Edit
    |:--- | :---: | :---: | :---:
    | Build Plan | {% octicon "check" aria-label="Required" %} | {% octicon "check" aria-label="Required" %} | {% octicon "check" aria-label="Required" %}
    | Deployment Project | {% octicon "check" aria-label="Required" %} | {% octicon "check" aria-label="Required" %} | {% octicon "x" aria-label="Not required" %}
    | Deployment Environment | {% octicon "check" aria-label="Required" %} |{% octicon "x" aria-label="Not required" %}| {% octicon "x" aria-label="Not required" %}

   After creating the token, copy it and save it in a safe location for later use.
1. In your terminal, run the {% data variables.product.prodname_actions_importer %} `configure` CLI command:

   ```shell
   gh actions-importer configure
   ```

   The `configure` command will prompt you for the following information:

   * For "Which CI providers are you configuring?", use the arrow keys to select `Bamboo`, press <kbd>Space</kbd> to select it, then press <kbd>Enter</kbd>.
   * For "{% data variables.product.pat_generic_caps %} for GitHub", enter the value of the {% data variables.product.pat_v1 %} that you created earlier, and press <kbd>Enter</kbd>.
   * For "Base url of the GitHub instance", {% ifversion ghes %}enter the URL for {% data variables.location.product_location_enterprise %}, and press <kbd>Enter</kbd>.{% else %}press <kbd>Enter</kbd> to accept the default value (`https://github.com`).{% endif %}
   * For "{% data variables.product.pat_generic_caps %} for Bamboo", enter the value for the Bamboo {% data variables.product.pat_generic %} that you created earlier, and press <kbd>Enter</kbd>.
   * For "Base url of the Bamboo instance", enter the URL for your Bamboo Server or Bamboo Data Center instance, and press <kbd>Enter</kbd>.

   An example of the `configure` command is shown below:

   ```shell
   $ gh actions-importer configure
   ✔ Which CI providers are you configuring?: Bamboo
   Enter the following values (leave empty to omit):
   ✔ {% data variables.product.pat_generic_caps %} for GitHub: ***************
   ✔ Base url of the GitHub instance: https://github.com
   ✔ {% data variables.product.pat_generic_caps %} for Bamboo: ********************
   ✔ Base url of the Bamboo instance: https://bamboo.example.com
   Environment variables successfully updated.
   ```

1. In your terminal, run the {% data variables.product.prodname_actions_importer %} `update` CLI command to connect to {% data variables.product.prodname_registry %} {% data variables.product.prodname_container_registry %} and ensure that the container image is updated to the latest version:

   ```shell
   gh actions-importer update
   ```

   The output of the command should be similar to below:

   ```shell
   Updating ghcr.io/actions-importer/cli:latest...
   ghcr.io/actions-importer/cli:latest up-to-date
   ```

## Perform an audit of Bamboo

You can use the `audit` command to get a high-level view of all projects in a Bamboo organization.

The `audit` command performs the following steps:

1. Fetches all of the projects defined in a Bamboo organization.
1. Converts each pipeline to its equivalent {% data variables.product.prodname_actions %} workflow.
1. Generates a report that summarizes how complete and complex of a migration is possible with {% data variables.product.prodname_actions_importer %}.

### Running the audit command

To perform an audit of a Bamboo instance, run the following command in your terminal:

```shell
gh actions-importer audit bamboo --output-dir tmp/audit
```

### Inspecting the audit results

{% data reusables.actions.gai-inspect-audit %}

## Forecasting usage

You can use the `forecast` command to forecast potential {% data variables.product.prodname_actions %} usage by computing metrics from completed pipeline runs in your Bamboo instance.

### Running the forecast command

To perform a forecast of potential {% data variables.product.prodname_actions %} usage, run the following command in your terminal. By default, {% data variables.product.prodname_actions_importer %} includes the previous seven days in the forecast report.

```shell
gh actions-importer forecast bamboo --output-dir tmp/forecast_reports
```

### Forecasting a project

To limit the forecast to the plans and deployments environments associated with a project, you can use the `--project` option, where the value is set to a build project key.

For example:

```shell
gh actions-importer forecast bamboo --project PAN --output-dir tmp/forecast_reports
```

### Inspecting the forecast report

The `forecast_report.md` file in the specified output directory contains the results of the forecast.

Listed below are some key terms that can appear in the forecast report:

* The **job count** is the total number of completed jobs.
* The **pipeline count** is the number of unique pipelines used.
* **Execution time** describes the amount of time a runner spent on a job. This metric can be used to help plan for the cost of {% data variables.product.prodname_dotcom %}-hosted runners.
  * This metric is correlated to how much you should expect to spend in {% data variables.product.prodname_actions %}. This will vary depending on the hardware used for these minutes. You can use the [{% data variables.product.prodname_actions %} pricing calculator](https://github.com/pricing/calculator) to estimate the costs.
* **Queue time** metrics describe the amount of time a job spent waiting for a runner to be available to execute it.
* **Concurrent jobs** metrics describe the amount of jobs running at any given time. This metric can be used to

## Perform a dry-run migration of a Bamboo pipeline

You can use the `dry-run` command to convert a Bamboo pipeline to an equivalent {% data variables.product.prodname_actions %} workflow. A dry-run creates the output files in a specified directory, but does not open a pull request to migrate the pipeline.

### Running a dry-run migration for a build plan

To perform a dry run of migrating your Bamboo build plan to {% data variables.product.prodname_actions %}, run the following command in your terminal, replacing `:my_plan_slug` with the plan's project and plan key in the format `<projectKey>-<planKey>` (for example: `PAN-SCRIP`).

```shell
gh actions-importer dry-run bamboo build --plan-slug :my_plan_slug --output-dir tmp/dry-run
```

### Running a dry-run migration for a deployment project

To perform a dry run of migrating your Bamboo deployment project to {% data variables.product.prodname_actions %}, run the following command in your terminal, replacing `:my_deployment_project_id` with the ID of the deployment project you are converting.

```shell
gh actions-importer dry-run bamboo deployment --deployment-project-id :my_deployment_project_id --output-dir tmp/dry-run
```

You can view the logs of the dry run and the converted workflow files in the specified output directory.

{% data reusables.actions.gai-custom-transformers-rec %}

## Perform a production migration of a Bamboo pipeline

You can use the `migrate` command to convert a Bamboo pipeline and open a pull request with the equivalent {% data variables.product.prodname_actions %} workflow.

### Running the migrate command for a build plan

To migrate a Bamboo build plan to {% data variables.product.prodname_actions %}, run the following command in your terminal, replacing the `target-url` value with the URL for your {% data variables.product.prodname_dotcom %} repository, and `:my_plan_slug` with the plan's project and plan key in the format `<projectKey>-<planKey>`.

```shell
gh actions-importer migrate bamboo build --plan-slug :my_plan_slug --target-url :target_url --output-dir tmp/migrate
```

The command's output includes the URL to the pull request that adds the converted workflow to your repository. An example of a successful output is similar to the following:

```shell
$ gh actions-importer migrate bamboo build --plan-slug :PROJECTKEY-PLANKEY --target-url https://github.com/octo-org/octo-repo --output-dir tmp/migrate
[2022-08-20 22:08:20] Logs: 'tmp/migrate/log/actions-importer-20220916-014033.log'
[2022-08-20 22:08:20] Pull request: 'https://github.com/octo-org/octo-repo/pull/1'
```

### Running the migrate command for a deployment project

To migrate a Bamboo deployment project to {% data variables.product.prodname_actions %}, run the following command in your terminal, replacing the `target-url` value with the URL for your {% data variables.product.prodname_dotcom %} repository, and `:my_deployment_project_id` with the ID of the deployment project you are converting.

```shell
gh actions-importer migrate bamboo deployment --deployment-project-id :my_deployment_project_id --target-url :target_url --output-dir tmp/migrate
```

The command's output includes the URL to the pull request that adds the converted workflow to your repository. An example of a successful output is similar to the following:

```shell
$ gh actions-importer migrate bamboo deployment --deployment-project-id 123 --target-url https://github.com/octo-org/octo-repo --output-dir tmp/migrate
[2023-04-20 22:08:20] Logs: 'tmp/migrate/log/actions-importer-20230420-014033.log'
[2023-04-20 22:08:20] Pull request: 'https://github.com/octo-org/octo-repo/pull/1'
```

{% data reusables.actions.gai-inspect-pull-request %}

## Reference

This section contains reference information on environment variables, optional arguments, and supported syntax when using {% data variables.product.prodname_actions_importer %} to migrate from Bamboo.

### Using environment variables

{% data reusables.actions.gai-config-environment-variables %}

{% data variables.product.prodname_actions_importer %} uses the following environment variables to connect to your Bamboo instance:

* `GITHUB_ACCESS_TOKEN`: The {% data variables.product.pat_v1 %} used to create pull requests with a converted workflow (requires `repo` and `workflow` scopes).
* `GITHUB_INSTANCE_URL`: The URL to the target {% data variables.product.prodname_dotcom %} instance (for example, `https://github.com`).
* `BAMBOO_ACCESS_TOKEN`: The Bamboo {% data variables.product.pat_generic %} used to authenticate with your Bamboo instance.
* `BAMBOO_INSTANCE_URL`: The URL to the Bamboo instance (for example, `https://bamboo.example.com`).

These environment variables can be specified in a `.env.local` file that is loaded by {% data variables.product.prodname_actions_importer %} when it is run.

### Optional arguments

{% data reusables.actions.gai-optional-arguments-intro %}

#### `--source-file-path`

You can use the `--source-file-path` argument with the `dry-run` or `migrate` subcommands.

By default, {% data variables.product.prodname_actions_importer %} fetches pipeline contents from the Bamboo instance. The `--source-file-path` argument tells {% data variables.product.prodname_actions_importer %} to use the specified source file path instead.

For example:

```shell
gh actions-importer dry-run bamboo build --plan-slug IN-COM -o tmp/bamboo --source-file-path ./path/to/my/bamboo/file.yml
```

#### `--config-file-path`

You can use the `--config-file-path` argument with the `audit`, `dry-run`, and `migrate` subcommands.

By default, {% data variables.product.prodname_actions_importer %} fetches pipeline contents from the Bamboo instance. The `--config-file-path` argument tells {% data variables.product.prodname_actions_importer %} to use the specified source files instead.

##### Audit example

In this example, {% data variables.product.prodname_actions_importer %} uses the specified YAML configuration file to perform an audit.

```bash
gh actions-importer audit bamboo -o tmp/bamboo --config-file-path "./path/to/my/bamboo/config.yml"
```

To audit a Bamboo instance using a config file, the config file must be in the following format, and each `repository_slug` must be unique:

```yaml
source_files:
  - repository_slug: IN/COM
    path: path/to/one/source/file.yml
  - repository_slug: IN/JOB
    path: path/to/another/source/file.yml
```

##### Dry run example

In this example, {% data variables.product.prodname_actions_importer %} uses the specified YAML configuration file as the source file to perform a dry run.

The repository slug is built using the `--plan-slug` option. The source file path is matched and pulled from the specified source file.

```bash
gh actions-importer dry-run bamboo build --plan-slug IN-COM -o tmp/bamboo --config-file-path "./path/to/my/bamboo/config.yml"
```

### Supported syntax for Bamboo pipelines

The following table shows the type of properties that {% data variables.product.prodname_actions_importer %} is currently able to convert.

| Bamboo                              | GitHub Actions                                  |  Status                |
| :---------------------------------- | :-----------------------------------------------| ---------------------: |
| `environments`                      | `jobs`                                          |  Supported             |
| `environments.<environment_id>`     | `jobs.<job_id>`                                 |  Supported             |
| `<job_id>.artifacts`                | `jobs.<job_id>.steps.actions/upload-artifact`   |  Supported             |
| `<job_id>.artifact-subscriptions`   | `jobs.<job_id>.steps.actions/download-artifact` |  Supported             |
| `<job_id>.docker`                   | `jobs.<job_id>.container`                       |  Supported             |
| `<job_id>.final-tasks`              | `jobs.<job_id>.steps.if`                        |  Supported             |
| `<job_id>.requirements`             | `jobs.<job_id>.runs-on`                         |  Supported             |
| `<job_id>.tasks`                    | `jobs.<job_id>.steps`                           |  Supported             |
| `<job_id>.variables`                | `jobs.<job_id>.env`                             |  Supported             |
| `stages`                            | `jobs.<job_id>.needs`                           |  Supported             |
| `stages.<stage_id>.final`           | `jobs.<job_id>.if`                              |  Supported             |
| `stages.<stage_id>.jobs`            | `jobs`                                          |  Supported             |
| `stages.<stage_id>.jobs.<job_id>`   | `jobs.<job_id>`                                 |  Supported             |
| `stages.<stage_id>.manual`          | `jobs.<job_id>.environment`                     |  Supported             |
| `triggers`                          | `on`                                            |  Supported             |
| `dependencies`                      | `jobs.<job_id>.steps.<gh cli step>`             |  Partially Supported   |
| `branches`                          |  Not applicable                                 |  Unsupported           |
| `deployment.deployment-permissions` | Not applicable                                  |  Unsupported           |
| `environment-permissions`           | Not applicable                                  |  Unsupported           |
| `notifications`                     | Not applicable                                  |  Unsupported           |
| `plan-permissions`                  | Not applicable                                  |  Unsupported           |
| `release-naming`                    | Not applicable                                  |  Unsupported           |
| `repositories`                      | Not applicable                                  |  Unsupported           |

For more information about supported Bamboo concept and plugin mappings, see the [`github/gh-actions-importer` repository](https://github.com/github/gh-actions-importer/blob/main/docs/bamboo/index.md).

### Environment variable mapping

{% data variables.product.prodname_actions_importer %} uses the mapping in the table below to convert default Bamboo environment variables to the closest equivalent in {% data variables.product.prodname_actions %}.

| Bamboo                                           | GitHub Actions                                      |
| :----------------------------------------------- | :-------------------------------------------------- |
| `bamboo.agentId`                                 | {% raw %}`${{ github.runner_name }}`{% endraw %}
| `bamboo.agentWorkingDirectory`                   | {% raw %}`${{ github.workspace }}`{% endraw %}
| `bamboo.buildKey`                                | {% raw %}`${{ github.workflow }}-${{ github.job }}`{% endraw %}
| `bamboo.buildNumber`                             | {% raw %}`${{ github.run_id }}`{% endraw %}
| `bamboo.buildPlanName`                           | {% raw %}`${{ github.repository }}-${{ github.workflow }}-${{ github.job }`{% endraw %}
| `bamboo.buildResultKey`                          | {% raw %}`${{ github.workflow }}-${{ github.job }}-${{ github.run_id }}`{% endraw %}
| `bamboo.buildResultsUrl`                         | {% raw %}`${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}`{% endraw %}
| `bamboo.build.working.directory`                 | {% raw %}`${{ github.workspace }}`{% endraw %}
| `bamboo.deploy.project`                          | {% raw %}`${{ github.repository }}`{% endraw %}
| `bamboo.ManualBuildTriggerReason.userName`       | {% raw %}`${{ github.actor }}`{% endraw %}
| `bamboo.planKey`                                 | {% raw %}`${{ github.workflow }}`{% endraw %}
| `bamboo.planName`                                | {% raw %}`${{ github.repository }}-${{ github.workflow }}`{% endraw %}
| `bamboo.planRepository.branchDisplayName`        | {% raw %}`${{ github.ref }}`{% endraw %}
| `bamboo.planRepository.<position>.branch`        | {% raw %}`${{ github.ref }}`{% endraw %}
| `bamboo.planRepository.<position>.branchName`    | {% raw %}`${{ github.ref }}`{% endraw %}
| `bamboo.planRepository.<position>.name`          | {% raw %}`${{ github.repository }}`{% endraw %}
| `bamboo.planRepository.<position>.repositoryUrl` | {% raw %}`${{ github.server }}/${{ github.repository }}`{% endraw %}
| `bamboo.planRepository.<position>.revision`      | {% raw %}`${{ github.sha }}`{% endraw %}
| `bamboo.planRepository.<position>.username`      | {% raw %}`${{ github.actor}}`{% endraw %}
| `bamboo.repository.branch.name`                  | {% raw %}`${{ github.ref }}`{% endraw %}
| `bamboo.repository.git.branch`                   | {% raw %}`${{ github.ref }}`{% endraw %}
| `bamboo.repository.git.repositoryUrl`            | {% raw %}`${{ github.server }}/${{ github.repository }}`{% endraw %}
| `bamboo.repository.pr.key`                       | {% raw %}`${{ github.event.pull_request.number }}`{% endraw %}
| `bamboo.repository.pr.sourceBranch`              | {% raw %}`${{ github.event.pull_request.head.ref }}`{% endraw %}
| `bamboo.repository.pr.targetBranch`              | {% raw %}`${{ github.event.pull_request.base.ref }}`{% endraw %}
| `bamboo.resultsUrl`                              | {% raw %}`${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}`{% endraw %}
| `bamboo.shortJobKey`                             | {% raw %}`${{ github.job }}`{% endraw %}
| `bamboo.shortJobName`                            | {% raw %}`${{ github.job }}`{% endraw %}
| `bamboo.shortPlanKey`                            | {% raw %}`${{ github.workflow }}`{% endraw %}
| `bamboo.shortPlanName`                           | {% raw %}`${{ github.workflow }}`{% endraw %}

> [!NOTE]
> Unknown variables are transformed to {% raw %}`${{ env.<variableName> }}`{% endraw %} and must be replaced or added under `env` for proper operation. For example, `${bamboo.jira.baseUrl}` will become {% raw %}`${{ env.jira_baseUrl }}`{% endraw %}.

### System Variables

System variables used in tasks are transformed to the equivalent bash shell variable and are assumed to be available. For example, `${system.<variable.name>}` will be transformed to `$variable_name`. We recommend you verify this to ensure proper operation of the workflow.

## Legal notice

{% data reusables.actions.actions-importer-legal-notice %}
