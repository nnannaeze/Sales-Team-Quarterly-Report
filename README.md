# **Infinity Flame** Sales-Team-Quarterly-Report

## Problem Statement

Infinity Flame, a multinational bicycle manufacturer, has multiple sales operations across the globe. The company’s executive board and sales management team require quarterly sales information to assess their performance for the previous quarter. This analysis should provide them with the insights they need to make strategic decisions for their upcoming quarter.

To achieve this task, we will execute the following procedures:

1. Create a data story through a dashboard, providing an overview of quarterly sales performance for the executive board of Adventure Works.

2. Create a granular report with interactivity about the same sales period for the sales management team.

3. Highlight any trends identified, such as higher/lower sales by region, category, and salespersons.

4. Provide a conclusion summarizing the key takeaways for both the board of directors and sales management team.

## Stage 1: Extract Transform and Load 

### Steps followed 

**Step 1: Launch the Power BI app and Load the Sales data**

1. Launch the Power BI Desktop in your local Machine

2. Load the Infinity Flame Sales.xlsx file into Power BI and select Transform.

Begin by loading the Infinix Flame Sales data file into Power BI.

![Screenshot (205)](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/2cc2c354-c490-4a0d-9133-75bb54a54d5f)

3. Select the Transform Data option to enter the Power Query Editor.
Observe that we have imorted the following Tables into the Power Query

        * Products
        * Region
        * Sales
        * Salespersons


**Step 2: Perform data cleaning on the Queries**

1. Optimize the dataset by assigning datatypes accordingly. Ensure that every column in all the Queries are in their correct datatype.

2. Navigate to the View tab. Select the Column Quality, Column Distribution, and Column Profile checkboxes. Examining column quality, distribution, and profiles helps assess the data's health. It also allows for the early detection of issues, saving time and preventing errors in later stages of the analysis.
![Screenshot (206)](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/ba1da378-8a18-409b-895a-9f0b47843fc3)

3. In the Products Table, the `ProductKey` is the primary key therefore ensure it contain equal number of Unique values as well as  Distinct values this is because, the primary key only contains unique values. the same should be for the following
 * `SalesterritoryKey` in the **`Region`** table
 * `SalesOrderNumber` in the **`Sales`** table
 * `EmployeeKey` in the **`Salesperson`** table

4. Check for missing values in the Columns(Entities)  by performing following:
* Assign correct datatypes to Entities
* In the view tab, check boxes for **column Quality**, **Column Distribution**, and **Column Profile** and then check the column headers for Entities that did not have the following         

        Valid  100%
        Errors 0%
        Empty  0%
In our case the columns does not contain missing values and errors
5. Navigate to the *home tab* and click *Close & Apply* to return to the Power BI Desktop


**Step 3: Load the Currency Exchange data using Python Script**
1. Select Get Data and choose Python script

![Screenshot (208)](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/5042b0b1-1cc7-49ad-8585-ca2daedac7b1)

and then paste the following code into the script window in Power BI:

        import pandas as pd
        from io import StringIO

        data = """Exchange ID;ExchangeRate;Exchange Currency
        1;1;USD
        2;1;USD
        3;1;USD
        4;1;USD
        5;1;USD
        6;1.35;CAD
        7;0.85;EUR
        8;0.85;EUR
        9;1.3;AUD
        10;0.75;GBP"""
        df = pd.read_csv(StringIO(data), sep=';')

        # Return the transformed dataframe
        df
        # Return the transformed dataframe
        df
![Screenshot (210)](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/3bfde48a-cb80-4871-b882-b10cbc647052)

Enter Ok to create the Table(df)

![Screenshot (209)](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/ae5f7f8f-f834-4e31-bbb8-5b55715c0b5d)

**Step 4: Create a relationship between the Countries and Exchange Datatables**
1. Create a relationship between the Region and Exchange Data tables on the Exchange ID field.

Select the Model view option from the left-hand sidebar to view your data model.
Begin by creating a relationship on the Exchange ID field that links the Region table with the Exchange Data table, which forms the foundation of the currency translation mechanism.

