# DBT Cheat Sheet

## Table of Contents
- [DBT Cheat Sheet](#dbt-cheat-sheet)
  - [Table of Contents](#table-of-contents)
  - [Concepts](#concepts)
  - [DBT Folders and Key Files](#dbt-folders-and-key-files)
  - [Command Line](#command-line)
    - [Setup](#setup)
    - [Running or Building](#running-or-building)
    - [Linting](#linting)
    - [Other](#other)
  - [Code Snippets](#code-snippets)
    - [Models](#models)
      - [From a source](#from-a-source)
      - [Model from another model or seed](#model-from-another-model-or-seed)
    - [Materializations](#materializations)
    - [Macros](#macros)
    - [Tests](#tests)
      - [Generic tests](#generic-tests)
      - [Singular tests](#singular-tests)
    - [Snapshots](#snapshots)
  - [Common dbt Errors and Resolutions](#common-dbt-errors-and-resolutions)
  - [Understanding DAG (Directed Acyclic Graph)](#understanding-dag-directed-acyclic-graph)
  - [Links](#links)

## Concepts

| Concept | Description |
|---------|-------------|
| Project folder structure and yml | The organization of files and directories in a dbt project. Defined in the dbt_project.yml |
| Sources, seeds, models and materialisation | Sources: External tables referenced in dbt models<br>Seeds: CSV files loaded into the data warehouse<br>Models: SQL files defining transformations<br>Materialisation: How dbt persists models (e.g., table, view, incremental) |
| Schemas, contracts, documentation, metadata and generic data tests | Defined in the schema.yml files<br>Schemas & Contracts: Database structures and enforce data types and structure<br>Documentation & Metadata: Descriptions of models, columns<br>Generic data tests: Reusable tests for data validation |
| Command line | Interface to run dbt commands like dbt run, dbt build, dbt test, dbt seed, etc |
| Target folder | Directory where dbt stores compiled SQL files and run artifacts |
| Packages | Reusable dbt projects that can be included in your project. e.g., dbt_utils<br>Defined in packages.yml |
| Snapshots | Capture and store the state of a table at a point in time. Useful for slowly changing dimensions. |
| Jinja & Macros | Jinja: Templating language used in dbt to write dynamic SQL<br>Macros: Reusable SQL snippets or functions written in Jinja |
| Profile | Configuration that specifies how dbt connects to your data warehouse<br>Defined in the profiles.yml file can be in user folder .dbt or project |
| Singular and Unit tests | Singular tests: Custom tests defined in SQL files<br>Unit tests: Tests that validate individual components of the dbt project |

## DBT Folders and Key Files

| Folder/File | Description |
|-------------|-------------|
| analysis | Contains SQL files for analysis that are not part of the core transformation. |
| data_tests | Holds custom data tests to validate data quality and integrity. |
| dbt_packages | Directory for installed dbt packages. |
| macros | Contains reusable SQL snippets and functions. |
| models | Directory for dbt models, which are SQL files defining transformations. |
| seeds | Holds CSV files that can be loaded into the database as tables. |
| snapshots | Contains snapshot definitions to capture changes in source data over time. |
| target | Directory where compiled SQL and run results are stored. |
| target\compile | Subdirectory for compiled SQL files. |
| target\run | Subdirectory for run results and artifacts. |
| dbt_project.yml | Main configuration file for the dbt project. |
| profiles.yml | Configuration file for database connection profiles and environment targets. |
| packages.yml | Configuration file for specifying dbt packages to install. |
| schema.yml | Defines tests, documentation, and relationships for models and sources. |
| sources.yml | Defines source tables and their metadata. |
| model.sql | Example SQL file defining a dbt model. |

## Command Line

### Setup

- [Trusted Adapters](https://docs.getdbt.com/docs/trusted-adapters)
- [Community Adapters](https://docs.getdbt.com/docs/community-adapters)

```bash
pip install dbt-core dbt-duckdb            # Install the dbt you have installed and adapter(s)
pip install --upgrade dbt-core dbt-duckdb  # Upgrade the version of dbt you have installed
dbt --version                                  # Get the version of dbt
dbt init project_name                         # Setup a new project called project_name
dbt debug                                     # Check your dbt setup
pip install sqlfluff sqlfluff-templater-dbt   # Install SQL linter
dbt deps                                      # Install or update any dept packages in packages.yml
dbt clean                                     # Remove packages and target folder. Run dbt deps after
pip install poetry                            # Install Poetry python package manger
```

Using Poetry Python Env Manager
```bash
pip install poetry                            # Install poetry
poetry self add poetry-plugin-shell           # install poetry shell
poetry init --name my_project_env             # setup poetry project (creates pyproject.yoml)
poetry add dbt-core dbt-duckdb                # Add python packages to poetry
poetry install                                # Install/update python packages using poetry and updates lock file
poetry lock                                   # update your lock file
poetry shell                                  # Switch the poetry managed python environment
peotry run dbt debug                          # run command in poetry managed python environment
```

### Running or Building
```bash
dbt build                                     # Run/seed/test/snapshot all your dbt models
dbt run                                       # Test all your dbt model tests
dbt test                                      # Test all your dbt model tests
dbt run --select my_dbt_model                 # Runs a specific model .sql is optional
dbt run -s my_dbt_model another_dbt_model.sql # Runs the models .sql is optional. -s is the same as --select
dbt compile -s my_dbt_model another_dbt_model.sql # Compile the models .sql is optional. -s is the same as --select
dbt run --select models\risk360\staging       # Runs all models in a specific directory
dbt run --select tag:nightly                  # Run models with the "nightly" tag
dbt run --vars 'run_date : "2021123"'         # Set a variable at run time
dbt run --select tag:staging                  # Run everything tagged as staging
dbt run --select +my_dbt_model                # Run everything up to and including my_dbt_model
dbt run --select my_dbt_model+                # Run everything from my_dbt_model and onwards
dbt run --select my_dbt_model+1               # Run my_dbt_model and one level of onward dependencies
dbt run --select +my_dbt_model+               # Run everything that is needed for my_dbt_model and everything that uses it
dbt run -s my_dbt_model                       # Runs a specific model .sql is optional  -s and --select are the same
dbt run -s my_dbt_model --target dev          # Runs a specific model against target dev
dbt run -s my_dbt_model -t dev                # Runs a specific model against target dev -t and --target are the same
dbt run --select my_dbt_model --dry_run       # Compiles against the DB my_dbt_model but without running the code
dbt run --select my_dbt_model --dry_run --empty # Compiles against the DB my_dbt_model but without running the code and avoids using any source data
dbt parse                                     # Check your project without connecting to the database
dbt source freshness                          # Run the freshness tests on your sources
dbt seed                                      # Run all your dbt seeds
dbt snapshot                                  # Run all your dbt snapshots
dbt snapshot --select my_snapshot             # Run selected dbt snapshots
dbt retry                                     # Rerun only the failed or skipped items from the last run. Use after fixing an issue
dbt run --select my_dbt_model --full-refresh  # Refresh incremental models fully
dbt run --select tag:staging --exclude my_dbt_model # Run everything tagged as staging excluding my_dbt_model
dbt run --select config.materialized:view     # select only view materializations
```

### Linting
```bash
sqlfluff fix filename                        # Reformat and check SQL format single file
sqlfluff fix path                            # Reformat and check SQL format in path
```

### Other
```bash
dbt clean                                    # A utility function that deletes all folders specified in the clean-targets list
dbt docs generate                           # Create the documentation
dbt serve --port 8081                       # Open the documentation page on port 8081
dbt run-operation my_macro                  # Run a macro manually. Useful for execution sql that does not build a table
dbt show --select my_dbt_model             # Show the first 5 rows from the model
dbt show --inline "select * from {{ ref('my_dbt_model') }}" limit 10  # Show the first 5 rows from the sql query
dbt source freshness --select source:my_source  # Show if the source table meets the freshness settings
dbt parse                                   # Quick check of the code integrity without connecting to the database
dbt list --select tag:staging              # List the models that are in the selection
dbt test --store-failures                  # Store the failure records for tests in a table
```

## Code Snippets

### Models

#### From a source
Define a source in the source.yml:
```sql
SELECT col1, col2
FROM {{ source('raw', 'my_source_table') }}
```

```yaml
version: 2

sources:
  - name: raw
    schema: raw
    description: raw
    tables:
      - name: my_source_table
```

#### Model from another model or seed
```sql
SELECT col1, col2
FROM {{ ref('my_model') }}
```

```sql
SELECT col1, col2
FROM {{ ref('my_seed') }}
```

### Materializations
If omitted uses the dbt_project.yml setting for the folder or view

```sql
{{ config(
    materialized='table'
) }}

SELECT col1, col2
FROM {{ ref('my_model') }}
```

```sql
{{ config(
    materialized='view'
) }}

SELECT col1, col2
FROM {{ ref('my_model') }}
```

```sql
{{ config(
    materialized='incremental',
    incremental_strategy='merge',
    unique_key=['col1','col2']
) }}

SELECT col1, col2, col3, updated_at
FROM {{ ref('my_model') }}
{% if is_incremental() %}
WHERE updated_at > (SELECT MAX(updated_at) FROM {{ this }})
{% endif %}
```

### Macros

Definition:
```sql
{% macro calculate_revenue(total_sales, cost_of_goods_sold) %}
{{ total_sales }} - {{ cost_of_goods_sold }}
{% endmacro %}
```

Usage:
```sql
SELECT
    order_id,
    {{ calculate_revenue('total_sales', 'cogs') }} AS revenue
FROM {{ ref('orders') }}
```

### Tests

#### Generic tests
Put in the yml file for the model:
```yaml
version: 2

models:
  - name: orders
    columns:
      - name: order_id
        data_tests:
          - unique
          - not_null
      - name: status
        data_tests:
          - accepted_values:
              values: ['placed', 'shipped', 'completed', 'returned']
      - name: customer_id
        data_tests:
          - relationships:
              to: ref('customers')
              field: id
```

#### Singular tests
Put in the data_tests folder:
```sql
-- Refunds have a negative amount, so the total amount should always be >= 0.
-- Therefore return records where total_amount < 0 to make the test fail.
select
    order_id,
    sum(amount) as total_amount
from {{ ref('fct_payments') }}
group by 1
having total_amount < 0
```

### Snapshots
```sql
{% snapshot user_snapshots %}
{{ config(
    target_schema='snapshots',
    unique_key='id',
    strategy='timestamp',
    updated_at='updated_at'
) }}

SELECT
    id,
    name,
    email,
    updated_at
FROM {{ source('raw_data', 'users') }}
{% endsnapshot %}
```

## Common dbt Errors and Resolutions

| Issue | Example | Resolution |
|-------|---------|------------|
| Duplicate Model Names | Two models both called "customers" | Rename one model or reorganize project |
| Missing Dependencies | `{{ ref('sales') }}` for non-existent model | Ensure referenced model exists |
| Missing Sources | `{{ source('raw'.'sales') }}` | List sources in sources.yml |
| Incorrect Database Setup | Wrong schema or database in config | Update dbt_project.yml or profile settings |
| Syntax or SQL Errors | Forgotten comma or unbalanced parentheses | Check SQL syntax |
| Missing Configs | No materialization specified | Add config block |
| Missing Package Dependency | Unrecognized macro | Include in packages.yml, run `dbt deps` |
| Missing Macro | Undefined macro | Define in macros/ directory |
| Circular References | Model A references Model B, which references Model A | Remove or refactor references |

## Understanding DAG (Directed Acyclic Graph)

A DAG is a network of tasks that:
- Flows from one task to another in a sequence
- Has no loops
- Prevents infinite loops
- Simplifies pipeline management
- Each node represents a task, and directed edges show dependencies between tasks.

## Links
- [What is dbt? | dbt Developer Hub](https://docs.getdbt.com/docs/introduction)
- [dbt Labs | Transform Data in Your Warehouse](https://www.getdbt.com/)
- [dbt - YouTube Channel](https://www.youtube.com/@dbt-labs)
- [Dbt Training Getting Started](https://courses.getdbt.com/courses/fundamentals)
- [dbt Quickstarts | dbt Developer Hub](https://docs.getdbt.com/quickstarts)
- [Introduction to dbt (data build tool) from Fishtown Analytics](https://www.youtube.com/watch?v=5rNquRnNb4E)
- [dbt-labs/jaffle_shop_duckdb](https://github.com/dbt-labs/jaffle_shop_duckdb)
- [Learn dbt with courses and workshops from dbt Labs | dbt Labs](https://courses.getdbt.com/)
