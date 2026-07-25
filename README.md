# Dự Án BI & Tư Vấn Chiến Lược: Tối Ưu Hóa Vận Hành & Chuỗi bán lẻ FMCG toàn cầu

## Thông Tin Tác Giả

* **Họ và tên:** Nguyễn Ngọc Huỳnh
* **Vai trò:** Data Analyst | Business Intelligence Specialist
* **Thời gian thực hiện:** Tháng 05/2026
* **Công cụ sử dụng:** Power BI, SQL, Excel
> **Nguồn dữ liệu:** Xóm Data | https://dataset.xomdata.com/datasets/schema/fmcg_sales | https://www.facebook.com/groups/1707916343455196

---

## 1. Kiến Trúc Dữ Liệu & Mô Tả Hệ Thống (Data Architecture & Metadata)

Toàn bộ chuỗi bán lẻ FMCG đạt **$3.99 Billion USD** doanh thu qua **6.22M đơn hàng** trong 4 tháng đầu năm, phục vụ **98,759 khách hàng** với AOV duy trì ổn định ở mức **$641.00 USD**.

### 1.1. Từ Điển Dữ Liệu Chi Tiết (Data Dictionary)

#### Bảng `fmcg_sales sales` (Fact Table - Bảng Sự Kiện Doanh Số)
*Lưu trữ 6.8M dòng giao dịch lịch sử bán hàng chi tiết.*

| Tên Cột (Column Name) | Kiểu Dữ Liệu (Data Type) | Cho Phép NULL | Mô Tả Ý Nghĩa (Description) |
| :--- | :--- | :---: | :--- |
| `sales_id` | Int64 / Primary Key | NO | Mã định danh duy nhất của dòng giao dịch. |
| `salesperson_id` | Int64 / Foreign Key | YES | Mã nhân viên thu ngân/bán hàng (liên kết sang `fmcg_sales employees`). |
| `customer_id` | Int64 / Foreign Key | YES | Mã khách hàng mua hàng (liên kết sang `fmcg_sales customers`). |
| `product_id` | Int64 / Foreign Key | YES | Mã sản phẩm (liên kết sang `fmcg_sales products`). |
| `quantity` | Int64 | YES | Số lượng sản phẩm tiêu thụ trên đơn hàng. |
| `discount` | Decimal | YES | Tỷ lệ chiết khấu giảm giá (0 - 1). |
| `total_price` | Decimal /  | NO | .... |
| `sales_date` | Date / Foreign Key | NO | Ngày phát sinh giao dịch (liên kết sang `Dim_Date[Date]`). |
| `transaction_number` | Decimal /  | NO | .... |

---

#### Các Bảng Thứ Nguyên (Dimension Tables)

##### 1. Bảng `fmcg_sales products` (452 dòng - Danh Mục Sản Phẩm)
| Tên Cột (Column Name) | Kiểu Dữ Liệu (Data Type) | Cho Phép NULL | Mô Tả Ý Nghĩa (Description) |
| :--- | :--- | :---: | :--- |
| `product_id` | Int64 / Primary Key | NO | Mã sản phẩm duy nhất. |
| `product_name` | NVarchar(45) | NO | Tên SKU sản phẩm chi tiết. |
| `price` | Decimal | YES | Đơn giá niêm yết nguyên giá ($). |
| `category_id` | Int64 / Foreign Key | YES | Mã ngành hàng (liên kết sang `fmcg_sales categories`). |
| `product_class` | NVarchar(15) | YES | Phân loại đẳng cấp sản phẩm. |
| `vitality_days` | Int64 | YES | Số ngày vòng đời / Tốc độ xoay vòng kho của SKU. |
| `modify_date` | DateTime2 | YES | Ngày cập nhật thông tin sản phẩm. |
| `resistant` | NVarchar(15) | YES | Khả năng bảo quản / độ bền sản phẩm. |
| `is_allergic` | NVarchar(20) | YES | Cảnh báo thành phần gây dị ứng. |

##### 2. Bảng `fmcg_sales categories` (11 dòng - Nhóm Ngành Hàng)
| Tên Cột (Column Name) | Kiểu Dữ Liệu (Data Type) | Cho Phép NULL | Mô Tả Ý Nghĩa (Description) |
| :--- | :--- | :---: | :--- |
| `category_id` | Int64 / Primary Key | NO | Mã nhóm ngành hàng duy nhất. |
| `category_name` | NVarchar(45) | NO | Tên ngành hàng (Confections, Meat, Grain,...). |

