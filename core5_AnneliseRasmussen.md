# Project #3 Instructions #
## Objective ##
- In this assignment, students will practice their fundamental dimensional modeling skills and ELT by creating and populating a star schema for a given business process and relational database.
del. 
-HINT! The dbt exercise we completed in class will be EXTREMELY helpful as you complete the assignment. 

## Background & Data ## 


- Group's transaction data is stored in the dw_group2 database. Access this database by using the following credentials:
     - Username: dw_group2
     - Password: password in canvas


## Assignment ##
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
    - Database: `dw_group2`
    - Username: `dw_group2`
    - Password: `password in canvas` (you'll need to click the dropdown for optional fields)
- Select `Scan Changes with User Defined Cursor`
- Click `Set up source`
    - Airbyte will run a connection test on the source to make sure it is set up properly
- Create a schema in your group2project database named deliverable3 and ensure you have a data warehouse named `group2_wh`

- Once Airbyte has run the connection test successfully, you will pick a destination, select `Pick a destination`.
- Find and click on `Snowflake`
    - Host: `https://sfedu02-etb90388.snowflakecomputing.com` 
    - Role: `TRAINING_ROLE` 
    - Warehouse: `group2_WH` 
    - Database: `group2project` 
    - Schema: `samssubs` 
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
- File > Open > Select your project (group2_wh)
- On the top bar of the application, select Terminal > New Terminal
    - This will open a terminal in the directory of your project within VSCode
- Right click on the models directory and create a new folder inside of it. (Be careful not to create it inside of the example directory.)
- Call this new folder `samssubs`
- Right click on samssubs and create a new file. Name this file `_src_samssubs.yml`
    - In this file we will add all of the sources for Sam's Subs' tables
- Populate the code that we will use in this file below: 
```
version: 2

sources:
  - name: subs_landing
    database: anneliserasmussen
    schema: samssubs
    tables:
    - name: Customer
    - name: Employee
    - name: Order_Line
    - name: Orders
    - name: Product
    - name: Product_Type
    - name: Store
    - name: Sandwich

```

- If you need to make any changes to your Snowflake information in your dbt project you can change it by going to your dbt profile.yml file. You may need to change the schema. 
    - On a mac, this is located under your user directory. You have to click Shift + command + . in order to see hidden folders. The .dbt folder will appear and inside is profiles.yml
    - On Windows, it's just in the user directory under the .dbt folder and the profiles.yml is inside.
    - Once you have found the profiles.yml file you can open in a text editor, change the needed parameters and save the file. 


#### dim customer ####
- Create a new file inside of the samssubs directory called `subs_dim_customer.sql`
- Populate the code that we will use in this file below: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_samssubs'
)}}

select 
    Customer_ID as customer_key,
    Customer_ID as customer_id,
    Customer_First_Name as fname,
    Customer_Last_Name as lname,
    Customer_Birthday as dob,
    phone_number
    
from {{ source('subs_landing', 'Customer')}}
```

- Save the file, after you have done that, you can go to your terminal and type `dbt run -m subs_dim_customer` to build the model.
    - Go to Snowflake to see the newly created table!

#### dim date ####
- Create a new file inside of the Sam's Subs directory called `subs_dim_date.sql`
- Populate the code that we will use in this file below: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_samssubs'
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

- Save the file, after you have done that, you can go to your terminal and type `dbt run -m subs_dim_date` to build the model. Go to Snowflake to see the newly created table!

#### dim_employee ####
- Create a new file inside of the Sam's Subs directory called `subs_dim_employee.sql`
- Populate the code that we will use in this file below: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_samssubs'
)}}

select 
    Employee_ID as employee_key,
    Employee_ID as employee_id,
    Employee_First_Name as fname,
    Employee_Last_name as lname,
    Employee_Birthday as dob
from {{ source('subs_landing', 'employee')}}
```

- Save the file and build the model. Go to Snowflake to see the newly created table! 

#### dim product ####
- Create a new file inside of the Sam's Subs directory called `subs_dim_product.sql`
- Populate the code that we will use in this file below: 


```
{{ config(
    materialized = 'table',
    schema = 'dw_samssubs'
)}}

select 
    Product_ID as product_key,
    Product_ID,
    Product_Name,
    Product_Calories
from {{ source('subs_landing', 'product')}}
```

- Save the file and build the model. Go to Snowflake to see the newly created table!


#### dim store ####
- Create a new file inside of the Sam's Subs directory called `subs_dim_store.sql`
- Populate the code that we will use in this file below: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_samssubs'
)}}

select 
    store_id as store_key,
    store_id,
    Store_Address as street,
    Store_City as city,
    Store_State as state,
    Store_Zip
