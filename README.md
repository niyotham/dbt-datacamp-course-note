Overview
========

Welcome to Astronomer! This project was generated after you ran 'astro dev init' using the Astronomer CLI. This readme describes the contents of the project, as well as how to run Apache Airflow on your local machine.

Project Contents
================

Your Astro project contains the following files and folders:

- dags: This folder contains the Python files for your Airflow DAGs. By default, this directory includes two example DAGs:
    - `example_dag_basic`: This DAG shows a simple ETL data pipeline example with three TaskFlow API tasks that run daily.
    - `example_dag_advanced`: This advanced DAG showcases a variety of Airflow features like branching, Jinja templates, task groups and several Airflow operators.
- Dockerfile: This file contains a versioned Astro Runtime Docker image that provides a differentiated Airflow experience. If you want to execute other commands or overrides at runtime, specify them here.
- include: This folder contains any additional files that you want to include as part of your project. It is empty by default.
- packages.txt: Install OS-level packages needed for your project by adding them to this file. It is empty by default.
- requirements.txt: Install Python packages needed for your project by adding them to this file. It is empty by default.
- plugins: Add custom or community plugins for your project to this file. It is empty by default.
- airflow_settings.yaml: Use this local-only file to specify Airflow Connections, Variables, and Pools instead of entering them in the Airflow UI as you develop DAGs in this project.

Deploy Your Project Locally
===========================

1. Start Airflow on your local machine by running 'astro dev start'.

This command will spin up 4 Docker containers on your machine, each for a different Airflow component:

- Postgres: Airflow's Metadata Database
- Webserver: The Airflow component responsible for rendering the Airflow UI
- Scheduler: The Airflow component responsible for monitoring and triggering tasks
- Triggerer: The Airflow component responsible for triggering deferred tasks

2. Verify that all 4 Docker containers were created by running 'docker ps'.

