# Restaurant Guest Order Management
> Phân hệ quản lý đơn hàng của thực khách trong hệ thống quản lý nhà hàng (SOA - 504070)

## 1. Giới thiệu
Trong hoạt động nhà hàng, việc ghi nhận order thủ công dễ gây sai sót, phục vụ chậm và khó theo dõi tiến độ món. Dự án này mô phỏng **phân hệ quản lý đơn hàng của thực khách** giúp:
- Tạo bàn & nhập thông tin khách
- Gọi món / cập nhật món (tăng/giảm số lượng, ghi chú, xóa món)
- Bếp nhận order và cập nhật trạng thái món
- Quản lý theo dõi, cập nhật menu, xem lịch sử & hỗ trợ thanh toán

Phân hệ đóng vai trò trung gian giữa thực khách và các bộ phận: **phục vụ – bếp – quản lý**, dữ liệu đồng bộ theo thời gian thực để nâng cao trải nghiệm và hiệu quả vận hành.

## 1.1 Mục tiêu & tính thực tế của dự án
### Mục tiêu
- Số hoá quy trình gọi món tại nhà hàng: từ tạo bàn, ghi nhận thông tin khách, gọi món, chế biến đến thanh toán.
- Giảm sai sót khi ghi order thủ công, đồng thời tăng tốc độ phục vụ nhờ dữ liệu cập nhật tập trung.
- Cho phép các bộ phận (phục vụ – bếp – quản lý) phối hợp theo một luồng rõ ràng với trạng thái món/đơn hàng minh bạch.
- Làm quen với kiến trúc dịch vụ & API (REST), thiết kế CSDL và triển khai một hệ thống web đơn giản.

### Tính thực tế
Dự án mô phỏng sát thực tế vận hành nhà hàng:
- **Phục vụ/Quản lý** có thể tạo bàn, nhập thông tin khách, nhận order và thao tác nhanh (tăng/giảm số lượng, ghi chú, xóa món).
- **Bếp** nhận danh sách món cần chế biến theo trạng thái và cập nhật hoàn thành, giúp đồng bộ tiến độ món giữa các bên.
- **Trạng thái món/đơn** giúp theo dõi minh bạch: món đang làm/đã xong, đơn chưa order/đang xử lý/đã thanh toán.
- Có thể mở rộng thêm các tính năng “thực chiến” như: phân quyền chi tiết, in hoá đơn, thống kê doanh thu, quản lý kho, realtime bằng WebSocket, tích hợp QR order tại bàn.

---

## Video demo
- YouTube: https://www.youtube.com/watch?v=HphUzWui0Dk

---

## 2. Tác nhân & quyền (Actors / Roles)
Hệ thống gồm các nhóm người dùng chính:

- **Khách hàng (Customer)**: tạo bàn, xem menu, gọi món, theo dõi trạng thái, yêu cầu thanh toán.
- **Nhân viên phục vụ (Waiter/Staff)**: tạo bàn, nhập thông tin khách, thêm/xóa món, ghi chú, điều chỉnh số lượng, thanh toán.
- **Nhân viên bếp (Kitchen)**: xem danh sách món cần làm, cập nhật trạng thái món (pending/done), đánh dấu món “hết”.
- **Quản lý (Manager)**: có quyền như phục vụ + quản lý menu (thêm/sửa/xóa/đánh dấu hết món), xem danh sách order & lịch sử hóa đơn.

---

## 3. Chức năng chính
### 3.1 Luồng nghiệp vụ (Workflow)
1) **Tạo bàn** (table) khi có khách → **nhập tên + số điện thoại**  
2) Tạo order ở trạng thái **chưa order** → phục vụ/quản lý bắt đầu **thêm món**  
3) **Bếp** nhận món → chuyển trạng thái món (pending → done) / đánh dấu món hết  
4) Khi các món hoàn tất → **thanh toán** → bàn trở về trạng thái sẵn sàng phục vụ