##### 3. Bảng `fmcg_sales customers` (98.8K dòng - Khách Hàng)
| Tên Cột (Column Name) | Kiểu Dữ Liệu (Data Type) | Cho Phép NULL | Mô Tả Ý Nghĩa (Description) |
| :--- | :--- | :---: | :--- |
| `customer_id` | Int64 / Primary Key | NO | Mã định danh khách hàng duy nhất. |
| `first_name` / `last_name` | NVarchar(45) | YES | Tên và Họ của khách hàng. |
| `middle_initial` | NVarchar(1) | YES | Tên lót viết tắt. |
| `city_id` | Int64 / Foreign Key | YES | Mã thành phố sinh sống (liên kết sang `fmcg_sales cities`). |
| `address` | NVarchar(90) | YES | Địa chỉ nhà riêng khách hàng. |

##### 4. Bảng `fmcg_sales employees` (23 dòng - Nhân Viên Thu Ngân)
| Tên Cột (Column Name) | Kiểu Dữ Liệu (Data Type) | Cho Phép NULL | Mô Tả Ý Nghĩa (Description) |
| :--- | :--- | :---: | :--- |
| `employee_id` | Int64 / Primary Key | NO | Mã nhân viên duy nhất. |
| `first_name` / `last_name` | NVarchar(45) | YES | Tên và Họ nhân viên. |
| `birth_date` | Date | YES | Ngày tháng năm sinh. |
| `gender` | NVarchar(10) | YES | Giới tính. |
| `hire_date` | DateTime2 | YES | Ngày bắt đầu vào làm việc tại chuỗi (Dùng tính Tenure Days). |
| `city_id` | Int64 / Foreign Key | YES | Mã thành phố làm việc. |

##### 5. Bảng `fmcg_sales cities` & `fmcg_sales countries` (96 Thành Phố & 206 Quốc Gia)
| Tên Cột (Column Name) | Kiểu Dữ Liệu (Data Type) | Cho Phép NULL | Mô Tả Ý Nghĩa (Description) |
| :--- | :--- | :---: | :--- |
| `city_id` | Int64 / Primary Key | NO | Mã thành phố. |
| `city_name` | NVarchar(45) | NO | Tên thành phố phân phối (Tucson, Jackson,...). |
| `zipcode` | NVarchar(10) | YES | Mã bưu chính. |
| `country_id` | Int64 / Foreign Key | YES | Mã quốc gia (liên kết sang `fmcg_sales countries`). |
| `country_name` | NVarchar(45) | NO | Tên quốc gia. |

##### 6. Bảng `Dim_Date` (Bảng Ngày Tháng Hệ Thống)
* Bảng thời gian được tạo riêng bằng DAX để hỗ trợ các hàm Time Intelligence (MoM, YoY).
* Trường cốt lõi: `Date` (Primary Key), `DateKey`, `Day of Month`, `Day of Week Name`, `Day of Week Number`.

---
### 1.2. Mối Quan Hệ Giữa Các Bảng (Entity-Relationship Diagram)

Hệ thống thiết lập các mối quan hệ vật lý **Một - Nhiều ($1 : \infty$)** chuẩn hóa lọc một chiều (Single-direction filtering) từ Bảng Thứ Nguyên sang Bảng Sự Kiện:

1. **Luồng dữ liệu Bán hàng Trung tâm (Fact - Sales):**
   * `Dim_Date[Date]` ($1$) $\rightarrow$ `fmcg_sales sales[sales_date]` ($\infty$)
   * `fmcg_sales products[product_id]` ($1$) $\rightarrow$ `fmcg_sales sales[product_id]` ($\infty$)
   * `fmcg_sales customers[customer_id]` ($1$) $\rightarrow$ `fmcg_sales sales[customer_id]` ($\infty$)
   * `fmcg_sales employees[employee_id]` ($1$) $\rightarrow$ `fmcg_sales sales[salesperson_id]` ($\infty$)