Note: Running 'astro dev start' will start your project with the Airflow Webserver exposed at port 8080 and Postgres exposed at port 5432. If you already have either of those ports allocated, you can either [stop your existing Docker containers or change the port](https://docs.astronomer.io/astro/test-and-troubleshoot-locally#ports-are-not-available).

3. Access the Airflow UI for your local Airflow project. To do so, go to http://localhost:8080/ and log in with 'admin' for both your Username and Password.

You should also be able to access your Postgres Database at 'localhost:5432/postgres'.

Deploy Your Project to Astronomer
=================================

If you have an Astronomer account, pushing code to a Deployment on Astronomer is simple. For deploying instructions, refer to Astronomer documentation: https://docs.astronomer.io/cloud/deploy-code/

Contact
=======

The Astronomer CLI is maintained with love by the Astronomer team. To report a bug or suggest a change, reach out to our support.

DBT notes
=========

`dbt init <project name>`

```sql

/*
    Welcome to your first dbt model!
    Did you know that you can also configure models directly within SQL files?
    This will override configurations stated in dbt_project.yml

    Try changing "table" to "view" below
*/

{{ config(materialized='table') }}

with source_data as (

    select 1 as id
    union all
    select null as id

)

select *
from source_data

/*
    Uncomment the line below to remove records with null `id` values
*/

 where id is not null

```

```sql
-- Use the `ref` function to select from other models
select *
from {{ ref('my_first_dbt_model') }}
where id = 1
```
### schema yaml file example
``` yml

version: 2

models:
  - name: my_first_dbt_model
    description: "A starter dbt model"
    columns:
      - name: id
        description: "The primary key for this table"
        tests:
          - unique
          - not_null

  - name: my_second_dbt_model
    description: "A starter dbt model"
    columns:
      - name: id
        description: "The primary key for this table"
        tests:
          - unique
          - not_null



```
### dbt project for test

```yml

# Name your project! Project names should contain only lowercase characters
# and underscores. A good package name should reflect your organization's
# name or the intended use of these models
name: 'nyc_yellow_taxi'
version: '1.0.0'
config-version: 2

# This setting configures which "profile" dbt uses for this project.
profile: 'nyc_yellow_taxi'

# These configurations specify where dbt should look for different types of files.
# The `model-paths` config, for example, states that models in this project can be
# found in the "models/" directory. You probably won't need to change these!
model-paths: ["models"]
analysis-paths: ["analyses"]
test-paths: ["tests"]
seed-paths: ["seeds"]
macro-paths: ["macros"]
snapshot-paths: ["snapshots"]

clean-targets:         # directories to be removed by `dbt clean`
  - "target"
  - "dbt_packages"


# Configuring models
# Full documentation: https://docs.getdbt.com/docs/configuring-models

# In this example config, we tell dbt to build all models in the example/
# directory as views. These settings can be overridden in the individual model
# files using the `{{ config(...) }}` macro.
models:
  nyc_yellow_taxi:
    # Config indicated by + and applies to all files under models/example/
    example:
      +materialized: view


```

#### Creating a project profile
As requested by your manager, you've initialized your dbt project. Reading further through the documentation, you notice that you need to create the project profile, which defines where your database / data warehouse information is defined. Your project is experimental at this point, and needs to work with DuckDB, a locally running database tool.

You notice that dbt primarily uses YAML (abbreviated .yml) files for its configuration. On YAML files, you'll need to maintain spacing / formatting as illustrated to make sure the file parsing succeeds.

- Open the `nyc_yellow_taxi/profiles.yml` file in the editor window.
- Modify the project name (the first line) to `nyc_yellow_taxi`.
- Change the type: to duckdb.
- Use the command `dbt debug ` to verify there are no errors.

```sql
*nyc_yellow_taxi*:
  outputs:
    dev:
      type: *duckdb*
      path: dbt.duckdb
  target: dev

```
How the materialized view of `taxi_rides_raw.sql` look like in `dbt` style for *duckdb* with the function `read_parquet()`

```sql
{{ config(materialized='view')}}


with src_data as (
    select * from read_parquet('yellow_tripdata_2023-01-partial.parquet')
)

select * from source_data
 ```
#### Workflow for dbt

1. Working with a first project
00:00 - 00:06
We've reached the last video of the chapter! Let's look at the process for working with your dbt project(s).

2. Workflow for dbt
A workflow for dbt depends somewhat on the user's needs, but typically follows as First, create the project using dbt init (or copy a working project from another location) Next, define or update its configuration in the profiles.yml file. We've already done these steps. The next step, and where you'll spend most of your time developing with dbt, is defining and using the data models. We'll go over what a model represents more in the next chapter. For now, it's basically the transformed version(s) of your data that you wish to make accessible in the data warehouse. Once the models are defined, we need to instantiate them using the `dbt run` subcommand. This command will take the source SQL code you've written, translate it as necessary for your deployment target (aka, profile), and actually execute the transformation process. When complete, you'll need to verify and test your data and, if necessary, troubleshoot any issues. Finally, this process is repeated as necessary, typically going back to the model level when a new data source is required.

3. Verifying database connections
One step often needed for working with dbt is verifying the connectivity to a database or data warehouse. dbt includes the subcommand dbt debug, which is designed to check the connections to your defined databases or warehouses. Note that the data warehouse must be created and accessible first, or you will see failures in the output. When using DuckDB and dbt, this may require executing dbt run prior to use. We'll mention this for any exercise that requires it in this course.

4. dbt run
As noted previously, dbt run is the subcommand that actually performs the data transformations and pushes updates to the warehouse. It should be run whenever there are model changes, or when the data process needs to be materialized. Make sure to analyze the output of the dbt run command - it provides many details on the success or failures of each step in the dbt process. Please note that in this circumstance, materialized has a specific meaning in dbt. It means to execute the transformations on the source data and place the results into tables or views.

5. Table vs View
One thing to remember is the difference between tables and views within a database or data warehouse. There's a lot of technical implementation details, but in general a table is an object within a database or warehouse that actually holds data. These objects take up space within the database, relative to the size of data inserted into the database. The content of a table is only updated when a specific command changes said data. Views, act like a table and are queryable as such, but they actually hold no information. As such, they take up very little space within the database itself. A view is usually defined as a select query against another table or tables. As such, the content in the response is generated with each query. We don't cover the implementation details here, but are introducing them as dbt can create tables or views within a database depending on the configurations you define. The actual implementation details for tables and views will be dependent on the database in question.

#### Running a project
You've successfully created a project and defined the required parameters to connect to your data warehouse. Your manager is pleased with your accomplishments thus far and would like to see the project run in demo form. They would like the data to be materialized into the data warehouse from the initial data source, using some example configurations a colleague tested with previously. Note that your colleagues have provided a test script called datacheck to validate the contents of the data warehouse.

#### excerise 

- Use the command dbt run to materialize the data into the data warehouse, noting the error that appears.
- The query in the taxi_rides_raw.sql file needs to be updated. Change the text to use the following query:
```sql 
select * from read_parquet('yellow_tripdata_2023-01-partial.parquet')
```

- Run the project again and verify the error has been fixed.
- In the terminal, run the command ./datacheck to verify there are 300000 total_rides and look at a sample of the content from the data warehouse.

```python
#!/usr/bin/env python3
import duckdb
con = duckdb.connect('dbt.duckdb', read_only=True)
print(con.sql('select * from taxi_rides_raw limit 10'))
print(con.sql('select count(*) as total_rides from taxi_rides_raw'))
if (con.execute('select count(*) as total_rides from taxi_rides_raw').fetchall()[0][0] == 300000):
  with open('/home/repl/workspace/successful_data_check', 'w') as f:
    f.write('300000')

```

`solution`

``` sql
-- Modify the following line to change the materialization type
with source_data as (
    -- Add the query as described to generate the data model
    select * from read_parquet('yellow_tripdata_2023-01-partial.parquet')
)

select * from source_data

```

### Modifying a model 
Your manager is pleased with the progress you've made thus far, but some requirements have changed. After speaking with the analytics team, they're concerned about the response time of a model. This model is currently configured to generate a view in the data warehouse, rather than a table. Currently the dbt configuration is set to create a view rather than generate a table for querying. Your manager asks that you update the appropriate configuration in the model and regenerate the transformations.
![](image1)
- Open the `models/taxi_rides/taxi_rides_raw.sql` file and modify the appropriate configuration to generate a table.
- Run the `dbt` `subcommand` to execute the project.
Verify the command ran correctly and generated the correct type of database object.

```sql
{{ config(materialized='table')}}


with source_data as (
    select * from read_parquet('yellow_tripdata_2023-01-partial.parquet')
)

select * from source_data

```

Refresh incremental models
If you provide the --full-refresh flag to dbt run, dbt will treat incremental models as table models. This is useful when

The schema of an incremental model changes and you need to recreate it.
You want to reprocess the entirety of the incremental model because of new logic in the model code.

```dbt run --full-refresh```


### Creating a dbt model
You've learned a bit about models in dbt, but decide you'd like to implement one of your own prior to creating any for your team. You've decided to use the NYC Yellow taxi dataset to create a simple model representing all the trips taken on Jan 2, 2023.

After looking at the NYC Yellow Taxi dataset, you decide it will be best to create a simple model that uses all columns in the NYC Yellow Taxi Parquet file.

- Create a new file called taxi_rides_raw.sql in the nyc_yellow_taxi/models/taxi_rides/ directory.
- In that file, create a simple query to select all data from the yellow_tripdata_2023-01-partial.parquet file. Use a single line and use single quotes around the filename, rather than a table name.
- Execute the appropriate dbt subcommand from the nyc_yellow_taxi/ directory to materialize the model.


```sql
select * from 'yellow_tripdata_2023-01-partial.parquet'
```
`cd nyc_yellow_taxi`

[](image2)

#### Updating dbt models

1. Updating dbt models

Welcome back! Our next topic is the idea of updating models in dbt, whether those are ones we've created directly, or those created by colleagues.

2. Why update?

An advantage of working with dbt is to easily make changes to your project or your team's project without requiring you to start from scratch. Let's take a moment to review some reasons why you'd want to update your models: It could be an iterative task, where the requirements for your project have changed or have not been fully implemented yet. A common scenario is fixing bugs in the queries that define the models. Given that SQL can be considered a programming language of its own, there are likely to be issues that need to be fixed. Migrating the data, be it source or destination locations, can also require an update to your project.

1 Photo by Caspar Camille Rubin on Unsplash
3. Update workflow

As we've discussed why you'd need to update your models, let's look at a potential workflow to use when updating a dbt project. The first step is to check out a dbt project from your source control system, such as git. An example would be git clone dbt_project, then opening the dbt_project folder. You don't have to use git with dbt, but it is one of the advantages of doing, so as you can easily track changes / updates / modifications. Once you have the current project source, you'll want to find the appropriate model file in question and then update the query contents. This could be updating the query directly, creating a subquery, or otherwise modifying the .sql file contents. After updating the model or models, you'll need to apply these changes to the project. This can usually be done by simply running dbt run. Occasionally, larger changes need a full refresh of the model, which can be done by adding dash f to the dbt run command. If you see an error in your update or the results don't appear as expected, you might want to add the full refresh option and try again. Note that depending on your data and models, it might take a bit longer to run than a simple update. Finally, if any updates have been made and verified to work, you'll want to check the changes back into source control to keep the process easy in the future.

4. YAML files

In addition to directly updating .sql files for dbt models, you may also need to make changes in some YAML / .yml files. Typically these updates would be in one of two types of files, either the dbt_project.yml file or in a model_properties.yml file.

5. dbt_project.yml

The dbt_project.yml file contains settings that relate to the full project. This can include things such as the project name and version, as well as directory locations. The materialization settings for a model can also reside here, though settings in this file are applied globally. These include defined whether models are created as tables / views / etc in the data warehouse. Note that there is one dbt_project.yml file per project.

6. model_properties.yml

The model_properties.yml file is specific to settings and details for model information. This can include description, documentation details, and much more. We'll cover some more of this later in the course, but refer to the dbt documentation for more information. One interesting note is the file can actually be named anything as long as it exists somewhere in the models/ subdirectory and ends in a .yml extension. You can also have as many of these .yml files as needed.

##### exercises chapter 2
Updating a dbt model
Your team lead just forwarded you an email with a data request from the marketing department. They'd like to count all the credit card users by day be available in the data warehouse and automatically updated based on the current dbt models. The new model should be called total_creditcard_riders_by_day and contain columns for day and total_riders.

There is currently a model (meaning a SQL table) available under the name taxi_rides_raw that you can query against. The schema document describing this table is available here: NYC Yellow Taxi data schema

Update the SQL model code with the requirements listed above, using the NYC Yellow Taxi data schema to determine how to filter the model.
Initiate the dbt command to execute the model.
Run the validation command ./datacheck in the nyc_yellow_taxi directory to check the contents match.

```sql
-- Update with SQL to return requested information
select 
  date_part('day', tpep_pickup_datetime) as day,
  count(*) as total_riders
from taxi_rides_raw
where payment_type = 1
group by day
```


#### Updating model hierarchies

You have successfully created and updated a couple of models in your dbt project. Your manager has asked that you verify all models will be generated in the appropriate order so that there are no errors during the materialization process.

Note: be mindful of the spacing when writing functions.


- Execute your dbt project using the appropriate command and look for any error messages.
Open the model file referenced in the error.
- Update the SQL to use a Jinja ref template and save the file.
- Rerun your dbt project to verify the error has been remedied. This time, include the -f switch to guarantee everything is updated

```sql
-- Update SQL to use Jinja reference
select 
   date_part('day', tpep_pickup_datetime) as day,
   count(*) as total_riders
from {{ref('taxi_rides_raw')}}
where payment_type = 1
group by day
```
#### Model troubleshooting
1. Model troubleshooting

Great job on those last exercises! Now let's spend some time with the techniques you'll use to troubleshoot common issues while working with models in dbt.

2. Common model issues

dbt encompasses several data technologies - as such, there are several areas that can cause trouble. Let's look at some common problems you may see when creating models in dbt. A widespread issue is errors in your queries that create the model. These can include syntax errors (a misspelled keyword or column) or logic errors (the SQL isn't doing what you initially expected.) Models in dbt are typically written in SQL, so any problem with your SQL code can appear here. Another common problem is invalid object references. This could be as simple as misspelling the table name, but it could also indicate trickier issues. Depending on how you reference your objects, you may see them named differently than you expect, causing errors. Let's look at each of these more closely.


