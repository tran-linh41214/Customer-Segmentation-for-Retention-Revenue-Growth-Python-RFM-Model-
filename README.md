
![banner rfm](https://github.com/user-attachments/assets/04e4c230-5608-4dff-ad47-0ebf3b8b9624)


# Python_RFM_project
# 📊 Project Title: [RFM Segmentation]  
Author: [Linh Tran]    
Tools Used: Python 

---

## 📑 Table of Contents  
1. [📌 Background & Overview](#-background--overview)  
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)  
3. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)

---

## 📌 Background & Overview  

### Objective:
### 📖 What is this project about? 
 
- A Python-based solution for customer segmentation using the RFM (Recency, Frequency, Monetary) model.
- Objective: Automate RFM analysis to identify high-value customers, improve retention strategies, and personalize marketing campaigns efficiently.


### 👤 Who is this project for?  

✔️ **Retail business owners & marketers**  
✔️ **Data analysts & business analysts**  
✔️ **E-commerce managers**  
✔️ **CRM & customer success teams**  
✔️ **Decision-makers & stakeholders**  

###  ❓Business Questions:  


✔️ Identify loyal customers and potential high-value customers.  
✔️ Segment customers based on purchasing behavior to personalize marketing campaigns.  
✔️ Evaluate customer value (Customer Value) to optimize retention and engagement strategies.  
✔️ Automate the customer segmentation process instead of manual Excel calculations.  
✔️ Provide insightful reports to help the Marketing team implement targeted campaigns effectively.

### 🎯Project Outcome:  
 
In summary, focusing on Frequency in the retail sector can provide stability and growth by fostering loyal customer relationships, improving operational efficiencies, and enhancing the overall customer experience. This approach helps build a dedicated customer base that ensures consistent revenue while also encouraging brand advocacy and community engagement.  

---

## 📂 Dataset Description & Data Structure  

### 📌 Data Source  

- Source: Company database  
- Size: 8 columns, 541909 rows 
- Format: .csv

### 📊 Data Structure & Relationships  

**This project used 2 tables:**

**Table 1: Transactions table:**

| Column Name | Data Type | Description |  
|-------------|----------|-------------|  
| InvoiceNo  | object      | Invoice number. Nominal, a 6-digit integral number uniquely assigned to each transaction. If this code starts with letter 'C', it indicates a cancellation. |  
| StockCode        | object     | Product (item) code. Nominal, a 5-digit integral number uniquely assigned to each distinct product. |
| Description    | object     | Product (item) name. Nominal. |  
| Quantity | int | The quantities of each product (item) per transaction. Numeric. |
| InvoiceDate       | object   | Invoice Date and time. Numeric, the day and time when each transaction was generated. |  
| UnitPrice | float | Unit price. Numeric, Product price per unit in sterling. |
| CustomerID | flotat| Customer number. Nominal, a 5-digit integral number uniquely assigned to each customer. |
| Country | object | Country name. Nominal, the name of the country where each customer resides. |


**Table 2: Segmentation table:**

| Column Name | Data Type | Description |  
|-------------|----------|-------------| 
| Segment      | object   | Customer Segmentation Category    |
| RFM Score    | string   | RFM Score assigned to each segment |

---

## ⚒️ Main Process

1️⃣  Exploratory Data Analysis (EDA)   


***1.1. Explore data***
```python
data.info()
     
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 541909 entries, 0 to 541908
Data columns (total 8 columns):
 #   Column       Non-Null Count   Dtype  
---  ------       --------------   -----  
 0   InvoiceNo    541909 non-null  object 
 1   StockCode    541909 non-null  object 
 2   Description  540455 non-null  object 
 3   Quantity     541909 non-null  int64  
 4   InvoiceDate  541909 non-null  object 
 5   UnitPrice    541909 non-null  float64
 6   CustomerID   406829 non-null  float64
 7   Country      541909 non-null  object 
dtypes: float64(2), int64(1), object(5)
memory usage: 33.1+ MB
```
- Decription nulls => remove column
- Customer ID nulls => remove row
- InvoiceDate => convert to Datetime type
- CustomerID => convert to object type

```python
print('DataFrame dimension: ', data.shape)
     
DataFrame dimension:  (541909, 8)

print(data.describe())
     
            Quantity      UnitPrice     CustomerID
count  541909.000000  541909.000000  406829.000000
mean        9.552250       4.611114   15287.690570
std       218.081158      96.759853    1713.600303
min    -80995.000000  -11062.060000   12346.000000
25%         1.000000       1.250000   13953.000000
50%         3.000000       2.080000   15152.000000
75%        10.000000       4.130000   16791.000000
max     80995.000000   38970.000000   18287.000000
```

