# Project #3 Instructions #
## Objective ##

new change
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
    - Host: `https://etb90388.snowflakecomputing.com` 
    - Role: `TRAINING_ROLE` 
    - Warehouse: `group2_WH` 
    - Database: `group2project` 
    - Schema: `samssubs` 
    - Username: 
    - Authorization Method: `Username and Password`
    - Password: 
    - Click `Set up destination`
TRA- Once the connection test passes, it will pull up the new connection window
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
    database: group2project
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
    Customer_Phone
    
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
from {{ source('subs_landing', 'Employee')}}
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
from {{ source('subs_landing', 'Product')}}
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
    Store_ID as store_key,
    Store_ID as store_id,
    Store_Address as street,
    Store_City as city,
    Store_State as state,
    Store_Zip as zip
from {{ source('subs_landing', 'Store')}}

```

- Save the file and build the model. Go to Snowflake to see the newly created table!

  #### dim bread ####
- Create a new file inside of the Sam's Subs directory called `subs_dim_bread.sql`
- SQL code generated with ChatGPT
- Populate the code that we will use in this file below:

```
{{ config(
    materialized = 'table',
    schema = 'dw_samssubs'
) }}

with bread_types as (

    select distinct
        Sandwich_Bread_Type
    from {{ source('subs_landing', 'Sandwich') }}
    where Sandwich_Bread_Type is not null

),

bread_type_keys as (

    select
        row_number() over (order by Sandwich_Bread_Type) as bread_type_key,  -- Surrogate key for each row
        
        -- Assigning logical keys to each bread type
        case 
            when Sandwich_Bread_Type = 'white' then 1
            when Sandwich_Bread_Type = 'sourdough' then 2
            when Sandwich_Bread_Type = 'lettuce wrap' then 3
            when Sandwich_Bread_Type = 'wheat' then 4
            else null
        end as bread_id,

        Sandwich_Bread_Type as bread_type
    from bread_types

)

select * from bread_type_keys

```

- Save the file and build the model. Go to Snowflake to see the newly created table!

#### dim order_method ####
- Create a new file inside of the Sam's Subs directory called `subs_dim_order_method.sql`
- SQL generated with ChatGPT 
- Populate the code that we will use in this file below: 
```
{{ config(
    materialized = 'table',
    schema = 'dw_samssubs'
) }}

with order_method_mappings as (

    select
        -- Surrogate key generated using row_number()
        row_number() over (order by order_method_type) as order_method_id,
        
        -- Logical key for each order method
        case
            when order_method_type = 'Phone' then 1
            when order_method_type = 'Online' then 2
            when order_method_type = 'In-Person' then 3
            else null
        end as order_method_key,
        
        order_method_type

    from (
        -- Define distinct order methods here
        select 'Phone' as order_method_type
        union all
        select 'Online'
        union all
        select 'In-Person'
    ) as base_order_methods

)

select * from order_method_mappings

```

- Save the file and build the model. Go to Snowflake to see the newly created table!

#### fact purchase ####
- Create a new file inside of the Sam's Subs directory called `fact_purchase.sql`
- Used ChatGPT to debug.
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
    om.order_method_key,
    ol. Order_Line_Price unit_price,
    ol.Order_Line_Qty qty,
    (ol.Order_Line_Qty * ol. Order_Line_Price) dollars_sold
from {{ source('subs_landing', 'Order_Line') }} ol
inner join {{ source ('subs_landing', 'Orders') }} o
    on o.Order_Number = ol.Order_Number
left join {{ ref('subs_dim_product') }} p
    on ol.Product_ID = p.product_id
left join {{ ref('subs_dim_customer') }} c
    on o.Customer_ID = c.customer_id
left join {{ ref('subs_dim_employee') }} e
    on o.Employee_ID = e.employee_id
left join {{ ref('subs_dim_store') }} s
    on s.store_id = o.Store_ID
left join  {{ ref('subs_dim_date') }} d
    on d.date_key = o.Order_Date
left join  {{ ref('subs_dim_order_method')}} om
    on o.Order_Method = om.order_method_type

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
    qty,
    unit_price,
    qty * unit_price as inventory_cost
from <inventory_data_table>
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
- Generated based on my starting labels in ChatGPT
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
    columns:
      - name: customer_key
        description: "Dimension Surrogate Key"

  - name: subs_dim_date 
    description: "Sam's Subs Date Dimension"
    columns:
      - name: date_key
        description: "Dimension Surrogate Key"

  - name: subs_dim_product
    description: "Sam's Subs Product Dimension"
    columns:
      - name: product_key
        description: "Dimension Surrogate Key"

  - name: subs_dim_store
    description: "Store Information Dimension"
    columns:
      - name: store_key
        description: "Dimension Surrogate Key"

  - name: fact_purchase
    description: "Sam's Subs Purchases Fact"
    columns:
      - name: purchase_id
        description: "Fact Table Surrogate Key"

  - name: subs_dim_bread
    description: "Inventory Fact Bread Type Dimension"
    columns:
      - name: bread_key
        description: "Dimension Surrogate Key"

  - name: subs_dim_order_method
    description: "Sam's Subs Order Method Description Dimension"
    columns:
      - name: order_method_key
        description: "Dimension Surrogate Key"

    
```

## Create a semantic layer model (2 points of EC!)
- Create a model that can query from the data warehouse we just built and reference upstream models.

```
{{{ config(
    materialized = 'table',
    schema = 'dw_samssubs'
)}}

select
    c.fname as cust_fname,
    c.lname as cust_lname,
    e.fname as employee_fname,
    e.lname as employee_lname,
    concat(s.street,',', s.city) as store,
    om.order_method_type,
    p.product_calories,
    p.product_name,
    d.date_id as date_of_purchase,
    f.qty as quantity_purchased,
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

left join {{ ref('subs_dim_order_method') }} om
    on f.order_method_key = om.order_method_key
```

- In order to view lineage, the dbt power user extension must be installed. Click on the Lineage tab in vscode (down by the terminal on the bottom), if you are inside the sem_claims.sql model, you should be able to see lineage for that model. View the lineage for the other files in the model as well. 