3. Query errors

As is common to anyone writing SQL queries, you may have to rewrite portions of your queries for many reasons. These include syntax-related issues such as misspellings, incorrectly ordering the query, or missing some necessary components. Another common problem is using non-standard SQL commands with a database that doesn't support it. This could include using TOP instead of LIMIT, custom functions, and so on. These queries may work with one dbt backend, but not with another. Lastly, common SQL logic issues can show up when creating dbt models. This includes forgetting to group by all non-aggregated columns as well as incorrectly formatting / referencing CTEs.

4. Invalid references

Another issue with writing dbt models is using an invalid reference. With dbt's different backends, the tables and views that are created can be referenced in different ways. The default method is to query the tables simply as named, but a different backend may use something different. For example, using Google's BigQuery looks for a context name first, while Databricks will often reference tables with a preceding underscore. A typical problem is trying to reference objects in your queries that have not yet been generated. dbt does its best to order the object creations based on the order of use, but sometimes there are circular references that keep this from happening.

5. Troubleshooting methods

There are several methods to determine exactly where problems in your dbt models may exist. The first is using `dbt run` to try generating and creating the dbt objects. If there are errors in creating the models, you'll receive an error message and a suggestion of what to do, if available. The next area to investigate are the dbt logs. The generic logs can be found in the logs directory under dbt.log. There is also a log file for each job called `run_results.json`. This log file contains various information about the tasks and can point out errors found during the run. While `dbt run` helps find many issues, sometimes the problem is more subtle. Another trick is manually reviewing the SQL output of the generated model. The problem may be apparent upon review, but you can also copy the generated code into a SQL editor (ideally with access to the data objects) and verify it works as you expect. Finally, make sure to verify your fixes work and don't cause other issues before continuing.