- Quantity < 0 => cancellation => remove in df calculate RMF, keep in df data to get insight
- Quantity too big => remove
- UnitPrice < 0 => remove

***1.2. Create a canceled orders table.***

```python
# Invoice chứa ký tự C ở đầu là những giao dịch bị hủy
cancelled = data['InvoiceNo'].astype(str).str.contains('C')
# Gán giá trị 0 với đơn không hủy, 1 với đơn hủy
cancelled.fillna(0, inplace=True)
cancelled = cancelled.astype(int)
cancelled.value_counts()
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/5631a1e9-36fa-4623-bb41-3c9defa68160" alt="image">
</p>


*=> Number of canceled orders: 9288*
*Calculate number of cancelled transactions and percentage:*
```python
c1 = data['order_cancelled'].value_counts()[1]
c2 = data.shape[0]
print("Number of cancelled transactions: ", c1)
print('Percent of orders cancelled: {}/{} ({:.2f}%) '.format(c1, c2, c1/c2*100))
```
*Number of cancelled transactions:  9288*
*Percent of orders cancelled: 9288/541909 (1.71%)*

***1.3. Clean data, tạo df_Transaction***

1.3. Clean data, tạo df_Transaction

```python
# Tạo một bản sao của df 'data'
df_Transaction = data.copy()

# Xóa các giao dịch bị hủy
df_Transaction = df_Transaction[df_Transaction['order_cancelled'] == 0]

# Bỏ cột Description
df_Transaction = df_Transaction.drop(columns=['Description'])

# Loại bỏ các dòng thiếu CustomerID
df_Transaction = df_Transaction.dropna(subset=['CustomerID'])

# Kiểm tra dupicates
print('Duplicate entries: {}'.format(df_Transaction.duplicated().sum()))
print('{}% rows are duplicate.'.format(round((df_Transaction.duplicated().sum()/df_Transaction.shape[0])*100),2))

# Bỏ duplicate data
df_Transaction.drop_duplicates(inplace = True)

# Đổi kiểu dữ liệu cho CustomerID và InvoiceDate
df_Transaction['CustomerID'] = df_Transaction['CustomerID'].astype(str)  # Chuyển CustomerID sang kiểu object
data['InvoiceDate'] = pd.to_datetime(data['InvoiceDate'], format='%d/%m/%Y %H:%M') #Chuyển InvoiceDate sang datetime

# Xóa UnitPrice và Quantity âm
df_Transaction = df_Transaction[(df_Transaction['UnitPrice'] > 0) & (df_Transaction['Quantity'] > 0)]

# Kiểm tra kết quả EDA
print(df_Transaction.head())
print(df_Transaction.describe())
print(df_Transaction.dtypes)
     
Duplicate entries: 5194
1% rows are duplicate.
  InvoiceNo StockCode  Quantity         InvoiceDate  UnitPrice CustomerID  \
0    536365    85123A         6 2010-12-01 08:26:00       2.55    17850.0   
1    536365     71053         6 2010-12-01 08:26:00       3.39    17850.0   
2    536365    84406B         8 2010-12-01 08:26:00       2.75    17850.0   
3    536365    84029G         6 2010-12-01 08:26:00       3.39    17850.0   
4    536365    84029E         6 2010-12-01 08:26:00       3.39    17850.0   

          Country  order_cancelled  
0  United Kingdom                0  
1  United Kingdom                0  
2  United Kingdom                0  
3  United Kingdom                0  
4  United Kingdom                0  
            Quantity                    InvoiceDate      UnitPrice  \
count  392690.000000                         392690  392690.000000   
mean       13.118997  2011-07-10 19:12:51.826224128       3.125913   
min         1.000000            2010-12-01 08:26:00       0.001000   
25%         2.000000            2011-04-07 11:12:00       1.250000   
50%         6.000000            2011-07-31 12:02:00       1.950000   
75%        12.000000            2011-10-20 12:53:00       3.750000   
max     80995.000000            2011-12-09 12:50:00    8142.750000   
std       180.492710                            NaN      22.241892   

       order_cancelled  
