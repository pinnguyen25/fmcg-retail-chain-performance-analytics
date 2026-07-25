# Dự Án BI & Tư Vấn Chiến Lược: Tối Ưu Hóa Vận Hành & Chuỗi bán lẻ FMCG toàn cầu

## Thông Tin Tác Giả

* **Họ và tên:** Nguyễn Ngọc Huỳnh
* **Vai trò:** Data Analyst | Business Intelligence Specialist
* **Thời gian thực hiện:** Tháng 05/2026
* **Công cụ sử dụng:** Power BI, SQL, Excel
> **Nguồn dữ liệu:** Xóm Data | https://dataset.xomdata.com/datasets/schema/fmcg_sales | https://www.facebook.com/groups/1707916343455196

---

## 1. Kiến Trúc Dữ Liệu & Mô Tả Hệ Thống (Data Architecture & Metadata)

Hệ thống cơ sở dữ liệu được thiết kế theo mô hình **Sơ đồ hình sao mở rộng (Extended Star Schema / Snowflake Hybrid)**, tối ưu hóa tốc độ truy vấn và đảm bảo tính toàn vẹn dữ liệu trên các công cụ Business Intelligence. Hệ thống gồm 1 bảng sự kiện trung tâm (`fmcg_sales sales`), 6 bảng thứ nguyên (`fmcg_sales products, fmcg_sales employees, fmcg_sales customers, fmcg_sales cities, fmcg_sales countries, fmcg_sales categories`).

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
| `total_price` | Decimal /  | NO | Tổng giá trị thực thu của dòng sản phẩm sau chiết khấu ($). |
| `sales_date` | Date / Foreign Key | NO | Ngày phát sinh giao dịch (liên kết sang `Dim_Date[Date]`). |
| `transaction_number` | NVarchar(25) | YES | Mã số hóa đơn / mã giao dịch. |

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
| `first_name` | NVarchar(45) | YES | Tên của khách hàng. |
| `middle_initial` | NVarchar(1) | YES | Tên lót viết tắt. |
| `last_name` | NVarchar(45) | YES | Họ của khách hàng. |
| `city_id` | Int64 / Foreign Key | YES | Mã thành phố sinh sống (liên kết sang `fmcg_sales cities`). |
| `address` | NVarchar(90) | YES | Địa chỉ nhà riêng khách hàng. |

##### 4. Bảng `fmcg_sales employees` (23 dòng - Nhân Viên Thu Ngân)
| Tên Cột (Column Name) | Kiểu Dữ Liệu (Data Type) | Cho Phép NULL | Mô Tả Ý Nghĩa (Description) |
| :--- | :--- | :---: | :--- |
| `employee_id` | Int64 / Primary Key | NO | Mã nhân viên duy nhất. |
| `first_name` | NVarchar(45) | YES | Tên nhân viên. |
| `middle_initial` | Nvarchar(1) | YES | Tên lót viết tắt. |
| `last_name` | NVarchar(45) | YES | Tên nhân viên. |
| `birth_date` | Date | YES | Ngày tháng năm sinh. |
| `gender` | NVarchar(10) | YES | Giới tính. |
| `hire_date` | DateTime2 | YES | Ngày bắt đầu vào làm việc tại chuỗi. |
| `city_id` | Int64 / Foreign Key | YES | Mã thành phố làm việc. |

##### 5. Bảng `fmcg_sales cities` (96 dòng - Thành Phố)
| Tên Cột (Column Name) | Kiểu Dữ Liệu (Data Type) | Cho Phép NULL | Mô Tả Ý Nghĩa (Description) |
| :--- | :--- | :---: | :--- |
| `city_id` | Int64 / Primary Key | NO | Mã thành phố. |
| `city_name` | NVarchar(45) | NO | Tên thành phố phân phối (Tucson, Jackson,...). |
| `zipcode` | NVarchar(10) | YES | Mã bưu chính. |
| `country_id` | Int64 / Foreign Key | YES | Mã quốc gia (liên kết sang `fmcg_sales countries`). |

##### 6. Bảng `fmcg_sales countries` (206 dòng - Quốc Gia)
| Tên Cột (Column Name) | Kiểu Dữ Liệu (Data Type) | Cho Phép NULL | Mô Tả Ý Nghĩa (Description) |
| :--- | :--- | :---: | :--- |
| `country_id` | Int64 / Primary Key | YES | Mã quốc gia duy nhất. |
| `country_name` | NVarchar(45) | NO | Tên quốc gia. |
| `country_code` | Nvarchar(2) | YES | Mã quốc gia 2 ký tự (ISO Code). |

