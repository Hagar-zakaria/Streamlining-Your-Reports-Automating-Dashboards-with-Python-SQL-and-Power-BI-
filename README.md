# Streamlining Your Reports: Automating Dashboards with Python, SQL, and Power BI

As a data analyst, this is one of the most interesting projects you'll work on because you'll see how business metrics change in real-time. In the real world, companies receive new data daily, so dynamic dashboard reports are essential.

## What Are Dynamic Dashboard Reports?

Dynamic dashboard reports are essentially real-time snapshots of data that reflect the latest updates or changes in a database.

For example, let's consider an online store. Yesterday, it received 2500 orders, but today it received 2800 orders. A static dashboard would only display the total number of orders from the previous day (2500). However, a dynamic dashboard report would show the most recent data, indicating that the store received 2800 orders today. It can also illustrate the growth in the number of orders between the previous day and the current day.

Imagine you're viewing such a dashboard in your business intelligence tool. Once you refresh the dashboard page, the dynamic report immediately reflects the most recent changes in the data. This capability ensures that users always have access to the latest information, allowing for better-informed decisions and insights.

![image](https://github.com/Hagar-zakaria/Streamlining-Your-Reports-Automating-Dashboards-with-Python-SQL-and-Power-BI-/assets/93611934/1ab5e3d0-466f-42f4-88ff-d0a4a5abdbf9)

Dynamic dashboard reports are crucial for tracking key business growth metrics such as customer churn/retention rate, revenue growth/decline, and new customer count on various time scales like daily, weekly, monthly, or yearly.

In this project, I'll be creating a dynamic dashboard report using data from a cryptocurrency web API. This API offers data on the revenue generated by multiple cryptocurrency projects each day.

The project aims to analyze revenue growth over time, daily revenue, monitor the active cryptocurrency projects generating revenue daily, and track the 24-hour revenue change.

Here's an example of the report we'll generate daily.

![image](https://github.com/Hagar-zakaria/Streamlining-Your-Reports-Automating-Dashboards-with-Python-SQL-and-Power-BI-/assets/93611934/6e2fe7a5-d2f0-425b-95d4-996e14395f89)

## Tools and Tech Used:

1. Python
2. SQL SERVER DATABASE
3. POWER BI
4. Windows Task Scheduler

## Project Architecture

![image](https://github.com/Hagar-zakaria/Streamlining-Your-Reports-Automating-Dashboards-with-Python-SQL-and-Power-BI-/assets/93611934/cc12ef96-f55a-4202-a85b-ee61a2d00405)

## Step 1

Dynamic dashboard reports are crucial for tracking key business growth metrics such as customer churn/retention rate, revenue growth/decline, and new customer count on various time scales like daily, weekly, monthly, or yearly.

In this project, I'll be creating a dynamic dashboard report using data from a cryptocurrency web API. This API offers data on the revenue generated by multiple cryptocurrency projects each day.

The project aims to analyze revenue growth over time, daily revenue, monitor the active cryptocurrency projects generating revenue daily, and track the 24-hour revenue change.

Here's an example of the report we'll generate daily.


### Importing Needed Libraries

```python
import pandas as pd
from sqlalchemy import create_engine
import sqlalchemy
import os
import requests
from datetime import date
from sqlalchemy import MetaData
from sqlalchemy import Table, Column, Integer, String, Numeric
from sqlalchemy.dialects.mysql import VARCHAR
import pyodbc
```

### Getting Current Date Because it doesn't Come With a Date Column

```python
from datetime import date

now = date.today()
todays_date = now.strftime('%Y/%m/%d')
todays_date
```

### Extracting Data from Ethereum API


```python
eth_api = 'https://api.llama.fi/overview/fees/ethereum?excludeTotalDataChart=true&excludeTotalDataChartBreakdown=true&dataType=dailyRevenue'
params = {'chain':'Ethereum'}
r = requests.get(eth_api,
                params = params)
eth_json = r.json()
eth_df = pd.DataFrame(eth_json['protocols'])
eth_df.head(3)
```

### Creating a Function to Extract Data from Any API Link Provided


```python
def extract_from_api(api_url, chain_name,params):
    params = {'chain': chain_name}
    r = requests.get(api_url,
                    params = params)
    chain_json = r.json()
    chain_df = pd.DataFrame(chain_json['protocols'])
    return chain_df
```

### This Function Transforms the Data and Keeps the Necessary Columns Needed


```python
def transform_data(chain_df, chain_name):
    cols = ['defillamaId', 'name', 'module','category', 'dailyRevenue', 'dailyFees']
    chain_df = chain_df[cols]
    chain_df.insert(4, "CHAIN_NAME", chain_name)
    chain_df.insert(7, "DATE", todays_date)
    return chain_df
```

### This Function Executes the Set of Functions Created Above

```python
def extract_and_transfrom(api_url, chain_name, params):
    chain_df = extract_from_api(api_url, chain_name, params)
    chain_df = transform_data(chain_df, chain_name)
    return chain_df
```

### Running the ETL Functions Created on Each API Link Provided


```python
eth_df = extract_and_transfrom('https://api.llama.fi/overview/fees/ethereum?excludeTotalDataChart=true&excludeTotalDataChartBreakdown=true&dataType=dailyRevenue', 'ETHEREUM', params)

arb_df = extract_and_transfrom('https://api.llama.fi/overview/fees/arbitrum?excludeTotalDataChart=true&excludeTotalDataChartBreakdown=true&dataType=dailyRevenue', 'ARBITRUM', params)

op_df = extract_and_transfrom('https://api.llama.fi/overview/fees/optimism?excludeTotalDataChart=true&excludeTotalDataChartBreakdown=true&dataType=dailyRevenue', 'OPTIMISM', params)

bsc_df = extract_and_transfrom('https://api.llama.fi/overview/fees/BSC?excludeTotalDataChart=true&excludeTotalDataChartBreakdown=true&dataType=dailyRevenue', 'BSC', params)

polygon_df = extract_and_transfrom('https://api.llama.fi/overview/fees/polygon?excludeTotalDataChart=true&excludeTotalDataChartBreakdown=true&dataType=dailyRevenue', 'POLYGON', params)

avalache_df = extract_and_transfrom('https://api.llama.fi/overview/fees/avalanche?excludeTotalDataChart=true&excludeTotalDataChartBreakdown=true&dataType=dailyRevenue', 'AVALANCHE', params)

base_df = extract_and_transfrom('https://api.llama.fi/overview/fees/base?excludeTotalDataChart=true&excludeTotalDataChartBreakdown=true&dataType=dailyRevenue', 'BASE', params)

solana_df = extract_and_transfrom('https://api.llama.fi/overview/fees/solana?excludeTotalDataChart=true&excludeTotalDataChartBreakdown=true&dataType=dailyRevenue', 'SOLANA', params)

cronos_df = extract_and_transfrom('https://api.llama.fi/overview/fees/cronos?excludeTotalDataChart=true&excludeTotalDataChartBreakdown=true&dataType=dailyRevenue', 'CRONOS', params)
```

### Putting All Pandas Dataframes into a List

```python
chain_df_list = [eth_df,
                arb_df,
                op_df,
                bsc_df,
                polygon_df,
                base_df,
                 avalache_df,
                solana_df,
                cronos_df]

```

### Importing SQLAlchemy Libraries

```python
from sqlalchemy import MetaData
from sqlalchemy import Table, Column, Integer, String, Numeric
from sqlalchemy.dialects.mysql import VARCHAR
import pyodbc
```

### Connecting with SQL Server Database Using SQLAlchemy


```python
SERVER = os.environ.get('MS SQL SERVER NAME')
DRIVER = os.environ.get('MS SQL SERVER DRIVER')
database_name = 'defi_db'
SQL_SERVER_CONNECTION = sqlalchemy.create_engine(f'mssql://{SERVER}/{database_name}?driver={DRIVER}')


SERVER = os.environ.get('MS SQL SERVER NAME')
DRIVER = os.environ.get('MS SQL SERVER DRIVER')
database_name = 'defi_db'

cnxn_str = (f"Driver={DRIVER};"
            f"Server= {SERVER};"
            "Database=defi_db;"
            "Trusted_Connection=yes;")
cnxn = pyodbc.connect(cnxn_str)
cursor = cnxn.cursor()
```

### This Function Loads the Data Extracted from the API into a SQL Server Database


```python
def concatenate_and_load(chain_list):
    final_df = pd.concat(chain_list)
    final_df.to_sql('staging_defi_revenue_details', SQL_SERVER_CONNECTION, 
    if_exists = 'replace', index = False,
                   dtype = {'defillamaId': sqlalchemy.types.INTEGER(),
                           'name': sqlalchemy.types.String(50),
                           'module': sqlalchemy.types.String(50),
                           'category': sqlalchemy.types.String(50),
                           'CHAIN_NAME': sqlalchemy.types.String(50),
                           'dailyRevenue': sqlalchemy.types.Numeric(10,2),
                           'dailyFees':sqlalchemy.types.Numeric(10,2),
                           'DATE': sqlalchemy.types.Date()})
```

### Removing Duplicates from the Staging Table Before Inserting Into Data Warehouse Table



```python
    cursor.execute('with check_for_duplicate_data as (\
                    select defillamaId, name, module, category,\
                    CHAIN_NAME, dailyRevenue, dailyFees, DATE,\
                    row_number() over (partition by defillamaId,\
                    name, module, category, CHAIN_NAME, dailyRevenue, dailyFees, DATE\
                    order by defillamaId, name, module, category,\
                    CHAIN_NAME, dailyRevenue, dailyFees, DATE) as duplicate_count\
                    from [dbo].[staging_defi_revenue_details])\
                    delete from check_for_duplicate_data \
                    where duplicate_count > 1')

```

### Inserting Into Data Warehouse Table

```python
    cursor.execute('INSERT INTO dbo.DW_DEFI_REVENUE_TABLE 
    (Defi_lama_ID, Dapp_name, Module, Category, Chain_name, Daily_Revenue, 
     Daily_Fees, Date) select * from dbo.staging_defi_revenue_details')

```

### Commiting the Execution

```python
    cnxn.commit()
    print('data successfully loaded into SQL SERVER DATABASE')

```

### Loading the Table Into the Database


```python
concatenate_and_load(chain_df_list)
```


### Closing and Disposing Connection to the Database


```python
SQL_SERVER_CONNECTION.dispose()

cnxn.close()
```



## Step 2

Automate the python script using either crontab or Windows Scheduler. In my case, I’m using Windows Scheduler, and as seen below, the script runs every day by 6:20 AM.

![image](https://github.com/Hagar-zakaria/Streamlining-Your-Reports-Automating-Dashboards-with-Python-SQL-and-Power-BI-/assets/93611934/de5a138b-7fa8-4c2f-905d-80059c0086b0)


## Step 3

Create dynamic SQL views to track important insights and then convert these views to appropriate visualizations using power bi or any BI tool.

1.24HR TOTAL REVENUE

```python
    CREATE VIEW TOTAL_24HOUR_REVENUE
    AS
    --CREATING A VIEW TO SHOW TOTAL REVENUE
    select round(SUM(Daily_Revenue),0) AS TOTAL_REVENUE
    from DW_DEFI_REVENUE_TABLE
    --WHERE CLAUSE ENSURES WE ARE ONLY GETTING DATA THE CURRENT DATE
    WHERE Date = cast(getdate()  as date)
```


Below is the result when converted to a power bi card visualization.

![image](https://github.com/Hagar-zakaria/Streamlining-Your-Reports-Automating-Dashboards-with-Python-SQL-and-Power-BI-/assets/93611934/fa40ac94-bb36-4458-8916-a33a438c1cc7)

2. 24 HOUR TOTAL REVENUE OF TOP 5 BLOCKCHAINS

```python
    --CREATING A VIEW TO SHOW THE TOP 5 BLOCKCHAINS BY REVENUE GENERATED 
   --WITHIN THE LAST 24 HOURS.
   CREATE VIEW TOP_5_HIGHEST_EARNING_BLOCKCHAINS_LAST_24HRS
   AS 
   SELECT top (5) Chain_name, 
   round(SUM(Daily_Revenue),0) AS TOTAL_DAILY_REVENUE
   FROM DW_DEFI_REVENUE_TABLE
   WHERE Date = cast(getdate()  as date)
   GROUP BY Chain_name
   ORDER BY SUM(Daily_Revenue) DESC
```


Below is the result when converted to a power bi bar chart.

![image](https://github.com/Hagar-zakaria/Streamlining-Your-Reports-Automating-Dashboards-with-Python-SQL-and-Power-BI-/assets/93611934/3d6e74a5-686e-479d-b8b4-82412493eae3)


3. 7 DAY REVENUE CHART
   
```python
    -- CREATING A VIEW TO CALCULATE GROWTH IN TOTAL REVENUE 
    -- OVER A 7 DAY PERIOD.
    CREATE VIEW TOTAL_DAILY_REVENUE_7_DAYS AS 
    select Date, sum(Daily_Revenue) as TOTAL_DAILY_REVENUE
    from DW_DEFI_REVENUE_TABLE
    where Date between
    cast(getdate() - 7 as date) and
    cast(getdate() as date)
    group by Date
```

Below is the result when converted to a power bi line chart.

![image](https://github.com/Hagar-zakaria/Streamlining-Your-Reports-Automating-Dashboards-with-Python-SQL-and-Power-BI-/assets/93611934/49506d7d-a1ca-4007-8718-96e8959ce99a)


4. TOTAL NUMBER OF ACTIVE DECENTRALIZED APPS WITHIN THE LAST 24HOURS

```python
   --AN DAPP IS CONSIDERED ACTIVE IF IT HAS GENERATED ANY FORM OF 
  --REVENUE WITHIN 24 HOURS.
  CREATE VIEW TOTAL_NUMBER_OF_ACTIVE_DEFI_PLATFORMS
  AS 
  WITH TOTAL_NUMBER_OF_ACTIVE_DEFI_PLATFORMS AS (
  select DISTINCT Module, SUM(Daily_Revenue) AS TOTAL_REVENUE,
  SUM(Daily_Fees) AS TOTAL_FEES
  from DW_DEFI_REVENUE_TABLE
  where Date = CAST(GETDATE() AS DATE)
  and Module not like Chain_name
  AND (Daily_Revenue IS NOT NULL
  OR Daily_Fees IS NOT NULL)
  GROUP BY Module
  HAVING (SUM(Daily_Revenue) != 0
  OR SUM(Daily_Fees) != 0))
  SELECT COUNT(*) AS ACTIVE_DEFI_PLATFROM_COUNT
  FROM TOTAL_NUMBER_OF_ACTIVE_DEFI_PLATFORMS
```

![image](https://github.com/Hagar-zakaria/Streamlining-Your-Reports-Automating-Dashboards-with-Python-SQL-and-Power-BI-/assets/93611934/eae306bb-699b-4b91-9758-bad318fdb912)

5. 24 HOUR REVENUE CHANGE

```python
--BELOW VIEW TRACKS 24 HOUR REVENUE CHANGE
 CREATE VIEW _24HR_REVENUE_CHANGE AS
 with todays_revenue as (
 SELECT Chain_name, sum(Daily_Revenue) as revenue_today
 FROM DW_DEFI_REVENUE_TABLE
 where date = cast(getdate() as date)
 group by Chain_name),
 yesterdays_revenue as (
 select Chain_name, sum(Daily_Revenue) as revenue_yesterday
 FROM DW_DEFI_REVENUE_TABLE
 where date = cast(getdate() - 1 as date)
 group by Chain_name)
 select tr.Chain_name, sum(ry.revenue_yesterday) AS YESTERDAYS_REVENUE,
 sum(tr.revenue_today) AS TODAYS_REVENUE,
 ((sum(tr.revenue_today) - sum(ry.revenue_yesterday)) / sum(ry.revenue_yesterday))
 AS PERCENTAGE_CHANGE_IN_REVENUE_24HRS
 from todays_revenue tr
 inner join yesterdays_revenue ry
 on tr.Chain_name = ry.Chain_name
 group by tr.Chain_name
```

![image](https://github.com/Hagar-zakaria/Streamlining-Your-Reports-Automating-Dashboards-with-Python-SQL-and-Power-BI-/assets/93611934/9a5b37b2-04d8-4885-aef1-4f6532572192)

FINAL REPORT SHOWING ALL THE IMPORTANT INSIGHTS IN ONE DASHBOARD.


![image](https://github.com/Hagar-zakaria/Streamlining-Your-Reports-Automating-Dashboards-with-Python-SQL-and-Power-BI-/assets/93611934/98523438-8f3f-4c90-8a65-fbfc8fc2812b)


Key Insights Summary

1. Ethereum Dominates Revenue Stability: Over the past 7 days, Ethereum consistently leads in daily revenue generation. Its revenue stability underscores the solidity of its ecosystem.
2. BASE Chain's Revenue Decline: Notably, Friend Tech, a decentralized app on the BASE chain, experienced a notable decline in revenue recently.
3. Polygon's Revenue Fluctuation: Polygon witnessed a substantial surge in revenue, followed by a sharp decline within 24 hours. The majority of this revenue surge stemmed from Metamask, a widely-used decentralized wallet for storing cryptocurrencies.
