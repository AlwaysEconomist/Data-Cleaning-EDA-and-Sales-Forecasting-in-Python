# Coffee Shop Sales Analysis and Forecasting

## About the project

I have been hired as an Analyst by Maven Roasters, a fictitious coffee shop operating out of three NYC locations. I have been tasked
to analyze the transaction records and provide insights to the business and predicts future sales.

## Dataset Description

Transactions of a Coffee Shop sales based in USA from 1/1/2023 to 6/30/2023.

  - transaction_id:        Unique sequential ID representing an individual transaction
  - transaction_date:      Date of the transaction (MM/DD/YY)
  - transaction_time:      Timestamp of the transaction (HH:MM:SS)
  - transaction_qty:       Quantity of items sold
  - store_id:              Unique ID of the coffee shop where the transaction took place
  - store_location:        Location of the coffee shop where the transaction took place
  - product_id:            Unique ID of the product sold
  - unit_price:            Retail price of the product sold
  - product_category:      Description of the product category
  - product_type:          Description of the product type
  - product_detail:        Description of the product detail


## Tool used 
 - Pandas: Data manipulation, cleaning, aggregation
 - Numpy:  Numerical computations, array operations
 - Matplotlib: Data visualization, plotting, insights
 - seaborn: Statistical graphics, trends 
 - Scikit-learn: Machine learning, forecasting, modeling
 - VS Code: Editor


## Data Preparation 
 - Import libraries and Data inspection.
```

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt 
import seaborn as sns 
pd.set_option('display.max.column', None)
import os
import warnings
warnings.filterwarnings('ignore')

# Data Loading and Inspection
df = pd.read_excel(r"C:\Users\phabr\Downloads\Coffee Shop Sales.xlsx")
print(df.head(10)) # Display first 10 rows 
print(df.tail(10)) # Display last 10 rows
print(df.shape)    # Number of records and fields in the dataset
print(df.info())   # Data types + missings values
print(df.duplicated().sum()) #Check for duplicates values 
print(df.select_dtypes(include='number').describe())   # Check for summary statistics of numerical columns 

# There are no missing and duplicated values in the dataset
```

<img width="1919" height="1015" alt="image" src="https://github.com/user-attachments/assets/7a7ff443-4042-4016-9ec0-12bfac68cd49" />


 - Feature Engineering

```
data = df.copy()  # Duplicate the dataset


# Add some news columns
data['transaction_timestamp'] = data['transaction_date'].astype(str) + ' ' + data['transaction_time'].astype(str) 
data['transaction_timestamp'] = pd.to_datetime(data['transaction_timestamp'])      # Convert the data type to datetime
data['transaction_month'] = data['transaction_date'].dt.month          # Extract the month number from the transaction date column
data['transaction_monthname'] = data['transaction_date'].dt.month_name    # Extract the month name from the transaction date column
data['day_of_the_week'] = data['transaction_date'].dt.day_of_week     # Extract the day of the week from the transaction date column
data['day'] = data['transaction_date'].dt.day_name         # Extract the day name from the transaction date column
data['hour'] = data['transaction_timestamp'].dt.hour   # Extract the hour from the transaction date column
data['revenue'] = data['unit_price'] * data['transaction_qty']     # Add the revenue column = unit_price * transaction_qty

# Check the updated dataset
print(data.head(10))
print(data.info())
```

# Data Exploration

```
print(data['transaction_timestamp'].min())   # Check for earliest transaction date
print(data['transaction_timestamp'].max())   # Check for latest transaction date
print(data['transaction_timestamp'].value_counts().sort_index(ascending=True)) # Check the unique transaction hours from the smallest to largest
print(data['transaction_qty'].value_counts())  # Check the unique transactions quantities
print(data['store_id'].nunique()) # Check number of uniques stores
print(data['store_location'].unique()) # Check the unique store location
print(data['product_category'].nunique()) # Check number of uniques product_categories
print(data['product_type'].unique()) # Check the unique product_type
```

# Data Analysis 
 - Calculating KPIs
```
Total_orders = data['transaction_id'].count()
print('Total_orders:' , Total_orders) # Number of transactions in the dataset

Quantity_sold = data['transaction_qty'].sum()
print('Number of units sold:',Quantity_sold) # Number of units sold 

No_of_days = (data['transaction_date'].max() - data['transaction_date'].min()).days
print('Number of days of transactions:', No_of_days) # Number of days of transactions

Avg_order_per_day = Total_orders / No_of_days 
print('Average order per day:', Avg_order_per_day) # Average order per day

Total_revenue = data['revenue'].sum()
print('Total revenue generated from the sales:', Total_revenue) # Total revenue generated from the sales

Avg_order_value = Total_revenue / Total_orders
print('Average revenue generated per order:', Avg_order_value) # Average revenue generated per order
```
 - Orders Analysis 
```
# Find the total orders by transaction hours 
hourly_orders = data.groupby(['hour'], as_index=False).agg(Total_orders=('transaction_id', 'count'))
print(hourly_orders)

# Plot the chart for hourly orders
fig, ax = plt.subplots(figsize=[10,3])
ax.bar(x=hourly_orders['hour'].astype('str'), height=hourly_orders['Total_orders'])

ax.set_title('Hourly Orders') #Add title
ax.set_xlabel('Transaction Hours') #Add Xaxis Label
ax.spines[['top', 'right', 'left']].set_visible(False) #Remove spines
ax.yaxis.set_visible(False) #Remove Yaxis
plt.show()

# Order by day of the week 
day_of_week = data.groupby(['day_of_the_week', 'day'], as_index=False).agg(Total_orders=('transaction_id', 'count'))
print(day_of_week)

# Plot a heatmap of the orders by day of the week and transaction hour

day_hour_orders = data.pivot_table(
    index= 'day_of_the_week',
    columns= 'hour',
    values= 'transaction_id',
    aggfunc= 'count'
)

print(day_hour_orders)

fig, ax = plt.subplots(figsize = [10,4])
sns.heatmap(day_hour_orders, cmap='Blues')
plt.show()

# Find the total orders by month 
monthly_orders = data.groupby(
    ['transaction_month', 'transaction_monthname'], as_index=False).agg(Total_orders=('transaction_id', 'count'))
print(monthly_orders)

# Plot Monthly orders
fig, ax = plt.subplots(figsize=(5, 3))
ax.bar(x=monthly_orders['transaction_monthname'].str[0:3], height=monthly_orders['Total_orders'], width=0.4)

# Remove borders, Y-axis
ax.spines[['left', 'right', 'top']].set_visible(False)
ax.yaxis.set_visible(False)

# Add Ylabel and Xlabel, Title, data annotation
ax.set_xlabel('Month')
ax.set_title('Total Orders by Month')
for index, values in enumerate(monthly_orders['Total_orders']):
    ax.annotate(f"{values/1000:.1f}K", xy=(index, values * 1.05), ha='center', va='bottom')

plt.show()


```