---
### 1.2. Mối Quan Hệ Giữa Các Bảng (Entity-Relationship Diagram)

Hệ thống thiết lập các mối quan hệ vật lý **Một - Nhiều ($1 : \infty$)** chuẩn hóa lọc một chiều (Single-direction filtering) từ Bảng Thứ Nguyên sang Bảng Sự Kiện:

1. **Luồng dữ liệu Bán hàng Trung tâm (Fact - Sales):**
   * `fmcg_sales products[product_id]` ($1$) $\rightarrow$ `fmcg_sales sales[product_id]` ($\infty$) *(Active)*
   * `fmcg_sales customers[customer_id]` ($1$) $\rightarrow$ `fmcg_sales sales[customer_id]` ($\infty$) *(Active)*
   * `fmcg_sales employees[employee_id]` ($1$) $\rightarrow$ `fmcg_sales sales[salesperson_id]` ($\infty$) *(Active)*

2. **Luồng Chuẩn hóa Phân cấp (Snowflake Hierarchy):**
   * `fmcg_sales categories[category_id]` ($1$) $\rightarrow$ `fmcg_sales products[category_id]` ($\infty$) *(Active)*
   * `fmcg_sales countries[country_id]` ($1$) $\rightarrow$ `fmcg_sales cities[country_id]` ($\infty$) *(Active)*
   * `fmcg_sales cities[city_id]` ($1$) $\rightarrow$ `fmcg_sales customers[city_id]` ($\infty$) *(Active)*
   * `fmcg_sales cities[city_id]` ($1$) $\dashrightarrow$ `fmcg_sales employees[city_id]` ($\infty$) *(Inactive Relationship - Nét đứt)*
---

## 2. Quy Trình Giải Quyết Vấn Đề Theo Khung 7 Bước McKinsey (McKinsey 7-Step Framework)

### Bước 1: Phát biểu Vấn đề (Define the Problem)
* **Bối cảnh:** Chuỗi bán lẻ FMCG đạt tổng doanh thu **$3.99 Billion USD** qua **6.22 Triệu đơn hàng** trong 4 tháng đầu năm, phục vụ **98,759 khách hàng** với AOV đạt **$641.00 USD**.
* **Vấn đề cốt lõi:** Tháng 2 ghi nhận cú **sụt giảm doanh thu nghiêm trọng (-$101.53M USD, tương ứng -8.2%)** và sản lượng tiêu thụ bị kéo lùi **-2.03M sản phẩm**.
* **Câu hỏi chiến lược:** Cú rơi Tháng 2 là do chuỗi bị mất thị phần/khách hàng rời bỏ (Churn) hay do đứt gãy chuỗi cung ứng? Làm thế nào để phân bổ tồn kho tối ưu tới từng thành phố?

### Bước 2: Cấu trúc hóa Vấn đề (Structure the Problem) - Nguyên tắc MECE & Issue Tree
Dòng doanh thu được phân rã theo công thức toán học và chuyển hóa thành cây vấn đề:
$$\text{Doanh thu} = \text{Số lượng đơn hàng} \times \text{Sản lượng/Đơn} \times \text{Đơn giá trung bình (ASP)} \times (1 - \text{Discount})$$

### Bước 3: Ưu tiên hóa các Nhánh Phân tích (Prioritize Issues)
* **Giả thuyết 1:** Sụt giảm Tháng 2 do nhóm sản phẩm chủ lực (Winners) bị đứt gãy nguồn cung hoặc đại lý xả kho định kỳ (Hiệu ứng Bullwhip).
* **Giả thuyết 2:** Sụt giảm do khách hàng ngưng quay lại chuỗi (Customer Churn).
* **Giả thuyết 3:** Sụt giảm do thất thoát dòng tiền từ việc lạm dụng khuyến mãi/chiết khấu của đội ngũ thu ngân.

### Bước 4: Lập Kế hoạch Phân tích & Triển khai (Issue Analysis & Data Gathering)
Sử dụng SQL để kiểm định dữ liệu thô. Xây dựng mô hình đo lường DAX nâng cao trong Power BI, bóc tách đa chiều mối liên hệ giữa các biến số chiến lược theo: **Thị trường Địa phương — Tốc độ Lưu kho Sản phẩm — Sức khỏe Vận hành Nhân sự & Khách hàng**.