from {{ source('subs_landing', 'store')}}

```

- Save the file and build the model. Go to Snowflake to see the newly created table!

  #### dim bread ####
- Create a new file inside of the Sam's Subs directory called `subs_dim_bread.sql`
- Populate the code that we will use in this file below:

```
{{ config(
    materialized = 'table',
    schema = 'dw_samssubs'
)}}

select 
    s
from {{ source('subs_landing', 'sandwich')}}

```

- Save the file and build the model. Go to Snowflake to see the newly created table!

#### dim order_method ####
- Create a new file inside of the Sam's Subs directory called `subs_dim_order_method.sql`
- Populate the code that we will use in this file below: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_samssubs'
)}}

select 
    Order_Number as order_method_key,
    Order_Number as order_id,
    Order_Method as order_method_type
from {{ source('subs_landing', 'orders')}}

```

- Save the file and build the model. Go to Snowflake to see the newly created table!

#### fact purchase ####
- Create a new file inside of the Sam's Subs directory called `fact_sales.sql`
- Populate the code that we will use in this file below: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_samssubs'
)}}

select
    p.product_key,
    c.customer_key,
    e.employee_key,
    s.store_key,
    d.date_key,
    ol. Order_Line_Price as unit_price,
    ol.Order_Line_ Qty as qty,
    (ol.Order_Line_ Qty * ol. Order_Line_Price) as dollars_sold
from {{ source('subs_landing', 'order_line') }} ol
inner join {{ source ('subs_landing', 'orders') }} o
    on o.order_id = ol.order_id
left join {{ ref('subs_dim_product') }} p
    on ol.product_id = p.product_id
left join {{ ref('subs_dim_customer') }} c
    on o.customer_id = c.customer_id
left join {{ ref('subs_dim_employee') }} e
    on o.employee_id = e.employee_id
left join {{ ref('subs_dim_store') }} s
    on s.store_id = o.store_id
left join  {{ ref('subs_dim_date') }} d
    on d.date_key = o.order_date

```

- Save the file and build the model. Go to Snowflake to see the newly created table!

#### fact inventory ####
- Create a new file inside of the Sam's Subs directory called `fact_inventory.sql`
- Populate the code that we will use in this file below
- inventory cannot be created yet. Bread dimension is created though: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_samssubs'
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
from {{ source('subs_landing', 'order_line') }} ol
inner join {{ source ('subs_landing', 'orders') }} o
    on o.order_id = ol.order_id
left join {{ ref('subs_dim_product') }} p
    on ol.product_id = p.product_id
left join {{ ref('subs_dim_store') }} s
    on s.store_id = o.store_id
left join  {{ ref('subs_dim_date') }} d
    on d.date_key = o.order_date
left join {{ ref(subs_dim_bread) }} b
     on i.breadkey = b.breadkey

```

- Save the file and build the model. Go to Snowflake to see the newly created table!

#### schema yaml file ####
- Create a new file inside the subs directory called `_schema_subs.yml`
- This file contains metadata about the models you build. Hint: check out the exercise to help you create this file. 
- Populate the code that we will use in this file below: 
```
version: 2

models:
  - name: subs_dim_employee
    description: "Sam's Subs Employee Dimension"
    columns: 
      - name: employee_key
        description: "Dimension Surrogate Key"
        tests:
        - unique
        - not_null
  - name: subs_dim_customer
    description: "Sam's Subs Customer Dimension"
  - name: subs_dim_date 
    description: "Sam's Subs Date Dimension"
  - name: subs_dim_product
    description: "Sam's Subs Product Dimension"
  - name: subs_dim_store
    description: "Store Information Dimension"  
  - name: fact_purchase
    description: "Sam's Subs Sales Fact"
    
```

## Create a semantic layer model (2 points of EC!)
- Create a model that can query from the data warehouse we just built and reference upstream models.

```
{{ config(
    materialized = 'table',
    schema = 'dw_samssubs'
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
from {{ ref('fact_purchase') }} f

left join {{ ref('subs_dim_product') }} p
    on f.product_key = p.product_key

left join {{ ref('subs_dim_customer') }} c
    on f.customer_key = c.customer_key

left join {{ ref('subs_dim_employee') }} e
    on f.employee_key = e.employee_key
    
left join  {{ ref('subs_dim_date') }} d
    on f.date_key = d.date_key

left join  {{ ref('subs_dim_store') }} s
    on f.store_key = s.store_key
```

- In order to view lineage, the dbt power user extension must be installed. Click on the Lineage tab in vscode (down by the terminal on the bottom), if you are inside the sem_claims.sql model, you should be able to see lineage for that model. View the lineage for the other files in the model as well. 