### 3.2 Các chức năng tiêu biểu
- Tạo bàn, xem chi tiết bàn
- Gọi món: thêm món, tăng/giảm số lượng, ghi chú theo món
- Theo dõi trạng thái chế biến món
- Thanh toán & xác nhận thanh toán
- Quản lý menu (thêm/sửa/xóa, đổi trạng thái còn/hết)
- Hủy hóa đơn (trường hợp đặc biệt)
- Xem lịch sử order/hóa đơn theo thời gian

---

## 4. Công nghệ sử dụng (Tech Stack)
- **Front-end**: HTML, CSS (web cơ bản)
- **Back-end**: Spring Boot (RESTful API)
- **Database**: MySQL
- **Docker**: dùng để triển khai nhanh môi trường (nếu repo có cấu hình)

---

## 5. Cơ sở dữ liệu (Database Overview)
Các thực thể chính:
- **Users**: tài khoản nội bộ (username, password, role)
- **Foods**: món ăn (name, category, price, status: còn/hết)
- **Orders**: đơn hàng (status, customerName, customerPhone, subtotal/tax/total, ...)
- **OrderDetails**: chi tiết món trong order (foodId, quantity, note, status done/pending, time)
- **Tables**: bàn ăn (capacity, status: khả dụng/đang dùng)

> File SQL khởi tạo dữ liệu: `Sql_soa_midterm.txt` (trong repo)

---

## 6. API (REST) – Base path: `/api/v1`
> Lưu ý: Endpoint có thể thay đổi tuỳ theo triển khai trong code. Khuyến nghị kiểm tra bằng Swagger UI sau khi chạy backend.

### 6.1 Foods
- `GET  /api/v1/foods?status={0|1}`  
  - `status=0`: còn món, `status=1`: hết món
- `POST /api/v1/foods`  
  - cập nhật thông tin món (id, name, category, price, status)
- `GET  /api/v1/foods/search?status=...&name=...&category=...`
- `GET  /api/v1/foods/categories`

### 6.2 Orders
- `POST /api/v1/orders/new`  
  - tạo order/bàn mới (thông tin khách + capacity… tuỳ hiện thực)
- `POST /api/v1/orders/{id}/add`  
  - body ví dụ:
    ```json
    { "foodId": 1, "quantity": 2, "orderId": 10 }
    ```
- `POST /api/v1/orders/{id}/payment`  
  - thanh toán order
- `GET  /api/v1/orders`  
  - note status: `1` đã thanh toán, `0` chưa thanh toán, `3` chưa order
- `GET  /api/v1/orders/{id}`
- `DELETE /api/v1/orders/{id}`
- `GET  /api/v1/orders/table/{tableId}` *(tuỳ theo code có thể khác)*

### 6.3 Order Details
- `GET    /api/v1/orderDetails`
- `POST   /api/v1/orderDetails`
- `DELETE /api/v1/orderDetails/{id}`

### 6.4 Auth / Tables
- `POST /api/v1/login`
- `GET  /api/v1/tables`

---

## 7. Hướng dẫn chạy dự án (How to Run)
> Vì repo có cả frontend + backend + SQL, bạn có thể chạy theo 1 trong 2 cách sau.

### 7.1 Chuẩn bị
- JDK (Java) + Maven (hoặc Maven Wrapper nếu project có `mvnw`)
- MySQL (hoặc Docker nếu repo có docker-compose)
- Import dữ liệu SQL: `Sql_soa_midterm.txt`

### 7.2 Setup Database
1. Tạo database (ví dụ: `restaurant_db`)
2. Import file `Sql_soa_midterm.txt` vào MySQL
3. Cấu hình kết nối DB trong backend (thường nằm ở `src/main/resources/application.properties`):
   - `spring.datasource.url`
   - `spring.datasource.username`
   - `spring.datasource.password`

### 7.3 Chạy Backend (Spring Boot)
Vào thư mục backend (thường là thư mục có `pom.xml`), chạy:

```bash
# nếu có maven wrapper
./mvnw spring-boot:run

# hoặc dùng maven
mvn spring-boot:run
