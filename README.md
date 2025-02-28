# Python_RFM_project
# 📊 Project Title: [RFM Segmentation]  
Author: [Linh Tran]    
Tools Used: Python  

---

## 📑 Table of Contents  
1. [📌 Background & Overview](#-background--overview)  
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)  
3. [🧠 Design Thinking Process](#-design-thinking-process)  
4. [📊 Key Insights & Visualizations](#-key-insights--visualizations)  
5. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)

---

## 📌 Background & Overview  

### Objective:
- A Python-based solution for customer segmentation using the RFM (Recency, Frequency, Monetary) model.
- Objective: Automate RFM analysis to identify high-value customers, improve retention strategies, and personalize marketing campaigns efficiently.

### 👤 Who is this project for?  

✔️ **Retail business owners & marketers**  
✔️ **Data analysts & business analysts**  
✔️ **E-commerce managers**  
✔️ **CRM & customer success teams**  
✔️ **Decision-makers & stakeholders**

###  ❓ **Business Questions:**  

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

#### 1️⃣ Tables Used:  
Mention how many tables are in the dataset.    


| Column Name | Data Type | Description |  
|-------------|----------|-------------|  
| InvoiceNo  | object      | Invoice number. Nominal, a 6-digit integral number uniquely assigned to each transaction. If this code starts with letter 'C', it indicates a cancellation. |  
| StockCode        | object     | Product (item) code. Nominal, a 5-digit integral number uniquely assigned to each distinct product. |
| Description    | object     | Product (item) name. Nominal. |  
| Quantity | int | gshsh |
| InvoiceDate       | object   | Invoice Date and time. Numeric, the day and time when each transaction was generated. |  
| UnitPrice | float | Unit price. Numeric, Product price per unit in sterling. |
| CustomerID | flotat| Customer number. Nominal, a 5-digit integral number uniquely assigned to each customer. |
| Country | object | Country name. Nominal, the name of the country where each customer resides. |


---

## 🧠 Design Thinking Process  

1️⃣ *Empathize* – Gathered insights into customer behavior and challenges faced by the marketing team in segmenting large datasets manually.

2️⃣ *Define Point of View* – Identified the core problem: the need for an automated and scalable segmentation solution to enhance targeted marketing efforts.

3️⃣ *Ideate* – Explored different methodologies, ultimately selecting the RFM model as the most suitable approach for customer segmentation.

4️⃣ *Prototype and Review* – Developed a Python-based implementation, tested the model on real data, and refined the approach based on feedback from the marketing team.  

![image](https://github.com/user-attachments/assets/c7bf4f9a-5845-4a61-9e46-7e25d32064b4)

---

## ⚒️ Main Process

1️⃣ Exploratory Data Analysis (EDA) 

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

![image](https://github.com/user-attachments/assets/5631a1e9-36fa-4623-bb41-3c9defa68160)

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

3️⃣ Python Analysis 
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

![image](https://github.com/user-attachments/assets/81fd94b8-4185-4028-bda5-9e956040046b)

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
![image](https://github.com/user-attachments/assets/db0f3f2b-6c63-4a93-a697-91f6d84d77c3)

# **3. Segmentation**
```python
# Tính số lượng khách hàng trong mỗi segment
df_describe = df_user.groupby('Segment').agg({'CustomerID': lambda x: len(x)}).reset_index()

df_describe.rename(columns={'CustomerID': 'Count'}, inplace=True)
df_describe['percent'] = (df_describe['Count'] / df_describe['Count'].sum()) * 100
df_describe['percent'] = df_describe['percent'].round(1)

print(df_describe)
```
![image](https://github.com/user-attachments/assets/f5cb947d-934f-40ac-a0ec-fb456f93eb66)

4️⃣ Power BI Visualization  (applicable for PBI Projects)

---

## 📊 Key Insights & Visualizations  

## 🔎 Final Conclusion & Recommendations  

| Phân đoạn               | Đặc điểm                                                                                   | Khuyến nghị                                                                                          |
|-------------------------|-------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| **Champions**           | Mua gần đây, mua thường xuyên và chi tiêu nhiều nhất                                    | Tặng quà, tặng các chương trình ưu đãi. Dùng thử sản phẩm mới miễn phí. Khuyến khích họ quảng bá thương hiệu. |
| **Loyal**               | Chi tiêu khá nhiều và mua thường xuyên. Phản hồi tốt với các chương trình khuyến mãi.     | Đề xuất các sản phẩm giá trị cao hơn. Thu thập đánh giá về sản phầm. Kết nối với họ thường xuyên.                 |
| **Potential Loyalist**  | Khách hàng mới, mua nhiều và mua hơn một lần.                                             | Đưa ra các chương trình thành viên/khách hàng trung thành, giới thiệu sản phẩm khác.                   |
| **New Customers**       | Vừa mới mua, nhưng không thường xuyên.                                                    | Cung cấp hỗ trợ ban đầu, giúp họ mua hàng thành công ngay từ lần đầu.               |
| **Promising**           | Người mua gần đây, nhưng chưa chi nhiều.                                             | Xây dựng nhận diện thương hiệu, cung cấp các bản dùng thử miễn phí.                                  |
| **Need Attention**      | Giá trị Recency, Frequency và Monetary trên trung bình, nhưng chưa mua thường xuyên.      | Đưa ra các ưu đãi có thời hạn dựa trên lịch sử mua hàng trước đây. Khuyến khích khách hàng mua hàng trở lại       |
| **About to Sleep**      | Giá trị Recency, Frequency và Monetary dưới trung bình, có thể sẽ mất nếu không kích hoạt. | Giới thiệu sản phẩm phổ biến hoặc ưu đãi giảm giá. Kết nối lại với họ.   |
| **At Risk**             | Chi tiêu nhiều và mua hàng thường xuyên, nhưng đã lâu không mua.                         | Gửi email, zalo OA,... cá nhân hóa để tái kết nối, đề nghị gia hạn.                    |
| **Cannot Lose Them**    | Đã mua với số lượng lớn và thường xuyên, nhưng lâu rồi không quay lại.                    | Thu hút lại bằng các sản phẩm mới hoặc gia hạn, không để mất khách hàng vào tay đối thủ.                     |
| **Hibernating customers** | Mua hàng đã lâu, ít chi tiêu và ít đơn hàng.                                           | Đề nghị các sản phẩm phù hợp khác và ưu đãi đặc biệt. Xây dựng lại giá trị thương hiệu.             |
| **Lost customers**      | Giá trị Recency, Frequency và Monetary thấp nhất.                                        | Khơi gợi sự quan tâm lại bằng các chiến dịch tiếp cận, nếu không có thể bỏ qua.                     |


👉🏻 Based on the insights and findings above, we would recommend the [stakeholder team] to consider the following:  

📌 Key Takeaways:  
✔️ Recommendation 1  
✔️ Recommendation 2  
✔️ Recommendation 3





# **4. Visualization**
***4.1. Histogram distribution***
![image](https://github.com/user-attachments/assets/cc9f48fc-d5a4-4658-ba9f-532934804a53)

![image](https://github.com/user-attachments/assets/dc357274-95c3-4935-a8bf-12ba6a64024b)

![image](https://github.com/user-attachments/assets/bbe15893-eafd-4d4d-951e-3d3cbae5e7f8)

***4.2. Treemap***

*Treemap cho Phân Khúc Khách Hàng*
![image](https://github.com/user-attachments/assets/5ec6066b-b81e-4773-a1fc-9616b4b45944)

*Treemap cho Tổng Doanh Thu*
![image](https://github.com/user-attachments/assets/de8ab72f-12a0-4cd3-a410-2c132ec23807)

***4.3. Scatter plots***

![image](https://github.com/user-attachments/assets/ae887a04-d378-48a3-8123-ed9320b7822c)

![image](https://github.com/user-attachments/assets/c6c0089a-1352-45ac-a26a-49dcc62fda22)
