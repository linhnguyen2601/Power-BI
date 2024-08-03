# 

## Business Requirements:

Case study này tập trung vào việc phân tích dữ liệu để đưa ra những nhận định chính xác về hành vi churn của khách hàng và những yếu tố tác động đến quyết định này. 

Những thông tin phân tích từ case study sẽ cung cấp cái nhìn về các xu hướng và yếu tố quyết định của khách hàng. Việc nắm rõ các nguyên nhân dẫn đến sự churn sẽ giúp các ngân hàng tối ưu hóa chiến lược giữ chân khách hàng, từ đó cải thiện trải nghiệm khách hàng và tăng cường mối quan hệ lâu dài. 

Thông qua việc phân tích dữ liệu và đưa ra những insights cụ thể, case study này sẽ đóng góp cho sự thành công của các chiến lược marketing và dịch vụ khách hàng của ngành ngân hàng, giúp họ duy trì và phát triển cơ sở khách hàng một cách hiệu quả và bền vững. 

 ## Data Dictionary:

Dataset gồm 10,000 dòng:

 CustomerId: chứa các giá trị ngẫu nhiên và không ảnh hưởng đến việc khách hàng rời khỏi ngân hàng. 

Surname: Họ của khách hàng không ảnh hưởng đến quyết định rời khỏi ngân hàng của họ. 

CreditScore: có thể ảnh hưởng đến tỷ lệ mất khách hàng, vì khách hàng có điểm tín dụng cao hơn sẽ ít có khả năng rời bỏ ngân hàng. 

Credit score: 

- Excellent: 800–850 

- Very Good: 740–799 

- Good: 670–739 

- Fair: 580–669 

- Poor: 300–579 

Geography: vị trí của khách hàng có thể ảnh hưởng đến quyết định rời khỏi ngân hàng của họ. 

Gender: thật thú vị khi tìm hiểu xem liệu giới tính có đóng vai trò gì trong việc khách hàng rời khỏi ngân hàng hay không. 

Age: điều này chắc chắn có liên quan, vì khách hàng lớn tuổi ít có khả năng rời bỏ ngân hàng hơn những khách hàng trẻ tuổi. 

Tenure: đề cập đến số năm mà khách hàng đã là khách hàng của ngân hàng. Thông thường, khách hàng lớn tuổi trung thành hơn và ít có khả năng rời khỏi ngân hàng. 

Balance: cũng là một chỉ số rất tốt về tỷ lệ mất khách hàng, vì những người có số dư cao trong tài khoản ít có khả năng rời khỏi ngân hàng hơn so với những người có số dư thấp hơn. 

NumOfProducts: là số lượng sản phẩm mà khách hàng đã mua thông qua ngân hàng. 

Biểu thị khách hàng có thẻ tín dụng hay không. Cột này cũng có liên quan, vì những người có thẻ tín dụng ít có khả năng rời khỏi ngân hàng hơn. (credit card holder, non credit card holder) 

isActiveMember: khách hàng tích cực ít có khả năng rời khỏi ngân hàng 

Estimated Salary: Tương tự như số dư, những người có mức lương thấp có nhiều khả năng rời khỏi ngân hàng hơn so với những người có mức lương cao hơn. 

Exited: Khách hàng có rời ngân hàng hay không. (Retain, Exit) 

Bank DOJ: Ngày mở tài khoản 

## Data Model: xây dựng bảng Dim, bảng Fact
- Tạo bảng Active Customer gồm cột ActiveCategory & ID
- Tạo bảng Credit Card gồm cột Credit & ID
- Tạo bảng Customer gồm CustomerID & Surname
- Tạo bảng Exit Retain gồm Exit_Category & Exit ID
- Tạo bảng Location gồm GeographyLocation & Location ID

![image](https://github.com/user-attachments/assets/e3d50fd8-d72e-4945-a93c-ed0e01806b73)

Tạo bảng DIM_DATE

```
DIM_DATE = ADDCOLUMNS( CALENDAR( FIRSTDATE(Bank_Churn[Bank DOJ]),
 LASTDATE(Bank_Churn[Bank DOJ]) ),
  "Day", DAY([Date]),
  "Month", MONTH([Date]),
  "Year", YEAR([Date]) ,
   "Month Name",
   FORMAT([Date], "MMM") )
```
Tạo cột Credit Type:

```
Credit type = SWITCH(TRUE(),
 Bank_Churn[CreditScore] >= 800 && Bank_Churn[CreditScore] <= 850, "Excellent" ,
 Bank_Churn[CreditScore] >= 740 && Bank_Churn[CreditScore] <= 799, "Very good" ,
 Bank_Churn[CreditScore] >= 670 && Bank_Churn[CreditScore] <= 739, "Good" ,
 Bank_Churn[CreditScore] >= 580 && Bank_Churn[CreditScore] <= 699, "Fair" ,
 Bank_Churn[CreditScore] >= 300 && Bank_Churn[CreditScore] <= 579, "Poor" ,
 Bank_Churn[CreditScore] < 300, "Very Poor" )
```

## Tạo các Measures

Measure Active Customer đếm tất cả các khách hàng là Active Member

```
Acitive customers = CALCULATE(COUNT(Bank_Churn[CustomerId]), 'Active Customer'[ActiveCategory] = "Active Member")
```

Measure CustomerID đếm các khách hàng trong bảng BankChurn

```
Total customer = DISTINCTCOUNT(Bank_Churn[CustomerId])
```

Measure Inactive Customer

```
Inactive customer = [Total customer] - [Acitive customers]
```

Measure Credit Card đếm các KH là Credit Card Holder

```
Credit Card holder = CALCULATE(COUNT(Bank_Churn[CustomerId]), Credit_card[Category] = "credit card holder")
```

Measure đếm các KH không có Credit Card

```
Non Credit Card holder = CALCULATE(COUNT(Bank_Churn[CustomerId]), Credit_card[Category] = "non credit card holder")
```

Đếm các KH exit
```
Exit Customers = CALCULATE([Total customer], 'Exit Retain'[ExitCategory] = "Exit")
```

Đếm các KH retain:
```
Retain Customer = CALCULATE([Total customer], 'Exit Retain'[ExitCategory] = "Retain")
```

```
Previous month exit customer = CALCULATE([Exit Customers], PREVIOUSMONTH(DIM_DATE[Date]))
```


Tạo measure tính churn rate
```
Churn rate = [Inactive customer]/[Total customer]
```

## Tạo cột Credit Type



