# About the Project
Perform some optimization tasks and best practices in SQL querry.

**Table of contents**
- [About the Project](#about-the-project)
- [Language](#language)
- [Library](#library)
- [Getting started](#getting-started)
- [Rum Project](#rum-project)
- [References](#references)
- [Author](#author)

## Language
- Python  3.9.12
- SQL

## Library
- Pandas  2.0.1
- re      
- Sqlite  

## Getting started
- [The Database](#the-database)
- [Import the library](#import-the-library)
- [Replace ‘LIKE’ clauses](#replace-like-clauses)
- [Replace ‘Case-when Like’](#replace-case-when-like)
- [Use temporary table](#use-temporary-table)
- [JOINs order](#joins-order)
- [GROUP BY](#group-by)
- [Use simple equi-joins](#use-simple-equi-joins)
- [Avoid subqueries in WHERE clause](#avoid-subqueries-in-where-clause)
- [Use Max instead of Rank](#use-max-instead-of-rank)

### The Database
The database consist of four tables and sixteen fields with fake informations about customers and address as show below, to support the SQL query optimization.
If you want run the project, it´s possible to use the [database](https://1drv.ms/u/s!AtuOpzYtzJ15hetWXLw1f1o-8GFhHg?e=JD6Tfk) that have 210MB and 1M of records or create your own database with fake data [here](https://github.com/IgorTraspadini/Sintectical_SQL_database).  
```
The tables on the database
    Table name
0   Address
1      City
2   Country
3  Customer

The tables structure"
Customer
cid       name     type  notnull dflt_value  pk
0  customer_id  INTEGER        1       None   1
1         Name     TEXT        0       None   0
2   Address_id  INTEGER        0       None   0
3        Email     TEXT        0       None   0
4        Phone     TEXT        0       None   0
5          Job     TEXT        0       None   0
6      Company     TEXT        0       None   0

Address
cid      name     type  notnull dflt_value  pk
0  Address_id  INTEGER        1       None   1
1     Address     TEXT        0       None   0
2     Zipcode     TEXT        0       None   0
3     City_id  INTEGER        0       None   0
4        City     TEXT        0       None   0
5  Country_id  INTEGER        0       None   0
6     Country     TEXT        0       None   0

City
cid   name     type  notnull dflt_value  pk
0  City_id  INTEGER        1       None   1
1     City     TEXT        0       None   0

Country
cid      name     type  notnull dflt_value  pk
0  Country_id  INTEGER        1       None   1
1     Country     TEXT        0       None   0
```
Sample of the database contents
Customer Table
|customer_id|              Name|Address_id|                      Email|  Phone           |   Job                     |  Company         |
|:---------:|:----------------:|:--------:|:-------------------------:|:----------------:|:-------------------------:|:----------------:| 
|          0|     Kevin Sanders|       100| christinamills@example.com| 260-962-0765x7932|Loss adjuster, chartered   | Jordan Inc       |
|          1|      Justin Patel|       101|  victoriahogan@example.org|    3637991023    | Engineer, aeronautical    |Harris, White & Mc| 
|          2|Matthew Hughes Jr.|       102|  jonesadrienne@example.com| (742)425-3943x211| Further education lecturer|Hubbard-Navarro   |

Address Table
|Address_id|                                          Address  | Zipcode| City_id | Country_id |  
|:--------:|:-------------------------------------------------:|:------:|:-------:|:----------:|
|       100|           687 Denise Creek\nJohnsonmouth, NH 48492|   49306|        0|           0| 
|       101|  6159 Lindsey Islands Suite 946\nHughestown, MN...|   95586|        1|           1|
|       102|                           USS Howard\nFPO AE 68571|   35641|        2|           2|

City Table                            
|City_id|        City      |          
|:-----:|:----------------:|           
|      0|   New Derricktown|          
|      1|      Rayborough  |          
|      2| North Andrewbury |          

Country Table
|Country_id|     Country      |
|:--------:|:----------------:|
|         0|Puerto Rico       |
|         1|Russian Federation|
|         2|  Ukraine         |

### Import the library
```python
import pandas as pd
import sqlite3

# create a connection and cursor to the database
conn = sqlite3.connect('db_test.db')
cursor = conn.cursor()
```

### Replace ‘LIKE’ clauses
Use ‘regexp_like’ to replace ‘LIKE’ clauses

❌
```SQL
SELECT *
FROM Customer
JOIN Address ON Customer.Address_id = Address.Address_id
LEFT JOIN City ON City.City_id = Address.City_id
LEFT JOIN Country ON Country.Country_id = Address.Address_id
WHERE lower(Country) LIKE '%bela%' OR
      lower(Country) LIKE '%bra%'  OR
      lower(Country) LIKE '%uk%'   OR
      lower(Country) LIKE '%uru%'  OR
      lower(Country) LIKE '%ger%'  OR
      lower(Country) LIKE '%afr%'  OR
      lower(Country) LIKE '%ven%'
```
✔️
```SQL
SELECT * 
FROM Customer
JOIN Address ON Customer.Address_id = Address.Address_id
LEFT JOIN City ON City.City_id = Address.City_id
LEFT JOIN Country ON Country.Country_id = Address.Address_id
WHERE REGEXP_LIKE('bela|bra|uk|uru|ger|afr|ven',lower(Country))
```
Benefits:
<p>✅ More concise code</p>
<p>✅ Better performance (depends on the database structure and the function regex)</p> 

### Replace ‘Case-when Like’
Use ‘regexp_extract’ to replace ‘Case-when Like’

❌
```SQL
SELECT CASE WHEN lower(Country) LIKE '%belarus%' THEN 'belarus'
            WHEN lower(Country) LIKE '%brazil%'  THEN 'brazil'
            WHEN lower(Country) LIKE '%uruguay%' THEN 'uruguay'
            WHEN lower(Country) LIKE '%germany%' THEN 'germany'
            WHEN lower(Country) LIKE '%nauru%'   THEN 'nauru'
            WHEN lower(Country) LIKE '%venezuela%' THEN 'venezuela' ELSE '' END AS Countr_
FROM Customer
JOIN Address ON Customer.Address_id = Address.Address_id
LEFT JOIN City ON City.City_id = Address.City_id
LEFT JOIN Country ON Country.Country_id = Address.Address_id
```
✔️
```SQL
SELECT REGEXP_CASE('belarus|brazil|uruguay|germany|nauru|venezuela',lower(Country))
FROM Customer
JOIN Address ON Customer.Address_id = Address.Address_id
LEFT JOIN City ON City.City_id = Address.City_id
LEFT JOIN Country ON Country.Country_id = Address.Address_id
```
Benefits:
<p>✅ More concise code</p>
<p>✅ Better performance (depends on the database structure and the function regex)</p> 

### Use temporary table
Convert long list of IN clause into a temporary table

### JOINs order
Order the JOINs from largest tables to smallest tables

### GROUP BY
Always "GROUP BY" by the attribute/column with the largest number of unique entities/values

### Use simple equi-joins
Two tables with date string e.g., ‘2020-09-01’, but one of the tables only has columns for year, month, day values

### Avoid subqueries in WHERE clause

### Use Max instead of Rank

## Rum Project
```bash
# Clone the reposotiry 
git clone https://github.com/IgorTraspadini/SQL_Optimization.git

# Import
requirements.txt

# Run the project
python xxxxxxxxxxxx.py
```

## References 
- [SQL](https://pt.wikipedia.org/wiki/SQL)
- [Regular Expression](https://docs.python.org/3/library/re.html)


## Author
[Igor Traspadini](https://www.linkedin.com/in/igor-chieppe-traspadini/?locale=en_US)