count         392690.0  
mean               0.0  
min                0.0  
25%                0.0  
50%                0.0  
75%                0.0  
max                0.0  
std                0.0  
InvoiceNo                  object
StockCode                  object
Quantity                    int64
InvoiceDate        datetime64[ns]
UnitPrice                 float64
CustomerID                 object
Country                    object
order_cancelled             int64
dtype: object
```
2️⃣ SQL/ Python Analysis 

# **2. TÍNH RFM**
***2.1. Tính RFM***
```python
df_rmf = df_Transaction.groupby('CustomerID').agg({'InvoiceDate': lambda x: (last_date - x.max()).days, 'InvoiceNo': lambda x: len(x), 'Revenue': lambda x: x.sum()}).reset_index()
df_rmf['InvoiceDate'] = df_rmf['InvoiceDate'].astype(int)

# Rename columns
df_rmf.rename(columns={'InvoiceDate': 'Recency',
                         'InvoiceNo': 'Frequency',
                         'Revenue': 'Monetary'}, inplace=True)
df_rmf.head()
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/81fd94b8-4185-4028-bda5-9e956040046b" alt="image">
</p>


***2.2. Tính RFM Score***
```python
# Tính điểm Recency (R), Frequency (F), Monetary (M)
df_rmf['R_Score'] = pd.qcut(df_rmf['Recency'], 5, labels=[5, 4, 3, 2, 1])
df_rmf['F_Score'] = pd.qcut(df_rmf['Frequency'], 5, labels=[1, 2, 3, 4, 5])
df_rmf['M_Score'] = pd.qcut(df_rmf['Monetary'], 5, labels=[1, 2, 3, 4, 5])

# Kết hợp các điểm df_rmf thành một chuỗi
df_rmf['RFM_Score'] = df_rmf['R_Score'].astype(str) + df_rmf['F_Score'].astype(str) + df_rmf['M_Score'].astype(str)

# Kết quả là DataFrame với các cột R_Score, F_Score, M_Score, và df_rmf_Score
print(df_rmf[['CustomerID', 'Recency', 'Frequency', 'Monetary', 'RFM_Score']])
```
<p align="center">
  <img src="https://github.com/user-attachments/assets/db0f3f2b-6c63-4a93-a697-91f6d84d77c3" alt="image">
</p>


