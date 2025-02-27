# Python_RFM-project
# **1. EDA**
***1.1. Explore data***

***1.2. Tạo bảng thống kê đơn hủy***

![image](https://github.com/user-attachments/assets/5631a1e9-36fa-4623-bb41-3c9defa68160)

*=> Số lượng đơn bị hủy là 9288*
*Tính Số lượng giao dịch bị hủy và tỷ lệ phần trăm*
```python
c1 = data['order_cancelled'].value_counts()[1]
c2 = data.shape[0]
print("Number of cancelled transactions: ", c1)
print('Percent of orders cancelled: {}/{} ({:.2f}%) '.format(c1, c2, c1/c2*100))
```
*Number of cancelled transactions:  9288*
*Percent of orders cancelled: 9288/541909 (1.71%)*

***1.3. Clean data, tạo df_Transaction***
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