#### Troubleshooting model errors
While trying to implement a model, you notice several errors that prevent the model from being generated. Your team lead has requested this model be implemented as quickly as possible, so you'll need to try to troubleshoot what errors are being seen and how to fix them.


Try running the dbt project with the appropriate command.
Analyze the error displayed and make the appropriate modification, then try running it with dbt run again.
Determine the source of the next error and fix it appropriately.
Run the project one last time using the -f option to verify you've fixed all errors.

[](image3and4)

### Using sources
Sources make it possible to name and describe the data loaded into your warehouse by your Extract and Load tools. By declaring these tables as sources in dbt, you can then

select from source tables in your models using the {{ source() }} function, helping define the lineage of your data
test your assumptions about your source data
calculate the freshness of your source data
Declaring a source
Sources are defined in .yml files nested under a sources: key.

```yml
version: 2

sources:
  - name: jaffle_shop 
    database: raw  
    schema: jaffle_shop  
    tables:
      - name: orders
      - name: customers

  - name: stripe
    tables:
      - name: payments

```
*By default, schema will be the same as name. Add schema only if you want to use a source name that differs from the existing schema.

Selecting from a source
Once a source has been defined, it can be referenced from a model using the `{{ source()}} function.`
```sql
select
  ...

from {{ source('jaffle_shop', 'orders') }}

left join {{ source('jaffle_shop', 'customers') }} using (customer_id)

```

dbt will compile this to the full table name:

```sql
--target/compiled/jaffle_shop/models/my_model.sql

select
  ...

from raw.jaffle_shop.orders

left join raw.jaffle_shop.customers using (customer_id)
```

[](image test)

Defining tests on a model
After learning about tests within dbt, your manager has asked you to implement some quality tests on the data used by the marketing department. It seems some entries are appearing without values, while others have a value, but do not match up with what's expected in the data.

Specifically, the issues with the columns are as follows:

`fare_amount`
Column is occasionally blank, which causes issues in other calculations.
Should be present for all rows.
`payment_type`
Column values need to be between 1 and 6.
Should be present for all rows.

- Update the model_properties.yml file below to include an appropriate test for the `fare_amount column`.
- Add the test(s) for the`payment_type` field.
- Generate your models with dbt run.
- Test your data with the appropriate command

```yml
version: 2

models:
- name: taxi_rides_raw
  columns:
    - name: fare_amount
      tests:
        - not_null
    - name: payment_type
      test:
        - not_null
        - accepted_values:
            values: [1, 2, 3, 4, 5, 6]
```

#### Finding bad data
Your manager is pleased with your progress - after implementing the data validation tests in your project, they ask you to verify the current dataset for any issues. As such, you plan to test the data, identify any problems, and update your model to handle the issues.

The initial project has been materialized for you with dbt run. You can test without running it first.


- Use the appropriate command to execute your data validations.
- If an error is present, determine which validation has failed. Make sure to scroll up in any output so you can see it all.
- Open the model file and update the SELECT statement to handle the case of a 0 value in the payment_type field.
- Run dbt run and then rerun the validations to verify you've fixed the issue.

<solution\>

1. run `dbt test`

```sql
{{ config(materialized='view')}}

with source_data as (
    select * from read_parquet('yellow_tripdata_2023-01-partial.parquet')
)

-- Add a filter / WHERE clause to the line below
select * from source_data
where payment_type != 0

```
2. run `dbt run`
3. run `dbt test`

##### Creating singular tests

1. Creating singular tests

