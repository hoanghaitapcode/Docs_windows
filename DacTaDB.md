# 🗄️ Thiết kế Cơ sở dữ liệu (Database Schema)

## 1. Tổng quan hệ thống

* **Database Engine:** PostgreSQL (thông qua Supabase).
* **Bảo mật (Security):**
    * Áp dụng **RLS (Row Level Security)** chặt chẽ.
    * Mỗi bảng có policy riêng biệt (Ví dụ: Chỉ `Staff` mới có quyền quản lý khách hàng, đơn hàng; `Public` có thể xem sản phẩm).
* **Tính năng nâng cao:**
    * **Database Triggers:** Tự động hóa các tác vụ (Sinh mã đơn `DH...`, trừ kho khi bán, sinh mã khách hàng `KH...`).
    * **Realtime:** Hỗ trợ lắng nghe thay đổi dữ liệu theo thời gian thực.

---

## 2. Các bảng dữ liệu (Tables)

Hệ thống bao gồm **7 bảng chính** với các mối quan hệ chặt chẽ:

### 👤 1. `profiles`
Thông tin mở rộng của nhân viên/admin.
* **Liên kết:** 1-1 với bảng `auth.users` của Supabase.
* **Phân quyền (Roles):** `admin` hoặc `sale`.

### 📂 2. `categories`
Quản lý danh mục sản phẩm.
* **Dữ liệu:** Tên danh mục (Điện thoại, Laptop, Phụ kiện...).

### 📱 3. `products`
Quản lý thông tin sản phẩm.
* **Giá:** Quản lý cả giá nhập (`import_price`) và giá bán (`price`).
* **Kho:**
    * `stock_quantity`: Số lượng tồn kho hiện tại.
    * `min_stock_level`: Ngưỡng cảnh báo hết hàng.
* **Media:** Hỗ trợ nhiều ảnh (column `images` kiểu mảng/array).

### 👥 4. `customers`
Quản lý thông tin khách hàng.
* **Mã KH:** Tự động sinh bởi Trigger (Ví dụ: `KH0001`, `KH0002`).
* **Loyalty:**
    * `loyalty_points`: Điểm tích lũy.
    * `membership_tier`: Hạng thành viên (`bronze`, `silver`, `gold`...).

### 🎫 5. `coupons`
Quản lý mã giảm giá/khuyến mãi.
* **Loại giảm giá:** Theo phần trăm (%) hoặc số tiền cố định (fixed amount).
* **Ràng buộc:** Giới hạn số lần sử dụng (`usage_limit`), ngày hết hạn (`valid_until`).

### 📦 6. `orders`
Quản lý đơn hàng bán ra.
* **Mã đơn:** Tự động sinh bởi Trigger (Format: `DH` + `YYMMDD` + `Sequence`, Ví dụ: `DH240101001`).
* **Trạng thái:** Thanh toán (`payment_status`), Trạng thái đơn (`status`).

### 🛒 7. `order_items`
Chi tiết sản phẩm trong từng đơn hàng.
* **Trigger quan trọng:** `update_stock_on_order`
    * Tự động **TRỪ** tồn kho (`products.stock_quantity`) khi tạo đơn.
    * Tự động **CỘNG** lại tồn kho khi hủy đơn hàng.
###  Các tính năng mở rộng (Views & Storage) 
* **Views (Báo cáo):**
    * daily_revenue: Báo cáo doanh thu, lợi nhuận theo ngày (tự động tính lãi = giá bán - giá nhập).
    * top_products: Top 10 sản phẩm bán chạy nhất.
    * low_stock_products: Danh sách sản phẩm sắp hết hàng cần nhập thêm.
* **Storage**: 2 Buckets products và assets (đều public).

---