### Bước 5: Phân tích sâu & Diễn giải Dữ liệu (Deep-Dive Interpretation)

#### 5.1. Phân hóa Hiệu suất Địa lý & Phân khúc Giỏ hàng (City & Regional Performance)

* **Nhóm thị trường bức phá doanh thu (High-End Champions) — Tucson & Jackson:**
  * *Tucson:* Dẫn đầu quy mô doanh thu toàn chuỗi (đạt sát mốc **$45M USD**), bán mạnh nhất dòng sản phẩm Bánh kẹo & Đồ uống.
  * *Jackson:* Lập kỷ lục AOV cao nhất hệ thống (**$667.19 USD/đơn**, vượt xa AOV trung bình $641.00 USD). Mặc dù lượng đơn hàng khá khiêm tốn, khách hàng tại đây có xu hướng đóng góp các đơn hàng giỏ cao cấp (High-value baskets).
  * *Insight:* Hai thị trường này có mật độ khách hàng VIP cao, ít nhạy cảm về giá, là động lực giữ vững biên lợi nhuận cho toàn chuỗi.

* **Nhóm thị trường cốt lõi mô Lớn (Mass Market Anchors) — New York, Chicago, Houston:**
  * Nằm tập trung ở dải doanh thu **$41M - $43M USD** với quy mô đơn hàng rất lớn.
  * *Insight:* Đây là vùng tạo guồng quay dòng tiền ổn định (Cash Generator), đảm bảo đầu ra sản lượng ổn định cho các nhà cung ứng.

* **Nhóm thị trường tiêu thụ khiêm tốn (Low-Volume Markets) — Omaha, Long Beach, Fort Worth:**
  * Doanh thu nằm ở nhóm đáy (dưới **$38M - $39M USD**), tệp khách hàng phân tán.
  * *Insight:* Sức mua yếu do danh mục sản phẩm chưa đáp ứng đúng gu tiêu dùng địa phương. Cần rà soát loại bỏ các mã SKU ứ đọng.

#### 5.2. Ma Trận Danh Mục Sản Phẩm & Tốc Độ Lưu Kho (Product Portfolio & Vitality Matrix)

* **Trụ cột Gánh Team (Winners) — Confections & Meat:**
  * Đóng góp doanh thu khổng lồ, lần lượt đạt **$512.5M USD** (Confections) và **$453.7M USD** (Meat).
  * Tốc độ xoay vòng kho vọt trội: **Vitality Days ngắn nhất hệ thống (< 20 - 22 ngày)**. Hàng nhập về kho được đẩy đi ngay.
  * *Rủi ro:* Vì đóng góp tới ~24% tổng doanh thu chuỗi, bất kỳ sự gián đoạn nguồn cung nào từ nhóm này cũng sẽ kéo lùi toàn bộ chỉ số tăng trưởng.

* **Sản phẩm Dẫn dắt Sản lượng (Volume Drivers) — Produce & Beverages:**
  * Đạt sản lượng tiêu thụ kỷ lục: Produce (**7.71M sản phẩm**) và Beverages (**6.81M sản phẩm**).
  * *Đặc đặc:* Đơn giá bình dân (~$40 - $45 USD) nên doanh thu ở mức trung bình, nhưng đóng vai trò là "chất dẫn" thu hút lượt ghé thăm (Traffic Generator) cho siêu thị.

* **Sản phẩm Tối ưu Biên Lợi nhuận (Value Drivers) — Snails & Dairy:**
  * Sản lượng tiêu thụ khiêm tốn (Dairy chỉ đạt 6.28M sản phẩm) nhưng đạt doanh thu tốt (**$326.3M - $342.9M USD**) nhờ đơn giá cao.
  * *Insight:* Nhóm hàng tối ưu chi phí vận chuyển & bốc xếp — bán ít hơn nhưng thu về lượng tiền mặt rất hiệu quả.

* **Nhóm Ẩn số Bán chậm (Hidden Gems) — Grain:**
  * Có đơn giá trung bình cao nhất hệ thống (gần **$60 USD/đơn vị**), nhưng doanh thu đứng chót bảng (**$298.3M USD**).
  * Thời gian lưu kho kéo dài (**Vitality Days > 35 - 40 ngày**), làm ứ đọng vốn lưu động.

#### 5.3. Chẩn Đoán Biến Động MoM & Kiểm Định Hiệu Ứng Bullwhip