Nice work on the previous exercises! Now let's take a look at the first kind of custom tests available in dbt - singular data tests.

2. What is a singular test?

So what is a singular test? It is the simplest form of a custom data test within dbt. It is written as a SQL query, which must return the failing rows. Singular tests are defined as .sql files within the appropriate tests directory for the type of object being tested.


3. Example singular test

Let's take a look at an example singular test. Let's say we have a simplified order table, with 7 columns. Two of those columns are the subtotal and order_total. We want to create a test to verify that the order_total is greater than or equal to the subtotal. We're essentially trying to verify that all taxes / shipping / etc have been applied appropriately. We'll create a query of select * from order where order_total < subtotal This may seem a little strange as this is the opposite of what we're wanting to validate. Remember that when writing dbt singular tests, we want to return the rows that specifically fail our test condition. In this case, we're trying to verify that all rows have an order_total greater than or equal to the subtotal. As such, we write our query looking for any rows where the order_total is less than the subtotal, indicating a problem row. By convention, we'll name this file assert_order_total_gte_subtotal.sql and place it in the tests directory. Note that we could name this file whatever we wish, but it's helpful to be descriptive in the name. The test name will be referenced in any errors `/ logs` of the project, so it helps in debugging to know exactly what failed.

4. Singular test with Jinja

Let's look at another quick example, this time using Jinja templates. Much like when we define models, we can use Jinja functions to simplify the creation of the test. The most common function we'll use is the one you're already familiar with, the ref function. As a reminder, this handles substituting the proper name of an object when it's compiled for our target data warehouse. In this case, the r`ef('order')` function would return the proper table name for order within the test. There are other functions that can be used, such as the source function. We'll talk about these further in later videos. Finally, it should be noted that dbt performs the substitutions when the test is run. As such, if your dbt profile changes, you need to run your project again before testing.

5. Test debugging

One last topic before we move to exercises - let's discuss test debugging. The workflow for tests is similar to developing models, but is less involved. When creating a new test, it's often best to use a SQL editor to create the initial query and work through any typical SQL issues there. Place the query into the appropriate file, making sure to name the test uniquely. If you don't, dbt can still use it but it will be difficult to determine which version of a test passes or fails in the output. To speed development, you can use the dbt test `--select `testname command to run only that specific test. When you get a large dataset and have many tests, the amount of time required to run them all can increase greatly. The `--select` option should cut this down noticeably. Finally, check any errors in your test and update accordingly.

#### Verifying trip duration
Your teammate stops by your desk to discuss an issue being seen in the source data. He describes how on some rows, the duration of the trip occasionally shows as 0. While it could be filtered at the source data level, there are some downstream models that are storing the bad rows for discussion with the data provider. As such, he'd like you to develop a test that fails for triggering purposes if there's ever data where the trip duration is 0. You agree to look at the issue and implement the test code.

Looking at the schema for the taxi_rides_raw model, you realize there's not really a single field representing the trip duration. After discussing it further with your colleague, the primary issue seems to be that the trip start and end fields are the same. You realize you can quickly write a query to check for this.

You can refer to the schema document for information about the fields in the taxi_rides_raw table.

Note that the dbt project has already been executed for you and should not be required.

- Open the test file named assert_trip_duration_gt_0.sql in the proper directory.
- Update the SQL statement to verify that the start and end values for the trip are not equal.
- Execute just your validation using the appropriate command and verify how many rows fail the test. --> `dbt test --select assert_trip_duration_gt_0.sql`

```sql
select * 
from taxi_rides_raw
-- Complete the test on the following line
where tpep_pickup_datetime = tpep_dropoff_datetime

```

#### Creating custom reusable tests

Welcome back! Let's now take a look at the most powerful kind of test available to us - the generic, or reusable, test.

2. What is a reusable test?

A reusable test, also called a generic test in dbt terminology, is a test that can be reused in multiple situations. It's much like a built-in dbt test, but can check on any condition you can query within SQL. A generic test is built using Jinja templating - we'll go over the details of that implementation in a moment. The file is then saved as a .sql file within the tests/generic subfolder in the dbt project. It should be noted that to use a reusable test, it must be defined for each model that uses it in the model_properties.yml file. Again, this is similar to how we use the built-in tests within dbt.

1 Photo by Sigmund on Unsplash
3. Creating a reusable test
[](testimage)
To actually create a usable test, we need to add a few things to our .sql file. For most any generic test, we will be substituting in two objects - model and column_name. The are treated as arguments to a Jinja function that becomes our test name. The first line of our test file must contain at least `{% test testname(model, column_name) %}` I say at least, as it's possible to add further arguments, which we'll discuss in a moment. The next portion of the file contains our actual query, but note that we'll substitute in the model argument as a Jinja reference for the table, and the column_name argument as a reference for our validation check, usually in the where clause. Finally we'll add the line {% endtest %} to finish the code of the test.

4. Reusable test example

Here's a simple example of a reusable test to check if a column is greater than 0. First, we define the test as mentioned using check_gt_0 as the function name, with model and column_name as parameters. Our query is a simple `select * from table where object greater than 0`. Note that we use the `{{ model }} and {{ column_name }}` substitutions. Finally, we add the footer as required and save the file in the tests/generic directory.

5. Applying reusable test to model

To apply our new reusable test, we update our model_properties.yml file as we did with the built-in tests. For each test we wish to apply, we add it to the model / column as necessary. The model name is the model argument in the test. The column name is the column_name argument in the test. You'll see we have a built-in test defined on the taxi_rides_raw table, verifying that the `tpep_pickup_datetime is not null`. Let's now add our check_gt_0 test to the total_fare column. The test will now be applied when we next use dbt test.

