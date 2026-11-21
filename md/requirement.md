## Yêu cầu Nghiệp vụ Chi tiết (Original Requirements)
- Nhân sự sẽ nhập các trường thông tin:
  + **Thông tin người đề xuất:** Họ và tên, chức vụ, bộ phận/phòng ban, ngày đề xuất = ngày tạo (mặc định tự động điền thông tin của mình từ phiên đăng nhập / API nhân sự).
  + **Nội dung đề xuất mua sắm:** Có thể có nhiều sản phẩm, mỗi sản phẩm có: Tên, Số lượng, Hình ảnh hoặc link đính kèm.
  + **Mục đích sử dụng:** Đoạn văn bản.
  + **Tổng giá trị dự kiến:** Nhập số tiền.
  + **Nguồn kinh phí:** Đoạn text.
  + **Danh sách nhà cung cấp:** Có thể có nhiều nhà cung cấp (Tên, mô tả).
  + **Thời gian dự kiến giao hàng:** Khoảng thời gian từ ngày A -> Đến ngày B.
  + **Chọn người duyệt đề xuất:** Chọn từ danh sách nhân viên (lấy từ API bên ngoài).
- **Luồng xử lý sau khi nhập xong thông tin:**
  + Tạo ra **1 bảng Đề xuất mua sắm riêng** có trạng thái (`Chờ duyệt`, `Đã duyệt`, `Từ chối`). Bảng này dùng để lưu trữ thông tin dự trù.
  + Tự động tạo ra **1 Công việc (Task)** trên module Tasks sẵn có:
    - **Người nhận việc (`assignees`):** Người được chọn duyệt đề xuất.
    - **Người theo dõi (`watchers`):** Các thành viên trong Ban Giám Đốc (chính là nhóm **BGĐ** trong module Tasks công việc).
  + **Thông báo trên App/Web:** Bắn thông báo cho người liên quan; khi ấn vào thông báo sẽ nhảy đúng vào công việc (Task) đó.
