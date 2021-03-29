# GitHub Actions CI for SPOG dbt Tests PoC

## Setup Github Actions
1. Add `./.github/workflow` to the repo. Any YAML files in this directory will be treated as Actions by Github UI. 
Additionally, there is a flow in Github UI that can be used as well.
1. Add GitHub Actions YAML file to the new directory. This is the example `dbt_test.yml` file I used in the PoC:
```yaml
# This is a basic workflow to help you get started with Actions

name: DBT Test on PR

# Controls when the action will run. 
on:
  # Triggers the workflow on push or pull request events but only for the develop branch
  push:
    branches: [ develop ]
  pull_request:
    branches: [ develop ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Env values for this Action  
env:
  DBT_PROFILES_DIR: ./
  DBT_GOOGLE_PROJECT: dbt-hello-world-305619
  DBT_GOOGLE_BIGQUERY_DATASET: dbt_john
  DBT_GOOGLE_BIGQUERY_KEYFILE: ./dbt-service-account.json


# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "test"
  test:
    # This value will be displayed to the user during runtime
    name: Running DBT Tests
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - name: Checkout branch
      uses: actions/checkout@v2

    - name: Install Python 3.8.x
      uses: actions/setup-python@v1
      with:
        python-version: 3.8.x

    - name: Install DBT
      # Runs a set of commands using the runners shell
      run: |
        python -m pip install --upgrade pip
        pip install dbt
    - run: 'echo "$KEYFILE" > ./dbt-service-account.json'
      shell: bash
      env:
        KEYFILE: ${{secrets.DBT_GOOGLE_BIGQUERY_KEYFILE}}

    - name: Run dbt Transforms
      run: dbt run

    - name: Run dbt Tests
      run: dbt test
```

## Open Concerns regarding SPOG integration
* Secrect Management
  * I used Github Secrets for this project, but I am not familiar enough with our security concersn to know if this is a good solution for SPOG,
    or if we need a different solution for our secrets.
    * An additional cavet for Github Secrects is that I don't believe there is anyway for a user to extract a secrets contents after 
      they have been added (to compare, Codeship secrets could be pulled down, unencrypted and re-encrypted). This means that any secrets 
      that we need ongoing access for will need to be tracked elsewhere (most likely 1Pass).

* Timing
  * Part of the CI flow for SPOG must be a `dbt run` step, in order to run any new transformations. My concern is that a recent build took several 
    hours to resolve against our full data set. This would obviously be a big hinderance if every PR needed several hours to become available to merge. 
    I know that Sam Chapin believes that there are ways to expidite the `dbt run` runtime, and we need to ensure that these improvements are present in
    any CI solution we attempt.

* Environment Management
  * I didn't broach this concern during PoC development, but it is definitely a MVP concern that needs to be addresses sooner rather than later.

* Snowflake Specific Concerns
  * I used BigQuery integration for the PoC, since it was what I had available from the `dbt-tutorial` I implemented recently. As such, I am unaware 
    (unknown-unknowns) of what issues might arise from Snowflake integration that were not present in the BigQuery implementation. 
  * We might also want to look into what solutions are available as HG moves away from users on our Snowflake environment (i.e., not using a _user-password_ 
    mapping to authenticate with Snowflake).