6. Extra parameters
[](extraparam)
It is also possible to add extra parameters to our reusable tests. This is similar to how the accepted_values and relationships tests have extra entries in the `model_properties.yml` file. To add the extra arguments, we create the additional arguments on the Jinja header then substitute into our select query as required. You can theoretically add multiple extra parameters as needed, but it's suggested to keep the number of required parameters fairly short.

7. Applying extra parameters tests

To apply tests with extra parameters, we first add the test as normal to the model_properties.yml file. We then add the extra arguments as options under the test name. In this case, we define the check_columns_unequal entry to use the column_name2 parameter. Note the spacing as this is significant in YAML.


#### Implementing a reusable test
After learning about reusable tests, you realize that you'd love to build an in-house set of common tests for you and your team to use as you see fit. You notice that your datasets have several scenarios where you want to verify the columns contain values greater than 0. You decide to start your library by writing a reusable test for this purpose.

Note: Your data has not been materialized yet.

Ide Exercise Instruction
100XP
In the file check_gt_0.sql, update the test file to check for the appropriate values.
Add the test to the columns fare_amount and total_amount on the taxi_rides_raw table.
Materialize your data within dbt prior to testing your data.
Run the appropriate dbt test command to determine if there are any errors in the newly created .sql file.

create test 
```sql
{% test check_gt_0(model, column_name) %}
select * 
from {{ model }}
where {{column_name}} <= 0
{% endtest %}
```

model_profile.yml
``` yaml
# Replace any entry of ____ with the appropriate content
models:
- name: taxi_rides_raw
  columns:
    - name: fare_amount
      tests:
        - not_null
    - name: total_amount
      tests:
        - check_gt_0   
```


run `dbt test --select taxi_rides_raw.sql`

[](selectpics)

model_properties.yml

```yml
models:
- name: taxi_rides_raw
  columns:
    - name: tpep_pickup_datetime
      tests:
        - columns_equal:
            column_name2: tpep_dropoff_datetime

```
columns_equal.sql
```sql
{% test columns_equal(model, column_name, column_name2) %}
select *
from {{ model }}
where {{column_name}} = {{column_name2}}
{% endtest %}
```

run `dbt run` comand
run `dbt test --select taxi_rides_raw.sql`

### Creating and generating dbt documentation
00:00 - 00:09
Welcome back! Now that we've worked with tests in dbt, let's take a look at an often overlooked issue in data engineering and warehousing: documentation.

2. Why document?

It may sound like a silly question, but we should consider why we want to document a data project. It's common when working on a data project to create notes within the code and queries for ourself or any other engineers working at that level. Data projects, however have more consumers than just these roles and it's not feasible to provide everyone with access to source code just for documentation purposes. Centralizing the source of documentation is another benefit to documenting data projects. While it is possible to provide this information via something like email, it's far easier to have the documentation available at a known location. We're also interested in providing details about updates and changes to our data feeds, as well as creating a repository for examples, suggestions for the use of the data, and details about SLAs (Service Level Agreements - how often the data is updated and any guarantees of accessibility of the data.)

3. Creating documentation in dbt

dbt provides options to automatically add documentation to your project in various ways. This includes adding information to model definitions, including the overall model description as well as individual column descriptions if desired. The dbt documentation also can show the data lineage or DAG (directed acyclic graph), meaning the flow of data from initial source tables and any transformation or aggregate tables we create. We can also get any information about tests and data validations that are applied to our models from the dbt documentation tools. Finally, we can also see the details about the generated data warehouse - including the column data types and data sizes that are created when the data is processed.

4. Generating documentation in dbt

To actually generate the documentation, dbt provides a subcommand, dbt docs. The dbt docs subcommand has a few subcommands of its own. This includes the help option, dbt docs -h, which gives a description of the commands available for dbt docs. Let's talk about the primary one, dbt docs generate. This will traverse the content of our project, automatically creating the documentation website and formatting it into a static website. Given this documentation will update as we add models, tests, and so on, we should run this command after generating the project with dbt run.

5. Accessing documentation

To access the generated documentation, we'll need a web browser and the documentation to be hosted somewhere. There are several options for hosting the documentation depending on our needs. These can include using the other subcommand for dbt docs, the dbt docs serve subcommand. This starts a webserver on the local system and provides access to the documentation. Note that while convenient, this should only be used locally during development as it is not designed with security in mind. The other option for hosting the documentation is using another hosting service. This can include dbt cloud, Amazon's S3, any modern web server including Nginx, Apache, and so forth.

6. Documentation example
[](docuimage)
This is an example view of the documentation page, which can provide details of the models, description information, column details, and the lineage graphs.

Creating dbt documentation
Your ELT projects are moving forward well, and your team is pleased with the work you've done thus far. You've noticed that you're spending more time providing information about the models, the data types, and so forth. As updates are ongoing, you'd like to automate as much of your process as possible.

While reading the documentation for dbt, you discover the dbt docs commands and realize it would be a great solution to your problems. You would like to implement these into your project for your current and future models and then serve the data to other users. Please note that we will not view the documentation within this exercise but rather make sure the content can be served.

Ide Exercise Instruction
100XP
Modify the models/model_properties.yml file to add the following description to the appropriate line:
Initial import of the NYC Yellow Taxi trip data from Parquet source

