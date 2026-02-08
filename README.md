# CustomerAnalysisNew
# ================================
# Customer Behavior Analytics & Insight Generation System
# ================================

# 1. Upload CSV file
from google.colab import files
files.upload()

# 2. Import libraries
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import sqlite3

# 3. Load dataset
df = pd.read_csv("customer_transactions.csv")

# 4. Basic understanding
print("DATA INFO")
df.info()

print("\nMISSING VALUES")
print(df.isnull().sum())

print("\nSTATISTICAL SUMMARY")
print(df.describe())

# 5. Data cleaning & transformation
df.drop_duplicates(inplace=True)
df['Date'] = pd.to_datetime(df['Date'])
df['TotalAmount'] = df['Quantity'] * df['UnitPrice']

# 6. Descriptive statistics
mean_sales = df['TotalAmount'].mean()
variance_sales = df['TotalAmount'].var()
correlation = df[['Quantity', 'UnitPrice', 'TotalAmount']].corr()

print("\nMEAN SALES:", mean_sales)
print("VARIANCE OF SALES:", variance_sales)

print("\nCORRELATION MATRIX")
print(correlation)

# 7. Outlier detection (IQR)
Q1 = df['TotalAmount'].quantile(0.25)
Q3 = df['TotalAmount'].quantile(0.75)
IQR = Q3 - Q1

outliers = df[
    (df['TotalAmount'] < Q1 - 1.5 * IQR) |
    (df['TotalAmount'] > Q3 + 1.5 * IQR)
]

print("\nOUTLIERS")
print(outliers)

# 8. Visualizations

# Category-wise sales
df.groupby('ProductCategory')['TotalAmount'].sum().plot(kind='bar')
plt.title("Sales by Product Category")
plt.ylabel("Total Sales")
plt.show()

# Region-wise sales
df.groupby('Region')['TotalAmount'].sum().plot(kind='pie', autopct='%1.1f%%')
plt.title("Region-wise Sales Distribution")
plt.ylabel("")
plt.show()

# Correlation heatmap
sns.heatmap(correlation, annot=True, cmap="coolwarm")
plt.title("Correlation Matrix")
plt.show()

# 9. SQL Integration
conn = sqlite3.connect("customer.db")
df.to_sql("transactions", conn, if_exists="replace", index=False)

query = """
SELECT Region, SUM(TotalAmount) AS TotalSales
FROM transactions
GROUP BY Region
"""

sql_result = pd.read_sql(query, conn)

print("\nSQL QUERY RESULT")
print(sql_result)

# 10. Business Insights
print("""
BUSINESS INSIGHTS:
1. Electronics category generates the highest revenue.
2. South and West regions show strong sales performance.
3. Quantity and TotalAmount have strong positive correlation.
4. EDA helps improve inventory and marketing decisions.
""")
