# Astronomer Actions
Custom Github Actions to create CI/CD pipelines for Astro. See [Astornomer's CI/CD documentation](https://docs.astronomer.io/astro/ci-cd) page for more information on creating CI/CD pipelines to deploy code to Astro Deployments.

This repo contains the following Custom Github Actions:
- Deploy

## Deploy Action

Deploys your Astro project to an Astro Cloud Deployment during a CI/CD workflow. This action should be used with a github ations workflow that runs when code is merged into a "main"(or equivalent) branch. Works with DAG-Only Deploys enabled or disabled. Enabling Dag-Only Deploys will skip an image deploy if only files in `/dags` folder change. More information can be found in Astronomer's ][DAG-only Deploys documentation](https://docs.astronomer.io/astro/deploy-code#deploy-dags-only[]).

This action will execute the following steps if DAG:
1. Checkout your current repo
2. Determine if only DAG code changed
2. Build your Astro Project into an image if files outside the `/dags` folder changed
3. Parse or Pytest your DAG code (optional)
4. Push your image to Astro's image Registry if files outside the `/dags` folder changed
5. Deploy your image to your Astro Deployment if files outside the `/dags` folder changed
6. Deploy only your `dags/` folder to your Astronomer Deployment if only files inside the `/dags` folder changed

### Usage

The Deploy Action is designed to be used with [Deployment API keys](https://docs.astronomer.io/astro/api-keys). You must have your `ASTRONOMER_KEY_ID` and `ASTRONOMER_KEY_SECRET` set as environment variables in your Github Actions Workflow for this command to work. We recommend doing this through Github Actions secretes. For an example workflow script scroll down to the Examples section. The action uses these keys to login to Astro and determine what Deployment to deploy to. 

#### Inputs

You can configure the Deploy Actions behavior through a few options. None of the options are required.

1. **dag-deploy-enabled**: If true only DAGs will be deployed when only DAG files are changed. Only make true if DAG Deploys has been enabled for the Deployment you are deploying to.
2. **dag-folder**: Specify the location of the DAG folder. The DAG folder location is `./dags` by default.
3. **parse**: If true DAGs will be parsed before deploying to Astro.
4. **pytest**: if true custom Pytests will be ran before deploying to Astro.
5. **pytest-file**: Specify custom Pytest files to run with the pytest command. For example you could specify the test `/tests/test-tags.py`. By default this option is blank and all Pytests in located in the repo will be run
6. **force**: If true your code will be force deployedto Astornomer. Mostly uesd to skip parse test on image deploys.

### Example Workflow File

The following example shows how the Deploy Action can be uesd to deploy your code to an Astro Deployment whenever code is pushed to the main branch of your projects repository. The ASTRONOMER_KEY_ID and ASTRONOMER_KEY_SECRET values come from your Deployment API key.

In this particular example DAG deploys are enabled are enabled and DAG files are being parsed in both image and DAG Deploys.

```
name: Astronomer CI - Deploy code

on:
  push:
    branches:
      - main

env:
  ## Sets Deployment API key credentials as environment variables
  ASTRONOMER_KEY_ID: ${{ secrets.ASTRONOMER_KEY_ID }}
  ASTRONOMER_KEY_SECRET: ${{ secrets.ASTRONOMER_KEY_SECRET }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Deploy to Astro
      uses: astronomer/actions/deploy@v0.1
      with:
        dag-deploy-enabled: true
        parse: true
```
### Change DAG Folder Example

In the following example the folder `/example-dags/dags` is specified as the DAG folder.

```
steps:
- name: Deploy to Astro
  uses: astronomer/actions/deploy@v0.1
  with:
    dag-folder: /example-dags/dags/
```

### Example using Pytests

In the following example the pytest located at `/tests/test-tags.py` runs before the image is deployed. 

```
steps:
- name: Deploy to Astro
  uses: astronomer/actions/deploy@v0.1
  with:
    pytest: true
    pytest-file: /tests/test-tags.py
```

### Example using force

In the following example the parse test is skipped because the user's code is force deployed.

```
steps:
- name: Deploy to Astro
  uses: astronomer/actions/deploy@v0.1
  with:
    force: true
```