Use the appropriate dbt command to write out the documentation.
Run the command to start the dbt documentation server.
Use Ctrl-C to cancel the documentation server.


run  `dbt docs serve` to start documentation server
#### dbt seeds

Nice work on the previous exercises! Let's take a moment and talk about dbt seeds and why we'd use them.

2. What are dbt seeds?

The first question you may ask is, what are dbt seeds? dbt seeds are CSV files that can be loaded into your data warehouse. These are typically rarely changing sets of data, such as lists of countries or lists of postal codes. Most importantly, though, seeds are not meant to contain raw data or data exported from another process.

3. Why?

You may be curious why to use dbt seeds. There are three primary reasons: First, the CSV files are relatively easy to manage. You can edit this manually if needed, copy them as needed, and so on. CSV files are easy to use in various scenarios, whether that's development, testing, or production options. Finally, CSV files, as text, are source controllable. These can be easily added into git repositories, compared against previous versions, and so forth.

4. How are seeds defined?

To define a seed in dbt, we can simply add a CSV file to the seeds subdirectory in your dbt project. Make sure the header is the first row of the file. Once ready, use the dbt seed command, and it will load the data into the data warehouse. Here's an example of a zipcode file found in our seeds directory. The header shows three columns of zipcode, place, and state, followed by a short list of locations. We then run the dbt seed command at the bash prompt to complete the import.

1 Postal codes provided via https://github.com/zauberware/postal-codes-json-xml-csv
5. Further configuration

There are further configuration options available for seeds in dbt. These include which schema and database to add the seed data. You can also define options, such as whether to quote any, all, or specific columns. More importantly, you can define which datatypes to assign to given columns within the seed data. These changes can be applied to a whole project, where it makes sense, or to individual seeds as needed. If added to the whole project, you can use the dbt_project.yml file. Otherwise, these can be added to the seeds/properties.yml file. Note that, as with other components of dbt, you can name the properties.yml file whatever you like as long as it's present in the seeds directory.

6. Defining datatypes

One very common operation when working with seeds is defining a datatype on the columns. The available types will depend on your data warehouse. The typical options are available, including integer, varchar, and so forth. If the type is not defined, the data type is inferred from the seed data. Here's a simple example with the necessary YAML structure. You might note that in this case, we're defining zipcode as a varchar of 5 characters instead of an integer. This is to preserve the leading zeros present in some US zip codes.

7. Tests & documentation

Seeds also support tests and documentation, just as models and sources do. In our zip code example, we've added a documentation description for the table and a unique validation test for the zipcode column.

8. Accessing seeds

A final note is how to access seeds in your models once they've been added to your data warehouse. To access the seed by name, you can use the Jinja ref function. The seed behaves as a model once created, so you'd use the same function as you would for a model.


##### ZIP is the code
In preparation for your dbt projects, you realize that having a list of New York & New Jersey zip (postal) codes and some additional information would be good to add. Reading through the information, you realize that utilizing the dbt seed functionality would be a great way to add this information. For reference, zipcodes are 5 digit numeric values representing the postal code of an area in the US.

The format of the zip file in question is:

zipcode,place,state
07093,West New York,New Jersey
12007,Alcove,New York
12009,Altamont,New York
12023,Berne,New York
This file is already downloaded for you as nynj_zipcodes.csv, but may not be in the correct location, nor have the correct configuration.


- Move the csv file nynj_zipcodes.csv into the appropriate subdirectory.
- Update the appropriate .yml file to define the zipcode column to preserve any leading zeros.
- Run the dbt seed command to generate the seed.
- Run the ./datacheck command to verify the data is available in your data warehouse.

datachek file
```python
#!/usr/bin/env python3
import duckdb
import os.path

try:
  con = duckdb.connect('dbt.duckdb', read_only=True)
  print(con.sql('select * from nynj_zipcodes'))
  if (con.execute('select count(*) as total_zipcodes from nynj_zipcodes').fetchall()[0][0] == 2877):
    print("Your data counts are correct!")
    with open('/home/repl/workspace/successful_data_check', 'w') as f:
      f.write('2877')
  else:
    print("There appears to be an issue with your data counts.")
except duckdb.CatalogException:
  if not os.path.isfile('/home/repl/workspace/nyc_yellow_taxi/seeds/nynj_zipcodes.csv'):
    print("It looks like the nynj_zipcodes.csv file is\n not present in the seeds/ directory.\n\nPlease move the file and try again.\n")
  else:
    print("It looks like your data warehouse doesn't exist yet\n\n Please run the dbt seed command and try again.\n")
```

properties.yml

```yml

version: 2

seeds:
  - name: nynj_zipcodes
    config:
      column_types:
        zipcode: varchar(5)

```
- Run the `dbt seed` command to generate the seed.
- Run the `./datacheck`

### dbt snapshopts


1. SCD2 with dbt snapshots
00:00 - 00:04
Welcome back! Let's cover a more advanced topic in dbt - snapshots.

2. What is a snapshot?
00:04 - 00:23
In general, a snapshot is a look into the changes of a dataset over time. Put another way, this illustrates the various states of an object and the times those states were valid. Consider something like a fast-food order status, the state of production for a car, or the shipping status for a book purchased online.

