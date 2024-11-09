# Core #5 Instructions #
## Objective ##
- In this assignment, students will practice their fundamental dimensional modeling skills and ELT by creating and populating a star schema for a given business process and relational database.
- For context, Core #3, #4, and #5 all build upon each other. 
     - Core #3: You created a draft dimensional model in LucidChart based on a given business process and transaction. 
     - Core #4: You will populate your draft dimensional model (and act upon any feedback received) from a dataset provided to you via a file. 
     - Core #5: You will Airbyte to pull in all of Oliver’s relational database data from SQL Server and then use dbt to transform and populate our final dimensional model. 
-HINT! The dbt exercise we completed in class will be EXTREMELY helpful as you complete the assignment. 

## Background & Data ##
- Oliver’s sweets and drinks serves a variety of products, including coffee, a variety of sodas, popcorn, and other tasty treats. Oliver’s currently has a transactional database system to track all store purchases across their 10 stores. They are now interested in developing a data warehouse using dimensional modeling (star schema) to improve their data analysis capabilities.  You will be modeling the point-of-sale business process at Oliver’s. You have already created a sample dimensional model and populated it for Oliver’s by uploading files of data. Now, after creating the dimensional model and showing it to Oliver’s data team, they have settled on the below dimensional model design. 
![alt text](oliverdimmodel.png)

- Oliver’s transaction data is stored in the 5360_Oliver database. Access this database by using the following credentials:
     - Username: 5360_student
     - Password: datawarehousing


## Assignment ##
- Now, we are going to use a semi-normalized transactional database given to us by Oliver's. You are very familiar with this dataset! You are going to create an ELT process for Oliver's using Airbyte & dbt. Download this markdown file and open it in VSCode, then populate the empty "code" boxes below with the code you use to complete this assignment. Then, submit this markdown file in Canvas. All teammates should complete the assignment in their own database, but you can troubleshoot together! Also, submit proof the data loaded in your Snowflake database (this can be a screenshot/query output).
### Extract and Load (Airbyte) ###
- Ensure that you are connected to the University VPN
- Open the Docker application
- Start Airbyte by opening a terminal and running the following (you may be able to just click the local host link below instead of running the following):
``` cd airbyte ```
``` ./run-ab-platform.sh ```
- Open up a browser and go to http://localhost:8000. It can take a while for the Airbyte service to start, so don't be surprised if it takes ~10 minutes.
    - Username: airbyte
    - Password: password
- Click `Set up a new source`
- When defining a source, select `Microsoft SQL Server (MSSQL)`
    - Host: `stairway.usu.edu`
    - Port: `1433`
    - Database: `5360_oliver`
    - Username: `5360_student`
    - Password: `datawarehousing` (you'll need to click the dropdown for optional fields)
- Select `Scan Changes with User Defined Cursor`
- Click `Set up source`
    - Airbyte will run a connection test on the source to make sure it is set up properly
- Create a schema in your firstnamelastname database named `Oliver` and ensure you have a data warehouse named `lastname_wh`

- Once Airbyte has run the connection test successfully, you will pick a destination, select `Pick a destination`.
- Find and click on `Snowflake`
    - Host: `https://rbb67081.snowflakecomputing.com` 
    - Role: `TRAINING_ROLE` 
    - Warehouse: `lastname_WH` 
    - Database: `firstnamelastname` 
    - Schema: `Oliver` (create this schema in your firstnamelastname database)
    - Username: 
    - Authorization Method: `Username and Password`
    - Password: 
    - Click `Set up destination`
- Once the connection test passes, it will pull up the new connection window
    - Change schedule type to `Manual`
    - Under `Activate the streams you want to sync`, click the button next to each table.
    - Click Set up connection
    - Click `Sync now`
    - Once it's done, go to Snowflake and verify that you see data in the landing database

### Transform (dbt) ###
- Open VSCode
- File > Open > Select your project (lastname_DW)
- On the top bar of the application, select Terminal > New Terminal
    - This will open a terminal in the directory of your project within VSCode
- Right click on the models directory and create a new folder inside of it. (Be careful not to create it inside of the example directory.)
- Call this new folder `oliver`
- Right click on oliver and create a new file. Name this file `_src_oliver.yml`
    - In this file we will add all of the sources for Oliver's tables
- Populate the code that we will use in this file below: 
```

```

- If you need to make any changes to your Snowflake information in your dbt project you can change it by going to your dbt profile.yml file. You may need to change the schema. 
    - On a mac, this is located under your user directory. You have to click Shift + command + . in order to see hidden folders. The .dbt folder will appear and inside is profiles.yml
    - On Windows, it's just in the user directory under the .dbt folder and the profiles.yml is inside.
    - Once you have found the profiles.yml file you can open in a text editor, change the needed parameters and save the file. 


#### dim customer ####
- Create a new file inside of the oliver directory called `oliver_dim_customer.sql`
- Populate the code that we will use in this file below: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_oliver'
)}}

select 
    customer_id as customer_key,
    customer_id,
    first_name,
    last_name,
    email,
    state,
    phone_number
    
