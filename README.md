# Analyzing Business Data and Building a Comprehensive Sales Report in Power BI - by Brian Hornick
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
3. What products haven't sold in the longest time?
   
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
Average Profit Margin (%) = AVERAGEX('Order Items', DIVIDE([Profit], [Revenue]))
```
### Step 10: Creating the Executive Sales View

Now that we have our measures it's time to design the first page. I have included a screenshot below for reference:

![image_alt](https://github.com/brianhornick/PowerBI-Bicycle-Business-Project/blob/main/Image/Screenshot%202025-03-26%20151734.png?raw=true)

Starting from the top, I created a heading bar that displays the company, what this dashboard is and page navigation to assist users with page navigation (and I think it looks sleek). This bar will display across all pages.

Below that, I have inserted a card bar that shows revenue, orders, unit cost, and profit as well as the KPI visual that can compare this quarter's revenue to last quarter's. On the far right, there is a slicer that can allow filtering by any select year, quarter and/or month. 

Next, I added 4 line graphs that show how these key metrics have changed over time. I added the date heirachy to the X axis so users can drill up, drill down or expand down to view these metrics by different time frames. I also added in a button slicer that switches all the line charts to bar graphs with data labels as shown below:

![image_alt](https://github.com/brianhornick/PowerBI-Bicycle-Business-Project/blob/main/Image/Screenshot%202025-03-26%20152528.png?raw=true)

I defaulted some of the drill features differently so that the charts could answer questions such as, "What months are revenue the highest?" without the user needing to drill.

I added some design and colour formatting accross all pages such as light colour backgrounds to help the visuals pop but not create too much contrast that it hurts the eyes. I also rounded the corners of the visual panes and added line bars to separate the visuals.

### Step 11: Creating the Late Orders % Measure

It is now time to move to our inventory view. The key metric here is late orders % as orders must be placed well enough in advance in order for customers to receive them on time. As the last year in this dataset is 2020, COVID has just hit making shipping times take longer. To calculate this here is the formula I used:

```
Late Orders % = 
VAR TotalOrders = 
    CALCULATE(
        COUNT(Orders[Order Date]),
        USERELATIONSHIP(Orders[Shipped Date], DateTable[Date])
    )

VAR LateOrders = 
    CALCULATE(
        COUNT(Orders[Order Date]),
        Orders[Shipped Date] > Orders[Required Date],
        USERELATIONSHIP(Orders[Shipped Date], DateTable[Date])
    )

RETURN 
    DIVIDE(LateOrders, TotalOrders, 0)
```

This measure is a little more complex than the others so far for a couple of reasons. One is that we must create 2 variables, "TotalOrders" and "LateOrders" and then divide them to calculate this.
The other is that 'Order Date', not 'Shipped Date' has an active relationship with the date table; however rather than create another date table, we can use the "USERELATIONSHIP" function.

First, we count all the orders in the 'TotalOrders' variable, then count all the orders where the date the products shipped was later than the required date ('LateOrders), and then divide the late orders by the total orders to get the 'Late Orders %'.

### Step 12: Designing The Inventory View

![image_alt](https://github.com/brianhornick/PowerBI-Bicycle-Business-Project/blob/main/Image/Screenshot%202025-03-27%20185728.png?raw=true)

In this page, starting from the top left, I included a bar graph that displays the top 10 products ordered by quantity. As the product names are quite long, rather than include them in the X Axis, I simply use the X Axis title to let users know they can hover over the graph to see the product name. Conditional formatting is added so that items low on stock show up as red. I determined low on stock as having less quantity in stock than the amount that product has been ordered in the past 2 years. (In the real-world, this may more be like the last few months; I just wanted the graph to show some red bars).

To the right, I added a bar graph that shows late order percentage by time and drill/expand down is enabled to see months and quarter metrics. I then added a filter to the right to enable seeing these values by certain years, quarters, and/or months and then added a card below to clearly display 'Late Order %'.

The scatter chart at the bottom shows a view of all products, comparing their in stock numbers to the quantity ordered in the past 2 years. The same conditional formatting is used to show items that are low on stock.

### Step 13: Designing the Customer & Sales View

![image_alt](https://github.com/brianhornick/PowerBI-Bicycle-Business-Project/blob/main/Image/Screenshot%202025-03-26%20172528.png?raw=true)

This dashboard functions similarly to the executive sales view, but instead of showing general metrics over time, it's designed to show metrics by store, customer and staff. I used the same card and filter bar at the top as the executive view so that various metrics can be displayed and users can filter by different time frames. 

Below that, starting to the right I added an area chart which shows revenue by store. Left of that I added 2 bar graphs which show sales by staff and the top 10 customers.

### Step 14: Adding a Last Sale Date Measure

In order to find which products have gone the longest without a sale we need a way to calculate the last date of sale. Using this formula, we can do just that:

```
Last Sale Date = 
    CALCULATE(
            MAX(Orders[Order Date]),
            FILTER('Order Items', 'Order Items'[Product Id] = [Product ID])
        )
```
This calculation uses the MAX function to find the most recent order date. Then this calculation is filtered by rows where the Product ID in the 'Order Items' table matches with the Product Id in question.

### Step 15: Designing the Brand & Product View