* **Tháng 2 — Chạm đáy ngắn hạn (-8.2% Revenue):**
  * Tổng doanh thu giảm **-$101.53M USD** (tương ứng mất **-2.03M sản phẩm**).
  * Mức giảm tập trung chủ yếu vào 2 ngành Winners: **Confections (giảm -$13.79M USD / -262K SP)** và **Meat (giảm -$11.77M USD / -228K SP)**.
* **Tháng 3 — Phục hồi đối ứng hoàn toàn (+11.1% Revenue):**
  * Doanh thu bùng nổ vọt lên **+$102.99M USD** (bù đắp 100% lượng thâm hụt Tháng 2).
  * Hai ngành gánh team hồi phục ấn tượng: **Confections (tăng +$13.57M USD)** và **Meat (tăng +$12.16M USD)**.
* **Kết luận chẩn đoán (Root Cause Analysis):**
  * *Zero Churn Rate:* Quy mô Active Members giữ phẳng ở mốc **98,759 người** qua cả 4 tháng.
  * *Tuân thủ 100% Chiết khấu:* Tỷ lệ discount của 23 thu ngân giữ ổn định ở dải **3.006% - 3.026%** (không lạm dụng chiết khấu).
  * **Bản chất vấn đề:** Cú rơi Tháng 2 không đến từ việc mất khách hàng hay gian lận chiết khấu, mà thuần túy là **Hiệu ứng Bullwhip (chu kỳ tái đặt hàng và xả kho định kỳ của các đại lý)**.

### Bước 6: Tổng hợp Kết luận (Synthesize Findings)

1. **Sức khỏe thương hiệu rất tốt:** 82.48% doanh thu đến từ hàng nguyên giá, chứng tỏ sức hút tự nhiên của sản phẩm, không phụ thuộc vào chương trình giảm giá.
2. **Điểm nghẽn vận hành nằm ở chu kỳ Logistics:** Doanh thu biến động mạnh do nhịp tái đặt hàng bị lệch pha ở 2 ngành chủ lực (Confections & Meat).
3. **Bài toán phân bổ tồn kho địa phương:** Các thị trường có sự phân hóa rõ rệt về gu tiêu dùng. Việc phân bổ đồng đều danh mục hàng hóa đang gây ra tình trạng thừa kho ở các thị trường yếu (Omaha) và nguy cơ thiếu kho ở các thị trường VIP (Tucson, Jackson).

### Bước 7: Đề xuất Giải pháp Hành động Chiến lược (Actionable Recommendations)

#### 1. "Làm mịn" chu kỳ cung ứng (Supply Smoothing & Bullwhip Dampening)
* **Phòng Logistics:** Áp dụng mô hình **VMI (Vendor Managed Inventory)** đối với các nhà cung cấp ngành Confections & Meat.
* **Hành động cụ thể:** Chia nhỏ lịch giao hàng theo tuần thay vì gom đơn theo tháng, giúp san đều áp lực vận tải và triệt tiêu biến động âm $100M USD lặp lại trong các chu kỳ sau.

#### 2. Chiến lược quản trị tồn kho phân hóa theo mức SKU địa phương (SKU-Level Inventory Strategy)
* **Đối với thị trường sức mua cao (Top 10 Cities — Tucson, Sacramento, Jackson):**
  * Cung cấp thêm **15% - 20% Quota tồn kho** cho **Top 5 Best-Selling SKUs** (như *Hot Chocolate*, *Pail With Metal Handle*).
  * Đảm bảo mức sẵn có trên kệ (On-shelf availability) đạt 99% để tối đa hóa doanh thu từ tệp khách hàng VIP.
* **Đối với thị trường tiêu thụ yếu (Bottom 10 Cities — Omaha, Long Beach, Fort Worth):**
  * Thực hiện **Rút gọn danh mục (SKU Rationalization)**: Cắt giảm các mã hàng bán chậm thuộc nhóm Grain.
  * **Chỉ duy trì định mức tồn kho an toàn tối thiểu (Min Safety Stock)** cho **Top 5 Core Essential SKUs** hiếm hoi còn tiêu thụ tại các địa phương này.
  * Điều chuyển lượng hàng dư thừa sang các kho khu vực Top 10 để giải phóng vốn lưu động.

#### 3. Tối Ưu Hóa Kênh Bán Hàng & Chăm Sóc Khách Hàng VIP
* **Phòng Marketing & Sales:** Thiết lập chương trình **VIP Customer Loyalty** riêng cho hai thị trường Jackson và Tucson.
* **Hành động cụ thể:** Thiết kế các gói Combo / Gift Basket cho nhóm hàng Snails & Confections để tiếp tục đẩy cao giá trị đơn hàng AOV vượt mốc **$667.19 USD**.

