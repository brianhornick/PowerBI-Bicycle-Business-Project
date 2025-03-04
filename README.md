# Analyzing Business Data and Building a Dashboard in Power BI - by Brian Hornick
## Problem Statement
A bicycle chain company currently owns 3 stores in the United States and needs a dashboard to make better data-driven decisions. This dataset was found on Kaggle.com LINK: (https://www.kaggle.com/datasets/dillonmyrick/bike-store-sample-database). This dataset will serve as a base, which we will use to design our dashboard. I will be outlining each step taken in creating this dashboard and the end goal is to complete a sharp-looking, easy-to-use dashboard that can answer a variety of key questions with ease. Some of these questions include:

1. How does revenue differ by store?
2. What products sell the most, how does model year impact sales?
3. How does sales differ by brand?
4. What months of the year are sales the highest? Is the stock high enough to meet demand?
5. How soon on average are orders being shipped before they are required? How often are orders being placed too late?

To answer these questions, along with many others, the first step required is to import the dataset into PowerBI.

### Step 1: Importing the dataset

This step is straightforward as all of the data is contained in separate CSV files. Simply by clicking "Get Data", clicking on CSV and loading each of the tables in, keeping everything else the same, we now have our 9 tables loaded in.

### Step 2: Data Model

Now it's time to set up relationships between the tables and build the model. In the "Data Load" section of the "Current File" settings, "Autodetect new relationships after data is loaded" is checked so Power BI should be able to do most of this work itself. After importing the CSVs, here is what the model has defaulted to:

![image_alt](https://github.com/brianhornick/PowerBI-Bicycle-Business-Project/blob/main/Image/Screenshot%202025-03-04%20132140.png?raw=true)

The data model looks good, all relationships are one to many, with the dimension tables flowing nicely to the two fact tables, "Orders" and "Order_items". This forms a nice 'Snowflake Schema' with the nuance being that the "Stock" table is tied to both products and "Stores". It is possible to merge the "Brands" and "Categories" tables with the "Products" table to create more of a 'Star Schema' but for this project, this should work fine.