# **3. Segmentation**
```python
# Tính số lượng khách hàng trong mỗi segment
df_describe = df_user.groupby('Segment').agg({'CustomerID': lambda x: len(x)}).reset_index()

df_describe.rename(columns={'CustomerID': 'Count'}, inplace=True)
df_describe['percent'] = (df_describe['Count'] / df_describe['Count'].sum()) * 100
df_describe['percent'] = df_describe['percent'].round(1)

print(df_describe)
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/f5cb947d-934f-40ac-a0ec-fb456f93eb66" alt="image">
</p>


| **Segment**             | **Characteristics**                                                                           | **Recommendations**                                                                                     |
|-------------------------|--------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------|
| **Champions**           | Recently purchased, frequent buyers, and highest spenders.                               | Offer gifts, special promotions, and free trials of new products. Encourage them to promote the brand. |
| **Loyal**               | Spend significantly and purchase frequently. Respond well to promotions.                  | Recommend higher-value products. Collect product reviews. Maintain regular engagement.                 |
| **Potential Loyalist**  | New customers who have made multiple purchases.                                          | Introduce loyalty programs or memberships and suggest other relevant products.                         |
| **New Customers**       | Recently made a purchase but not frequently.                                             | Provide initial support to ensure a smooth first-time shopping experience.                             |
| **Promising**           | Recently made a purchase but with low spending.                                          | Build brand awareness and offer free trials.                                                           |
| **Need Attention**      | Recency, Frequency, and Monetary (RFM) values are above average but not consistent.      | Provide time-limited offers based on past purchases. Encourage repeat purchases.                       |
| **About to Sleep**      | Below-average RFM values, at risk of becoming inactive.                                  | Recommend popular products or special discounts. Reconnect with them.                                  |
| **At Risk**             | Previously spent a lot and purchased frequently but haven’t bought in a while.          | Send personalized emails, messages, or promotions to re-engage them.                                   |
| **Cannot Lose Them**    | Used to make large and frequent purchases but have not returned for a long time.        | Re-attract them with new product offerings or renewal options to prevent losing them to competitors.    |
| **Hibernating Customers** | Haven't purchased in a long time, low spending, and few orders.                      | Suggest relevant products with special offers. Reinforce brand value.                                  |
| **Lost Customers**      | Lowest RFM values, least engaged customers.                                             | Attempt re-engagement campaigns, but consider deprioritizing if they remain inactive.                  |
---

## 🔎 Final Conclusion & Recommendations  

### Strategic Recommendations for the Top 5 Segments

#### 1. Cannot Lose Them
- **Recency:** High Recency median, indicating they haven't made a purchase in a while but were significant spenders.
  - **Strategy:** Implement win-back campaigns with personalized offers. Provide loyalty incentives or exclusive discounts to encourage re-engagement. Conduct surveys to understand their drop-off reasons.
- **Frequency:** A broad frequency range suggests a mix of once-frequent shoppers.
  - **Strategy:** Analyze past purchases and target these customers with reminders or promotions on their preferred products. Introduce a loyalty program to re-engage them.
- **Monetary:** High median monetary value with a wide range.
  - **Strategy:** Focus on high-value offerings and VIP experiences. Provide exclusive access to premium products and personalized services.

#### 2. Champions
- **Recency:** Low Recency scores, indicating recent purchases.
  - **Strategy:** Maintain engagement with exclusive updates, early access to new products, and tailored experiences. Leverage their feedback for product and service improvements.
- **Frequency:** High and consistent purchasing behavior.
  - **Strategy:** Introduce a subscription model or referral program. Reward loyalty with exclusive perks.
- **Monetary:** Consistent spending, though not as high as "Cannot Lose Them."
  - **Strategy:** Implement upselling and cross-selling strategies, offering product bundles or complementary items.

#### 3. At Risk
- **Recency:** Higher Recency compared to Champions, indicating a growing inactivity period.
  - **Strategy:** Deploy reactivation campaigns with limited-time discounts. Remind them of past positive experiences.
- **Frequency:** Moderate, with some previously frequent buyers.
  - **Strategy:** Personalized re-engagement programs focusing on their frequent purchases. Offer incentives to restore previous buying habits.
- **Monetary:** Varied spending, with some high-value outliers.
  - **Strategy:** Provide premium customer service and personalized recommendations to high-spending customers.

#### 4. Loyal
- **Recency:** Generally low, indicating recent interactions.
  - **Strategy:** Keep the brand top-of-mind with regular updates and product insights. Introduce tiered loyalty rewards.
- **Frequency:** Consistent purchases, though less frequent than Champions.
  - **Strategy:** Increase engagement through events, community forums, and a point-based reward system.
- **Monetary:** Moderate and consistent spending behavior.
  - **Strategy:** Increase spending by promoting a broader range of products. Implement a loyalty program that incentivizes higher spending.

#### 5. Promising
- **Recency:** Very low Recency, suggesting recent engagement.
  - **Strategy:** Enhance their initial purchase experience with excellent follow-up service. Encourage reviews and engagement.
- **Frequency:** Lowest among all segments, but potential for growth.
  - **Strategy:** Encourage repeat purchases with follow-up offers and a welcome campaign introducing the brand’s value.
- **Monetary:** Lowest median spending, indicating room for growth.
  - **Strategy:** Recommend products in their price range. Offer introductory discounts to boost spending.

### Selecting the Parameter: R/F/M

#### Why Emphasize Frequency (F) in RFM Analysis for Retail?

##### Customer Relationship Building
- **Repeat Business:** Frequent shoppers are more likely to continue purchasing, ensuring steady revenue.
- **Customer Lifetime Value (CLV):** Higher purchase frequency can significantly boost CLV.
- **Customer Feedback & Data:** Frequent interactions provide insights into customer preferences and enhance personalization.

##### Marketing and Sales Strategies
- **Targeted Promotions:** Use purchase history data to personalize promotions.
- **Personalization:** Predictive analytics can tailor recommendations to frequent buyers.

##### Operational Efficiency
- **Inventory Management:** Frequent purchase data helps optimize stock levels.
- **Resource Allocation:** Focus marketing and customer service efforts on frequent buyers.

##### Brand Advocacy
- **Referrals:** Loyal, frequent shoppers are likely to recommend the brand.
- **Community Building:** Engaging frequent buyers in exclusive events strengthens brand loyalty.

##### Strategic Development
- **Predictive Analytics:** Identifying patterns in frequent shoppers aids in forecasting demand.
- **Business Growth:** Increasing purchase frequency contributes to sustainable revenue growth.

#### Conclusion
Focusing on Frequency in the retail sector fosters loyal customer relationships, improves operational efficiencies, and enhances the customer experience. This strategy helps develop a strong, engaged customer base that contributes to consistent revenue while promoting the brand through advocacy and community engagement.


---