from {{ source('oliver_landing', 'customer')}}
```

- Save the file, after you have done that, you can go to your terminal and type `dbt run -m oliver_dim_customer` to build the model.
    - Go to Snowflake to see the newly created table!

#### dim date ####
- Create a new file inside of the oliver directory called `oliver_dim_date.sql`
- Populate the code that we will use in this file below: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_oliver'
)}}

with cte_date as (
    {{ dbt_date.get_date_dimension("1900-01-01","2050-12-31")}}
)
select 
    date_day as date_key,
    date_day as date_id,
    day_of_week as dayofweek,
    month_of_year as month,
    quarter_of_year as quarter,
    year_number as year
from cte_date
```

- Save the file, after you have done that, you can go to your terminal and type `dbt run -m oliver_dim_date` to build the model. Go to Snowflake to see the newly created table!

#### dim_employee ####
- Create a new file inside of the oliver directory called `oliver_dim_employee.sql`
- Populate the code that we will use in this file below: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_oliver'
)}}

select 
    employee_id as employee_key,
    employee_id,
    first_name,
    last_name,
    email,
    position,
    hire_date,
    phone_number
from {{ source('oliver_landing', 'employee')}}
```

- Save the file and build the model. Go to Snowflake to see the newly created table! 

#### dim product ####
- Create a new file inside of the oliver directory called `oliver_dim_product.sql`
- Populate the code that we will use in this file below: 


```
{{ config(
    materialized = 'table',
    schema = 'dw_oliver'
)}}

select 
    product_id as product_key,
    product_id,
    product_name,
    description
from {{ source('oliver_landing', 'product')}}
```

- Save the file and build the model. Go to Snowflake to see the newly created table!


#### dim store ####
- Create a new file inside of the oliver directory called `oliver_dim_store.sql`
- Populate the code that we will use in this file below: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_oliver'
)}}

select 
    store_id as store_key,
    store_id,
    store_name,
    street,
    city,
    state
from {{ source('oliver_landing', 'store')}}

```

- Save the file and build the model. Go to Snowflake to see the newly created table!


#### fact sales ####
- Create a new file inside of the oliver directory called `fact_sales.sql`
- Populate the code that we will use in this file below: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_oliver'
)}}

select
    p.product_key,
    c.customer_key,
    e.employee_key,
    s.store_key,
    d.date_key,
    ol.unit_price,
    ol.quantity,
    (ol.unit_price * ol.quantity) as dollars_sold
from {{ source('oliver_landing', 'orderline') }} ol
inner join {{ source ('oliver_landing', 'orders') }} o
    on o.order_id = ol.order_id
left join {{ ref('oliver_dim_product') }} p
    on ol.product_id = p.product_id
left join {{ ref('oliver_dim_customer') }} c
    on o.customer_id = c.customer_id
left join {{ ref('oliver_dim_employee') }} e
    on o.employee_id = e.employee_id
left join {{ ref('oliver_dim_store') }} s
    on s.store_id = o.store_id
left join  {{ ref('oliver_dim_date') }} d
    on d.date_key = o.order_date

```

- Save the file and build the model. Go to Snowflake to see the newly created table!



#### schema yaml file ####
- Create a new file inside the oliver directory called `_schema_oliver.yml`
- This file contains metadata about the models you build. Hint: check out the exercise to help you create this file. 
- Populate the code that we will use in this file below: 
```
version: 2

models:
  - name: oliver_dim_employee
    description: "Oliver's Employee Dimension"
    columns: 
      - name: employee_key
        description: "Dimension Surrogate Key"
        tests:
        - unique
        - not_null
  - name: oliver_dim_customer
    description: "Oliver's Customer Dimension"
  - name: oliver_dim_date 
    description: "Oliver's Date Dimension"
  - name: oliver_dim_product
    description: "Oliver's Product Dimension"
  - name: oliver_dim_store
    description: "Store Information Dimension"  
  - name: fact_sales
    description: "Oliver's Sales Fact"
    columns: 
      - name: dollars_sold
        decription: "Quantity * Unit Price Per Line Item"
```

## Create a semantic layer model (2 points of EC!)
- Create a model that can query from the data warehouse we just built and reference upstream models.
- Create a new file called `sem_sales.sql` inside of the oliver directory.
- Basically, your code will create a new table that will be a semantic layer that is easy for consumption. The table should include key information that an analyst could easily pull from to run quick analysis. 
- This model should use 'ref' instead of source in the from statements. This will allow dbt to build lineage dag of the model dependencies:
- Populate the code that we will use in this file below: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_oliver'
)}}

select
    c.first_name as cust_fname,
    c.last_name as cust_lname,
    e.first_name as employee_fname,
    e.last_name as employee_lname,
    s.store_name as store,
    d.date_id as date_of_purchase,
    f.quantity as quantity_purchased,
    f.unit_price as product_price,
    f.dollars_sold as total_dollars_spent
from {{ ref('fact_sales') }} f

left join {{ ref('oliver_dim_product') }} p
    on f.product_key = p.product_key

left join {{ ref('oliver_dim_customer') }} c
    on f.customer_key = c.customer_key

left join {{ ref('oliver_dim_employee') }} e
    on f.employee_key = e.employee_key
    
left join  {{ ref('oliver_dim_date') }} d
    on f.date_key = d.date_key

left join  {{ ref('oliver_dim_store') }} s
    on f.store_key = s.store_key
```

- In order to view lineage, the dbt power user extension must be installed. Click on the Lineage tab in vscode (down by the terminal on the bottom), if you are inside the sem_claims.sql model, you should be able to see lineage for that model. View the lineage for the other files in the model as well. 