2. **Luồng Chuẩn hóa Phân cấp (Snowflake Hierarchy):**
   * `fmcg_sales categories[category_id]` ($1$) $\rightarrow$ `fmcg_sales products[category_id]` ($\infty$)
   * `fmcg_sales countries[country_id]` ($1$) $\rightarrow$ `fmcg_sales cities[country_id]` ($\infty$)
   * `fmcg_sales cities[city_id]` ($1$) $\rightarrow$ `fmcg_sales customers[city_id]` ($\infty$)
   * `fmcg_sales cities[city_id]` ($1$) $\rightarrow$ `fmcg_sales employees[city_id]` ($\infty$)
---

## 🏗️ Data Architecture & Modeling

Hệ thống dữ liệu được xây dựng chuẩn theo mô hình **Star Schema** tối ưu hiệu năng cho các phép tính Time Intelligence trong DAX:
* **Fact Table:** `fmcg_sales sales` (Giao dịch bán hàng, sản lượng, discount).
* **Dimension Tables:** `dim_date`, `fmcg_sales products`, `fmcg_sales customers`, `fmcg_sales cities`, `Spending_Tiers`.

---

## 📊 Dashboard Breakdown & Analytical Narrative

---

### 🟢 Page 1: Executive Overview (Bức Tranh Tổng Quan 5W1H)

![Page 1 Overview](page1_overview.png)

#### 1. Bức Tranh Tổng Quan & Dòng Tiền (KPIs & Trend)
* **Tháng 1:** Khởi đầu mạnh mẽ đạt **$1.031bn USD** doanh thu (1.607M đơn hàng).
* **Tháng 2:** Doanh thu chạm đáy ở mức **$0.929bn USD** (sụt giảm ~8%).
* **Tháng 3:** Phục hồi ấn tượng vọt lên **$1.032bn USD** (vượt nhẹ mốc Tháng 1).
* **Chất lượng dòng tiền:** **82.48%** doanh thu đến từ hàng nguyên giá, khẳng định sức thu hút tự nhiên của chuỗi, không lệ thuộc vào khuyến mãi.

#### 2. Phân Tích Cơ Cấu Ngành Hàng (Product Mix)
* 🏆 **Core Pillars (Confections & Meat):** Thống trị vị trí Top 1 & Top 2 ($0.51bn & $0.45bn USD).
* 🔄 **Volume Drivers (Produce & Beverages):** Đóng góp sản lượng lớn (7.71M & 6.81M đơn vị), vòng quay kho cực nhanh, giữ vai trò kéo Traffic cho quầy kệ.
* 💎 **Value Drivers (Snails & Dairy):** Sản lượng bán ra khiêm tốn (Dairy 6.28M) nhưng nhờ đơn giá tốt nên duy trì tỷ trọng doanh thu cao, giúp tối ưu chi phí bốc xếp vận chuyển.

#### 3. Hiệu Suất Địa Lý (City Scatter Plot)
* **Vùng Cốt Lõi (Mass Market):** New York, Chicago, Houston tập trung ở dải doanh thu $41M - $43M USD với quy mô đơn hàng lớn, giữ nhịp dòng tiền cho hệ thống.
* **Vùng VIP (High-End Champions):** **Tucson & Jackson** bứt phá cận mốc $45M USD. Jackson đạt AOV kỷ lục toàn chuỗi (**$667.19 USD**) trên tệp đơn hàng đắt đỏ.

---

### 🔵 Page 2: Product & Portfolio Performance (Chiều Sâu Sản Phẩm & Logistics)

![Page 2 Product](page2_product.png)

#### 1. Nghịch Lý Cấu Trúc Giá & Ma Trận Xoay Vòng Kho (Vitality Days)
* **Nghịch lý giá:** Ngành **Grain** có đơn giá cao nhất (gần $60 USD) nhưng doanh thu ở đáy. **Confections & Meat** đơn giá tầm trung ($50 USD) nhưng gánh tổng doanh số toàn chuỗi -> *Tăng trưởng nhờ quy mô sản phẩm phổ thông.*
* **Portfolio Matrix:** Confections & Meat nằm ở ô **Winners** với thời gian lưu kho cực ngắn (**Vitality Days < 20-22 ngày**). Grain nằm ở ô **Hidden Gems** (> 35-40 ngày).

