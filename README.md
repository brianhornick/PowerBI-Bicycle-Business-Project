# Analyzing Business Data and Building a Dashboard in Power BI - by Brian Hornick
## Problem Statement
A bicycle chain company currently owns 3 stores in the United States and needs a dashboard to make better data-driven decisions. This dataset was found on Kaggle.com LINK: (https://www.kaggle.com/datasets/dillonmyrick/bike-store-sample-database). This dataset will serve as a base, which we will use to design our dashboard. I will be outlining each step taken in creating this dashboard and the end goal is to complete a sharp-looking, easy-to-use dashboard that can answer a variety of key questions with ease. Some of these questions include:

1. How does revenue differ by store?
2. What products sell the most, how does model year impact sales?
3. How does sales differ by brand?
4. What months of the year are sales the highest? Is the stock high enough to meet demand?
5. How soon on average are orders being shipped before they are required? How often are orders being placed too late?

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
DateTable = CALENDAR(DATE(2016, 1, 1), DATE(2018, 12, 28))
```

This DateTable spans from the earliest date (Jan 1, 2016) across the three date-related columns to most recent date (Dec 28, 2018). Now this table must be worked into the model, so a relationship will be created to all 3 of the date-related columns in the "Orders" table. The active relationship being to the "order_date" column and 2 inactive relationships set to the "required_date" column and the "shipped_date" column. All these relationships are One-to-Many, from the "DateTable" table to the "Orders" table.

### Step 5: Cleaning Up Column Headers

Before we start creating any measures or visuals, the last thing that would improve our model is to change the column names from lowercase, underscore type (i.e product_name) to proper, spaced type (i.e Product Name) this dataset was originally made in SQL which is likely why it has this format. Uisng this M Code expression, this transformation can be done easily:

```
= Table.TransformColumnNames(#"Changed Type", each Text.Proper(Text.Replace(_, "_", " ")))
```
The (#"Changed Type") part will just be changed to whatever the last transformation was, luckily the query editor automatically adds this part in anyways when you create a new step in the editor. Table.TransformColumnNames as well as Text.Proper and Text.Replace will ensure the column names are changed to proper case, replacing the underscores with spaces. Each is used to select all the column names.

