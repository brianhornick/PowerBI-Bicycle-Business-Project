# Analyzing Business Data and Building a Report in Power BI - by Brian Hornick
## Problem Statement
A bicycle chain company, SaveOnBikes, owns 3 stores in the United States and the owner needs a visual report to make better data-driven decisions. This dataset was found on Kaggle.com LINK: (https://www.kaggle.com/datasets/dillonmyrick/bike-store-sample-database). I used ChatGPT to add some extra sales data up until 2021 and modified some of the values to more accurately resemble a business (changing average delay times for shipping to vary by make). I also added a unit cost column to calculate measures like profit and profit margin. This dataset will serve as a base, which we will use to design our report. I will be outlining each step taken in creating this dashboard and the end goal is to complete a sharp-looking, easy-to-use dashboard that can answer a variety of key questions with ease. These questions include:

General Sales:

1. Can total revenue, orders, unit cost and profit be clearly shown and easily filtered by time period?
2. What months of the year are sales the highest?
3. How does this quarter's revenue compare to last quarter?
4. How has profit and profit margin changed over time?

Inventory:

1. Is inventory high enough to meet demand for the top products?
2. What products are in high demand recently?
3. What percentage of orders are arriving after the required date, how has COVID impacted this?

Customers & Sales:

1. Who are our top customers?
2. Which stores are seeing the largest revenue growth?
3. Which staff members are contributing the most to sales?

Brand and Products:
1. What is the top selling brand, has this changed throughout the years?
2. Are customers more often paying more for the newer or settling for the older products?
3. What is the percentage of sales lost by price increases?
4. What do the products are most often purchased together?
   
To answer these questions, along with many others, the first step required is to import the dataset into Power BI.

### Step 1: Importing the Dataset

This step is straightforward as all of the data is contained in separate CSV files. Simply by clicking "Get Data", clicking on CSV and loading each of the tables in, keeping everything else the same, we now have our 9 tables loaded in.

### Step 2: Data Model

Now it's time to set up relationships between the tables and build the model. In the "Data Load" section of the "Current File" settings, "Autodetect new relationships after data is loaded" is checked so Power BI should be able to do most of this work itself. After importing the CSVs, here is what the model has defaulted to:

![image_alt](https://github.com/brianhornick/PowerBI-Bicycle-Business-Project/blob/main/Image/Screenshot%202025-03-04%20132140.png?raw=true)

The data model looks good, all relationships are one to many, with the dimension tables flowing nicely to the two fact tables, "Orders" and "Order_items". This forms a nice 'Snowflake Schema' with the nuance being that the "Stock" table is tied to both products and "Stores". It is possible to merge the "Brands" and "Categories" tables with the "Products" table to create more of a 'Star Schema' but for this project, this should work fine.

### Step 3: Data Cleaning

Utilizing the view features of "Column Quality", "Column Distribution" and "Column Profile" in the Power Query Editor, as well as ensuring to change "Column profiling based on top 1000 rows" to "based on entire dataset" allows checking for missing values, errors and duplicates. Using these tools, the dataset looks good, with no errors and the Primary Keys all have the same number of distinct and unique values. 

For nulls however, the "shipped_date" column does contain some null values, likely for some orders that did not go through and therefore did not ship. These did not show up in the Column Quality view as they were set as "NULL" not "null". As I want to set this column as a 'Date' type column I will simply use the 'Replace Values' feature to change these "NULLS" to blanks, and I will ensure to factor them in when using this column for any time-intelligence functions. The "phone" column in the "Customers" table (which contains customer phone numbers) also contains lots of the same "NULLS", however I will filter this column out using 'Remove Columns' as I don't plan on using it in this dashboard. Lastly, the "Staff" table has a null for the "manager_id" column for "Fabiola Jackson" who is the owner of this bike chain, so there is no one she reports to. I will change this "NULL" to blank as well.

### Step 4: Creating a Date Table

To perform these time-intelligence functions required, as well as help ensure consistency with the way we use our dates, a 10th table will be created, which will be a standardized date table. 

To create this table, "New Table" is selected from the home menu and this DAX expression is used:

```
DateTable = CALENDARAUTO()
```

Using the 'CALENDARAUTO() function, a "Date Table" has been created which stretches from the earliest date to the latest date in our dataset. Now this table must be worked into the model, so a relationship will be created to all 3 of the date-related columns in the "Orders" table. The active relationship being to the "order_date" column and 2 inactive relationships set to the "required_date" column and the "shipped_date" column. All these relationships are One-to-Many, from the "DateTable" table to the "Orders" table.

### Step 5: Cleaning Up Column Headers

Before we start creating any measures or visuals, the last thing that would improve our model is to change the column names from lowercase, underscore type (i.e product_name) to proper, spaced type (i.e Product Name). This dataset was originally made in SQL, which is likely why it has this format. Using this M Code expression, this transformation can be done easily:

```
= Table.TransformColumnNames(#"Changed Type", each Text.Proper(Text.Replace(_, "_", " ")))
```
The (#"Changed Type") part will just be changed to whatever the last transformation was, luckily the query editor automatically adds this part in anyways when you create a new step in the editor. Table.TransformColumnNames as well as Text.Proper and Text.Replace will ensure the column names are changed to proper case, replacing the underscores with spaces. "each" is used to select all the column names.

I also changed Order_items table to Order Items

### Step 6: Creating a Revenue Measure

To create a revenue measure, three columns must be factored in: "List Price," "Quantity," and "Discount." These are all found in the "Order Items" table. Using the SUMX function that can calculate this iterated for each row, the measure can be created like this:

```
Revenue = SUMX('Order Items', 'Order Items'[List Price] * 'Order Items'[Quantity] * (1 - 'Order Items'[Discount]))
```

The discount is the percentage the customer paid less than the "List Price," so multiplying "List Price" times "Quantity" times 1 minus the "Discount" will give the revenue for each row.

### Step 7: Creating This Quarter and Previous Quarter Measures

To create a KPI visualization that compares a selected quarter's revenue to the previous quarter's revenue, we must create 2 measures that calculate this quarter's revenue and the previous quarter's revenue, respectively. This can be done using the DAX functions "DATESQTD" and "PREVIOUSQUARTER," as shown below:
```
This Quarter Rev = CALCULATE([Revenue], DATESQTD('DateTable'[Date]))
```
```
PreviousQuarter = CALCULATE([Revenue], PREVIOUSQUARTER('DateTable'[Date]))
```
### Step 8: Creating Profit Measure

To create this measure, we simply need to subtract the unit cost from the revenue. I used this calculation to accomplish this:

```
Profit = [Revenue] - SUMX('Order Items', 'Order Items'[Unit Cost])
```
### Step 9: Creating the Profit Margin Measure

As each transaction has a different profit margin as unit cost varies by product and time of purchase (I modified the unit cost for each product to jump by small increments every 1000 transactions, to reflect inflation), we must calculate the average profit margin, iterated for each transaction. I used the formula below to do this:

```
Average Profit Margin (%) = AVERAGEX('Order Items', DIVIDE([Profit], [Revenue]) * 100)
```
### Step 10: Creating the Executive Sales View

Now that we have our measures it's time to design the first page. I have included a screenshot below for reference:

![image_alt](https://github.com/brianhornick/PowerBI-Bicycle-Business-Project/blob/main/Image/Screenshot%202025-03-26%20151734.png?raw=true)