---

## 3. Kiến Trúc Hệ Thống Báo Cáo Chiến Lược (Dashboard Purpose & Business Value)

Hệ thống báo cáo được thiết kế theo tư duy phân tầng thông tin quản trị (**Top-down Approach**), đi từ bức tranh tài chính tổng quan vĩ mô đến các góc nhìn bóc tách chi tiết theo thực tế vận hành. 

---

### Trang 1: FMCG Retail Chain — Executive Overview

![Trang 1 Executive Overview](page1_overview.png)

#### A. Mục đích chiến lược
* Cung cấp một góc nhìn toàn cảnh tức thời (At-a-glance) về "sức khỏe" tài chính, quy mô đơn hàng và chỉ số dòng tiền cốt lõi của toàn hệ thống bán lẻ.
* Đóng vai trò là công cụ giám sát cấp cao dành cho Ban Giám Đốc (CEO/CFO), giúp phát hiện sớm các tín hiệu bất thường hoặc điểm gãy đột ngột của doanh số theo chuỗi thời gian.

#### B. Trang này làm những gì?
* **Theo dõi & hợp nhất 4 chỉ số sinh mệnh:** Doanh thu ($3.99Bn USD), Khối lượng đơn hàng (6.22M đơn), Giá trị đơn hàng trung bình (AOV $641.00 USD) và Quy mô tệp khách hàng (98,759 người).
* **Kiểm định chất lượng dòng tiền (Sales Type Analysis):** Bóc tách tỷ trọng giữa dòng tiền nguyên giá (Full Price Items) và dòng tiền từ sản phẩm khuyến mãi (Discounted Items).
* **Phân rã cơ cấu danh mục & vị thế địa lý:** Phân tích song song Doanh thu vs Sản lượng của 11 ngành hàng và định vị hiệu suất AOV vs Doanh thu của 96 thành phố trên Ma trận Scatter Plot (City Performance).

#### C. Ý nghĩa kinh doanh
* **Khẳng định độ bền vững của thương hiệu:** Phơi bày thực tế chất lượng dòng tiền khi **82.48% doanh thu ($3.29Bn USD)** đến từ hàng nguyên giá. Ý nghĩa dữ liệu khẳng định chuỗi bán lẻ có sức hút tự nhiên rất lớn, tăng trưởng lành mạnh và không bị phụ thuộc vào các chương trình xả hàng giảm giá.
* **Định vị thị trường VIP để tối ưu hóa biên lợi nhuận:** Phát hiện hai thị trường ngôi sao **Tucson & Jackson** bứt phá mốc doanh thu $45M USD, trong đó Jackson đạt kỷ lục AOV cao nhất chuỗi (**$667.19 USD/đơn**). Kết quả này giúp các sếp khoanh vùng tệp khách hàng cao cấp để triển khai các gói sản phẩm độc quyền.

---

### Trang 2: FMCG Retail Chain — Product & Inventory Performance (Tối Ưu Danh Mục & Chuỗi Cung Ứng)

![Trang 2 Product Performance](page2_product.png)

#### A. Mục đích chiến lược
* Bản đồ hóa và chẩn đoán chuyên sâu hiệu suất danh mục sản phẩm kết hợp với tốc độ lưu kho (**Vitality Days**).
* Cung cấp công cụ điều phối tồn kho chính xác tới cấp độ SKU cho phòng Logistics & Mua hàng, giải mã tận gốc nguyên nhân biến động doanh thu theo từng tháng.

#### B. Trang này làm những gì?
* **Kiểm toán hiệu suất cấu trúc giá:** Đối chiếu trực diện giữa đơn giá trung bình (Avg Unit Price) và tổng doanh thu để tìm ra nghịch lý đóng góp dòng tiền.
* **Định vị ma trận lưu kho:** Phân định 11 ngành hàng lên Ma trận 4 ô (*Winners, Stable Pillars, High-risk, Hidden Gems*) dựa trên hai trục: Doanh thu và Tốc độ xoay vòng kho (Vitality Days).
* **Điều hướng nhu cầu địa phương linh hoạt (Geographic Market Demand):** Tích hợp bộ lọc tương tác **Top 10 / Bottom 10** gán kèm **Report Page Tooltip** tự động soi **Top 5 SKUs** cốt lõi cho từng thành phố.
* **Chẩn đoán biến động MoM:** Bóc tách chênh lệch doanh thu và sản lượng giữa tháng hiện tại và tháng liền trước ở cấp độ ngành hàng.