1 Photo by micheile henderson on Unsplash
3. SCD2
00:23 - 00:54
Multiple methods exist to monitor data changes/updates/etc within a data warehouse. One primary method is using what's known as a slowly changing dimension, specifically type 2. This is often abbreviated as SCD2, and is a common component of a Kimball-style data warehouse. You can refer to DataCamp's Introduction to Data Warehousing for more information. Primarily, SCD2 is a standard method to track changes over time. Snapshots are used in dbt to implement SCD2.

1 Photo by Luke Chesser on Unsplash
4. SCD2 example
00:54 - 01:30
Let's consider an example: order status, where we have three available states: Received, Packed, and Shipped. Normally, if we only update a given row, we lose the additional information about the states in between. If we show rows with a state of Packed, we're unsure how long it takes to complete the packing process once received. With Shipped alone, we can no longer determine how long that took place. Here we see a simple row showing the order id, its order status, and the last time it was updated. We'd love to see the following to easily identify when a row had a specific state without requiring changes to our application.

5. SCD2 in dbt
01:30 - 02:03
dbt can implement SCD2 using snapshots. It can automatically track our changes and add extra columns to our output. These columns are dbt_valid_from and dbt_valid_to, which track the datetimes of when the individual values for a row are valid. From our earlier example, you can see an updated version of our order_status information showing the state for Received, Packed, and Shipped along with the datetimes the rows were current. This is the most recent row if you see a null value in dbt_valid_to.

6. Implementing dbt snapshots
02:03 - 03:39
To implement a snapshot in dbt, we must define another file representing the data we want to snap. This is contained in a SQL file in the snapshots directory named snapshotname.sql, such as snapshot_orders.sql. The actual file content has several parts. First, we start with a Jinja snapshot command and the snapshot's name. The file ends with the endsnapshot directive. We use a Jinja config function within the directives, taking several arguments. The first is the target schema. This should be a separate schema from your default one, as snapshots can break existing tools due to the additional columns present. Next is the strategy used. Usually, this is timestamp, meaning that dbt should check the timestamp to determine validity. There is another strategy, check_cols, that we won't cover here. Basically, it checks a set of columns to determine what changed since the last snapshot. We then add a unique_key entry, the column's name representing the row, usually a primary key. The last argument is the updated_at entry, which takes the column's name, showing the datatime of the change. After closing the config entry, we need to add our SQL command to query against. This is often a dbt source, as a snapshot should be fairly early in the data process. Therefore, we can query for select * from our raw orders source. Note that we could use ref as well to access a model if preferred. There are other options available for snapshots and you can refer to the dbt documentation for an exhaustive list.

7. dbt snapshot
03:39 - 04:04
To create the snapshots, we need to use the dbt snapshot subcommand. Then, we can create or update a model to query against the snapshot_orders table using the ref Jinja command. Finally, we must run dbt snapshot often to see and maintain the potentially changed data. If we run the command infrequently, we may miss the changes in the data, so it's best to schedule the command to run automatically.

[](snapshotsteps)


##### Adding a snapshot
When working with your team, you learn about a new tangential dataset being added to the data warehouse. This dataset represents the set of vehicles that will be in use for a given taxi license. While a taxi is likely to be used for quite some time, it is possible that the license may be reassigned to a new vehicle during a given timeframe. One of your colleagues realizes this may cause issues with future reporting as a ride may not be represented by the proper vehicle.

The dataset looks like the following:

`column_name	description`
license_id	The numeric ID assigned to the taxi company
vehicle_make	The manufacturer of the vehicle
vehicle_model	The model of the vehicle
vehicle_year	The year the vehicle was manufactured
last_updated	Date when the record was last modified
Looking at this information you realize this is a great time to implement snapshots using dbt. After discussing this with the team, your team lead asks you to implement the snapshot functionality in the nyc_yellow_taxi project, using the source named 'raw.vehicle_list'.


- Add the snapshot directive to the vehicle_list_snapshot snapshot definition.
- Update the config directive to use the proper entries based on the columns in the raw.- - vehicle_list source.
- Write a query to find all rows from the source.
- Run the appropriate command to generate the snapshot.

```sql
{% snapshot vehicle_list_snapshot %}

{{
    config(
      target_schema='main',
      unique_key='license_id',
      strategy='timestamp',
      updated_at='last_updated'
    )
}}

select * from {{ source('raw', 'vehicle_list') }}
{% endsnapshot %}
```

run `dbt snapshot` command


### Potential next topics

1. Let's talk for a moment about some more advanced concepts in dbt that might be of interest to you in the future.

There is the concept of incremental models, which is processing only new / changed data in your warehouse without reloading everything. Studying more advanced SQL, including common table expressions or CTEs, will improve your models and allow for code reuse. There are many additional Jinja commands and macros available, including the ability to access environment variables and use Jinja loops to automatically define complex SQL code. A new feature in the latest versions of dbt include the ability to create models within Python in addition to SQL. While early, this could provide a lot of functionality and collaboration with dbt users who aren't as familiar with SQL. Documentation blocks is another idea to extend the level of information applied to dbt documentation, mainly giving the ability to add documentation inline to a given dbt object. Finally, there are many options for production and automation scenarios such as adding hooks to automate tasks and integrating with orchestrators like Apache Airflow.

2. References

There are many sources of information for learning more about dbt, including the dbt documentation found at docs.getdbt.com. The dbt slack channel is particularly active and covers pretty much any topic even tangentially related to dbt. It can be a little overwhelming, but people are quite eager to assist. There are of course countless blogs and newsletters covering dbt (including my own at news.thedatarodeo.com) in various forms. Finally, there will likely be future DataCamp courses covering more advanced usage of dbt.