#### 2. Phân Bổ Tồn Kho Địa Phương (Top 10 / Bottom 10 Dynamic SKU Tooltip)
* **Thị trường Sức mua cao (Top 10 - Tucson, Sacramento, Jackson):** Bật Tooltip soi **Top 5 Best-Selling SKUs** -> Phòng Mua hàng ưu tiên cấp thêm quota, tự tin nhập kho số lượng lớn.
* **Thị trường Sức mua yếu (Bottom 10 - Omaha, Long Beach, Fort Worth):** Bật Tooltip màu cam soi **Top 5 Core Essential SKUs** -> Chỉ duy trì **Min Safety Stock** cho 5 SKU này, mạnh tay điều chuyển/cắt giảm mã thừa để giải phóng dòng tiền.

#### 3. Chẩn Đoán Biến Động MoM (Month-over-Month Matrix)
| Mốc Thời Gian | Biến Động Doanh Thu | Biến Động Sản Lượng | Chẩn Đoán Nguyên Nhân |
| :--- | :--- | :--- | :--- |
| **Tháng 2** | `-$101.53M USD` | `-2,037,611` sản phẩm | Do 2 ngành Winners: Confections (-$13.79M) & Meat (-$11.77M) sụt giảm |
| **Tháng 3** | `+$102.99M USD` | `+2,068,102` sản phẩm | Bùng nổ bù đắp trọn vẹn đáy Tháng 2 (Confections +$13.57M, Meat +$12.16M) |
| **Tháng 4** | `-$34.93M USD` | `-701,479` sản phẩm | Chu kỳ điều chỉnh nhẹ của chuỗi cung ứng |

> 💡 **Kết luận Chẩn đoán:** Sự đối ứng đối xứng chuẩn xác về cả số tiền lẫn sản lượng giữa Tháng 2 và 3 khẳng định chuỗi **KHÔNG BỊ MẤT THỊ PHẦN**. Đây thuần túy là **Hiệu ứng Bullwhip (Tái đặt hàng & xả kho xen kẽ)**.

---

### 🟣 Page 3: Customer & Employee Performance (Sức Khỏe Khách Hàng & Vận Hành)

![Page 3 Customer](page3_customer.png)

#### 1. Sức Khỏe Tệp Khách Hàng (Customer Health)
* **Zero Churn Rate:** Dù sản lượng Tháng 2 giảm, quy mô active users vẫn đứng yên ở mốc **98,759 người (~99K)** qua cả 4 tháng. Khách hàng không bỏ đi.
* **Customer Value Quartiles:** Tệp khách hàng chia đều 25% cho 4 nhóm chi tiêu. Nhóm VIP (High Spenders) đóng góp dòng tiền lớn nhất.

#### 2. Hiệu Suất Nhân Sự & Kỷ Luật Thu Ngân (Employee Compliance)
* **Chuẩn hóa Năng suất:** 23 nhân viên đạt average **$173.45M USD/người**. Thâm niên (Tenure) không tạo sự chênh lệch -> *Hệ thống bán hàng được quy chuẩn hóa rất tốt.*
* **Kỷ luật Chiết khấu 100%:** Tỷ lệ discount của 23 thu ngân nằm gọn trong dải **3.006% - 3.026%** (gần như phẳng tuyệt đối so với mốc chuẩn 3.00%), tuyệt đối không có gian lận voucher làm thất thoát dòng tiền.

---

## 🛠️ Technical Highlights (DAX Engineering)

Một số công thức DAX chính được xây dựng để xử lý Time Intelligence và Dynamic Segmentation:

### 1. Hàm Tính Doanh Thu Tháng Trước (Bảo toàn Filter Context với `KEEPFILTERS`)
```dax
Prior Month Revenue = 
VAR CurrentMonth = SELECTEDVALUE('dim_date'[Month Number])
VAR CurrentYear = SELECTEDVALUE('dim_date'[Year], MAX('dim_date'[Year]))
VAR TargetMonth = IF(CurrentMonth = 1, 12, CurrentMonth - 1)
VAR TargetYear = IF(CurrentMonth = 1, CurrentYear - 1, CurrentYear)
RETURN
IF(
    HASONEVALUE('dim_date'[Month Number]) && NOT(ISBLANK(CurrentMonth)) && CurrentMonth > 1,
    CALCULATE(
        [Total Revenue],
        REMOVEFILTERS('dim_date'),
        KEEPFILTERS('dim_date'[Month Number] = TargetMonth),
        KEEPFILTERS('dim_date'[Year] = TargetYear)
    ),
    BLANK()
)