#### C. Ý nghĩa kinh doanh
* **Giải mã bản chất cú sụt giảm Tháng 2 (-$101.53M USD):** Bảng chẩn đoán MoM phơi bày nguyên nhân giảm sâu ở Tháng 2 đến từ 2 ngành gánh team: Confections (-$13.79M USD) và Meat (-$11.77M USD). Việc Tháng 3 bật tăng trở lại trọn vẹn (+$102.99M USD) chứng minh đây là **Hiệu ứng Bullwhip (chu kỳ xả kho & tái đặt hàng của đại lý)** chứ doanh nghiệp không bị mất thị phần.
* **Tối ưu hóa vốn lưu động & chống cháy hàng cục bộ:** Chỉ ra chính xác các mặt hàng cần bơm thêm Quota nhập kho tại các thành phố sức mua lớn (Tucson, Jackson), đồng thời cho phép phòng Logistics mạnh tay điều chuyển các mã hàng ứ đọng ở nhóm Hidden Gems (Grain - lưu kho > 35-40 ngày) khỏi các thị trường yếu (Omaha).
---

### Trang 3: FMCG Retail Chain — Customers & Employees Health (Sức Khỏe Khách Hàng & Kỷ Luật Vận Hành)

![Trang 3 Customers Employees Health](page3_customer.png)

#### A. Mục đích chiến lược
* Đánh giá mức độ gắn kết dài hạn của tệp khách hàng (Customer Retention) kết hợp với việc kiểm soát kỷ luật vận hành và năng suất bán hàng của đội ngũ nhân sự tuyến đầu (23 Thu ngân/Sales).
* Đảm bảo bộ máy vận hành bên dưới tuân thủ nghiêm túc các quy chuẩn kinh doanh, triệt tiêu rủi ro thất thoát dòng tiền từ các điểm bán lẻ.

#### B. Trang này làm những gì?
* **Đo lường độ ổn định tệp thành viên (Membership Stability):** Giám sát biến động số lượng đơn hàng song song với đường quy mô khách hàng Active theo từng tháng.
* **Phân khúc giá trị khách hàng (Customer Value Segmentation):** Phân chia tệp 98.759K khách hàng thành 4 nhóm Tứ phân vị chi tiêu (*High Spenders VIP, Medium-High, Medium-Low, Low Spenders*).
* **Kiểm toán năng suất & thâm niên nhân sự (Employee Productivity vs Tenure):** Đối chiếu Doanh số Rolling 30 ngày với Số ngày làm việc (Tenure Days) của 23 nhân viên kinh doanh.
* **Giám sát kỷ luật chiết khấu thu ngân (Cashier Discount Compliance):** Đo lường chi tiết tỷ lệ discount thực tế áp dụng tại ca làm việc của từng thu ngân so với trần quy định 3.00%.

#### C. Ý nghĩa kinh doanh
* **Xác nhận trạng thái Zero Churn Rate:** Dữ liệu chỉ ra dù sản lượng Tháng 2 giảm, **đường quy mô khách hàng Active vẫn nằm ngang tuyệt đối ở mốc 98,759 người (~99K)** qua cả 4 tháng. Điều này khẳng định 100% khách hàng không quay lưng với chuỗi, sụt giảm doanh số hoàn toàn do nhịp mua hàng tạm thời.
* **Khẳng định năng lực quy chuẩn hóa bộ máy (Standardized Operations):** Biểu đồ Năng suất cho thấy dù thâm niên từ 500 đến 3,000 ngày, doanh số đóng góp của 23 nhân viên đều nằm ngang ở dải tiệm cận **$30M USD/người**. Ý nghĩa chứng minh quy trình đào tạo và vận hành ca bán hàng cực kỳ xuất sắc.
* **Kiểm soát rủi ro gian lận tài chính:** Tỷ lệ giảm giá của 23 thu ngân dao động cực hẹp từ **3.006% đến 3.026%** (phẳng tuyệt đối ở mốc chuẩn 3.00%), chứng minh đội ngũ thu ngân chấp hành kỷ luật chiết khấu 100%, không có hiện tượng lạm dụng voucher khuyến mãi gây rò rỉ lợi nhuận.