![Screenshot (213)](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/3439b1d7-d980-4ee6-aeda-5799547ecae5)

Notice that the Power Bi has already automatically established relationships among the initial tables. We check the relationships to confirm their suitability.

![Screenshot (214)](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/77880bf0-e578-466a-b1f9-6e1de70f7d1d)


**Step 5: Create Sales in USD Calculated Table**
Create a new Sales table(SalesUSD) that will convert the `Cost` and `Total Sales` which has so many currency into a single currency USD.
1. Select New Table and add the following DAX code to create a new calculated table:

        SalesUSD = 
        ADDCOLUMNS(
            Sales,
            "Country", RELATED(Region[Country]),
            "Exchange Rate", RELATED('Exchange data'[ExchangeRate]),
            "Exchange Currency", RELATED('Exchange data'[Exchange Currency]),
            "Cost USD", [Cost] * RELATED('Exchange data'[ExchangeRate]),
            "Total Sales USD", [Total Sales] * RELATED('Exchange data' [ExchangeRate]
            )

2. Format the newly created columns to currency datatype and 2 decimal places.

![Screenshot (215)](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/b731d41b-8658-4bb0-bee7-35fa361fec7b)


**Step 6: Create a relationship between the Sales in USD and Sales tables**
1. Create a relationship between the Sales in USD and Sales tables on the SalesOrderNumber field.

Link the Sales in USD table with the Sales table by creating a relationship based on the Order ID field, enabling Infinity Flame to contrast the original sales data with its USD equivalent.

![Screenshot (216)](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/492a1951-55e7-4556-add9-37893df2f5eb)



**Step 7: Create required Measures** 

1. Create a table where all the measures will be stored.

        Navigate to home tab and select Enter data
        a new Table appears, we rename it Key Measures

2. Click Ok

![Screenshot (218)](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/5bda6011-22ce-4fbf-a7c0-e36bb1c48af0)

3. To create the Total Sales Measure

        right click on Key Measure and select New measures
        write the following DAX expression
                Total Sales = SUM(SalesUSD[Total Sales USD])
                Enter
4. Repeat the step to create the following Measures

        total cost = SUM(SalesUSD[Cost USD])
        Enter


        Sales comparison = CALCULATE([Total Sales], DATEADD(SalesUSD[OrderDate].[Date], -1, MONTH))
        Enter


        Profit_PY = CALCULATE([Profit], DATEADD(SalesUSD[OrderDate].[Date], -1,YEAR))
        Enter


        Top 3 Salespersons = 
        VAR 
        RankigContext = VALUES(Salesperson[Salesperson])
        RETURN 
        CALCULATE([Total Sales], TOPN(3,
        ALL(Salesperson[Salesperson]), [Total Sales]),
        RankigContext)
        Enter


        Revenue YoY % = 
        VAR RevenuePreviousYear = CALCULATE([Total Sales], SAMEPERIODLASTYEAR( SalesUSD[OrderDate].[Date]))
        RETURN
        DIVIDE(
        [Total Sales] - RevenuePreviousYear, RevenuePreviousYear)
        Enter


        Revenue PY = 
        VAR RevenuePreviousYear = CALCULATE([Total Sales], SAMEPERIODLASTYEAR( SalesUSD[OrderDate].[Date]))
        RETURN
        RevenuePreviousYear
        Enter


        Profit = [Total Sales]-[total cost]
        Enter


        Best Salesperson = 
        VAR 
        RankigContext = VALUES(Salesperson[Salesperson])
        RETURN 
        CALCULATE([Total Sales], TOPN(1,
        ALL(Salesperson[Salesperson]), [Total Sales]),
        RankigContext)
        Enter


        Best Product = 
        VAR 
        RankigContext = VALUES(Products[Product])
        RETURN 
        CALCULATE([Total Sales], TOPN(1,
        ALL(Products[Product]), [Total Sales]),
        RankigContext)
        Enter


        Profit Margin = [Profit]/[Total Sales]
        Enter


        Quarterly Profit = CALCULATE([Profit],DATESQTD(SalesUSD[OrderDate]))


        Profit_Previous Year = CALCULATE([Profit], DATEADD(SalesUSD[OrderDate].[Date], -1, YEAR)) 
        Enter



5. Ensure all the Measures are in their correct format.

        Total sales : currency Format and 2 decimal places

        Profit : currency Format and 2 decimal places

        Profit Margin : percentage format 

        Revenue PY : currency Format and 2 decimal places

        Sales comparison : currency Format and 2 decimal places

        Revenue YoY % : percentage format 

        Quarterly Profit : currency Format and 2 decimal

        total cost :currency Format and 2 decimal


![Screenshot (219)](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/9c01dfaf-e291-4fb3-8972-f797e96dd91b)


This brings an end to Stage 1 


## Stage 2: Analysis

### Steps followed

**Step 1: Format the Canvas**

1. Choose a Canvas Settings Type size of 16.9

2. Choose a Canvas Background color of `#101726` and set Transparency to 0%

**Step 2: Create visualizations**

1. Create a  Total Sales quarterly visual using the Stacked Column Chart visualization. This is achieved by the following Steps.

        1. select the Stacked Column Chart from the visualization pane
        2. drag the `Total Sales` Measure to the fields well y-axis 
        3. drag the `SalesUSD[OrderDate] to the fields well x-axis

![image](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/c6dd9c40-efbc-448d-8440-8791949f65bc)


2. Create a Sales by Category visual using the Donot Chart visualization. This is achieved by the following Steps.

        1. select the Donot Chart from the visualization pane
        2. drag the Product[Category] to the fields well legend 
        3. drag the `Total Sales` Measure to the fields well values

![image](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/400019a5-2b39-4013-b0e0-777c4a66c3d3)
        
3. Create a   Sales by Coutry visual using the Bar Chart visualization. This is achieved by the following Steps.

        1. select the Bar Chart from the visualization pane
        2. drag the Region[Country] to the fields well y-axis 
        3. drag the `Total Sales` Measure to the fields well x-axis

4. Create a   Sales by Salesperson visual using the Bar Chart visualization. This is achieved by the following Steps.

        1. select the Bar Chart from the visualization pane
        2. drag the Region[Country] to the fields well y-axis 
        3. drag the `Total Sales` Measure to the fields well x-axis

5. Create a Monthty Sales time line visual using the Line Chart visualization. This is achieved by the following Steps.

        1. select the Line Chart from the visualization pane
        2. drag the `Total Sales` Measure to the fields well y-axis 
        3. drag the `SalesUSD[OrderDate] to the fields well x-axis

6. Create a Monthty Profit time line visual using the Line Chart visualization. This is achieved by the following Steps.

        1. select the Line Chart from the visualization pane
        2. drag the `Profit` Measure to the fields well y-axis 
        3. drag the `SalesUSD[OrderDate] to the fields well x-axis
 ![image](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/f9c4c07d-2b8e-4cf4-a2ef-5c35f906c56d)
       

7. Create a Year Slicer using the Slicer visualization. This is achieved by the following Steps.

        1. select the Slicer from the visualization pane
        2. drag the`SalesUSD[OrderDate] to the fields well 

8. Repeat same for the Quarterly Slicer

![image](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/182a0e4f-99c5-457f-a3a6-7f0be0a9a656)

9.  Create a card visuals to highlight key metrics for the executive board such as `Profit`, `revenue`, `profit_Previous Quarter` and `Profit Margin`
![image](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/cc81ed8f-836a-40b5-936c-16535485a656)


10. Create a Salesperson Slicer using the Slicer visualization. This is achieved by the following Steps.

        1. select the Slicer from the visualization pane
        2. drag the`Salesperson[Salesperson] to the fields well and use dropdown Slicer style
![image](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/70756703-a981-4e4b-a44c-6b8eab2fc763)


11. Create a Multiple card visual to highlight the 3 Top Salesperson.

        1. select the Multiple card from the visualization pane
        2. drag the Top 3 Salesperson Measure and Salesperson[Salesperson]to the fields well.
        3. go to the Filters pane and select TOPN in the filter type of Filters on this visual
        4. select top 3 and drag Salesperson[Salesperson] into the `by value` fields

![image](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/64817917-edda-4895-b3cf-95d690b009d2)



**Step 3: Create Customized Tooltip and Drill-through**

1. Create a Sales by Salesperson Customized tooltip visual using the Bar Chart visualization. This is achieved by the following Steps.

        1. Create a new page and rename it Tooltip
        2. format the report Canvas setting to custom:
           Height: 620px
           width: 320px 
        3. select the Bar Chart from the visualization pane
        4. drag the `Total Sales` Measure to the fields well xy-axis 
        5. drag the Salesperson[Salesperson] to the fields well y-axis

2. Create a `Total sales` card visual to highlight key metrics for the executive board such as when the cursor is hovered over the `Sales by Country` visual, a Tooltip containing the Tooltip page will pop up. 
![Presentation123](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/88a9e44b-90dd-4a7e-96e6-1dc40ca943b7)

3. Select `Sales by Country`visual and go to Format, select Tooltip from  General and perform the following under Options

        1. Type: Report page
        2. page: Tooltip

4. Create a detailed page table visual using the Table visualization. This is achieved by the following Steps.

        1. Create a new page and rename it Drillthrough
        2. format the report Canvas Background to Black
        3. select the Table from the visualization pane
        4. drag  `Subcategory`, `Product`, `Sum of Quantity`,`Total Sales`,`Profit Margin`, to the Column fields well
        5. use conditional formatting to format the `Profit Margin` such that objects changes color from red to green depending on it's percentage content.
        6. drag the Product[Category] to the fields well Drill through.

5. Create a `Product`[Category] card visual to highlight the selected category

![image](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/5ff2d3a1-3a1e-4712-a2a2-0ca1a406cbb8)


**Step 4: Publish to Power BI Service**

1. Save the Project in power Bi Desktop and select Publish

2. choose the Workspace to Publish the report 
![image](https://github.com/nnannaeze/Sales-Team-Quarterly-Report/assets/148848746/e110c454-8fa2-4e34-9574-b6b68d4a5707)





# Insights

### 1. for 2020, 1st Quarter, 
    Total Sales = $432.46k
    Previous Year Total Sales = $282.30k
    Profit = $493.60
    Profit Previous Quarter = $1.46k
    Profit Margin = 0.11% 

### 2. Sales by Country:
**United State** has the highest number of Sales(*$101.99k*) recorded in the 2nd quarter of 2020 followed by 

* Canada

* Australia

* France etc

### 3. Sales by Category:
**Bikes** has the highest number of sales(*147.87k*) whichconstitute of 74.44% of the total sales recorded in the 2nd quarter of 2020 followed by 

* Component

* Clothing and 

* Accessories** 

### 4. Top 3 Salesperson:
* Jae Pak

* Linda Mitchell

* Tsvi Reiter
### 5. Sales Timeline for 2020, 1st Quarter:
Sales where at $95,107.52 in the Month of January. It increased to $183,123.08 in the Month of February and decreased slightly to $154,231.00 in the Month of March

### 6. Profit Timeline for 2020, 1st Quarter:
Profit was at $495.98 in the Month of January. It increased to $1,047.43 in the Month of February and decreased slightly to $57.84 in the Month of March

### 7. Profit Timeline for 2020, 1st Quarter:
Profit was at $495.98 in the Month of January. It increased to $1,047.43 in the Month of February and decreased slightly to $57.84 in the Month of March

## Key takeaways

### 1. Under performing salespersons
the following are the least performing Salesperson across Regions
* Amy Alberts
* Syed Abbas
* Stephen Jiang
* Brian Welcker

### 2. Observations:
1. The Total Sales for 2020, 1st Quarter = $432.46k
2. The Total Sales for same Period last year(2019, 1st Quarter) = $282.30k
3. profit for  2020, 1st Quarter = $432.46k
4. profit for the same period last year = $13.20k

The Total sales for the 1st quarter 2020 is higher than that for the same period last year but have a very low profit than same period last year.
