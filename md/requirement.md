# YÊU CẦU NGHIỆP VỤ & BỐI CẢNH HỆ THỐNG: MODULE ĐỀ XUẤT MUA SẮM THIẾT BỊ (PURCHASE REQUISITION)

## 1. Bối cảnh Hệ thống & Ranh giới Phạm vi (System Context & Scope Boundary)
- **Vị trí trong hệ sinh thái:** Module **Đề xuất mua sắm thiết bị** là một module nhỏ (phân hệ độc lập) nằm trong hệ thống Quản lý nội bộ **24h Group** (`https://24hgroup.com.vn/`).
- **Tận dụng các dịch vụ & API sẵn có (External APIs):**
  1. **Thông tin Nhân sự & Ban Lãnh Đạo (gọi API bên ngoài):**
     - Module **không** tự xây dựng hay quản lý bảng Nhân viên (`Employee`), Phòng ban/Chi nhánh (`Branch`) hay Nhóm người dùng.
     - Toàn bộ thông tin người đề xuất (đang đăng nhập), danh sách nhân viên (để chọn người duyệt) và danh sách thành viên thuộc **nhóm Ban Giám Đốc (BGĐ)** đều được lấy thông qua **API bên ngoài** của hệ thống 24h / Tasks Service (ví dụ: `GET /api/v1/tasks/employees/search`, `GET /api/v1/tasks/groups`, `GET /api/v1/tasks/groups/:groupId/members`).
  2. **Module Giao việc & Quản lý tiến độ (`Tasks Service` — `/api/v1/tasks`) đã có sẵn:**
     - Hệ thống 24h đã có sẵn Microservice **Tasks** quản lý công việc (`Task`), người nhận việc (`TaskAssignee`), người theo dõi (`TaskWatcher`), nhóm công việc (`Group` / `GroupMember` — trong đó có nhóm **BGĐ**), bình luận (`Comment`) và nhật ký hoạt động (`TaskActivityLog`).
     - Khi tạo công việc qua `POST /api/v1/tasks/tasks`, hệ thống Tasks đã tích hợp sẵn với `notification_client` (`sendGlobalNotification` / `notifyTaskCreated`) để bắn thông báo đẩy (App Mobile FCM + Web Push + Notification DB) kèm đường dẫn điều hướng `/tasks?task_id=<task_id>` nhảy thẳng vào chi tiết công việc đó.
- **Giới hạn phạm vi của Module Đề xuất mua sắm (In-Scope vs Out-of-Scope):**
  - **Trong phạm vi (In-Scope):**
    - Quản lý bảng **Đề xuất mua sắm** riêng để lưu thông tin dự trù mua sắm thiết bị và theo dõi 3 trạng thái phê duyệt: `Chờ duyệt`, `Đã duyệt`, `Từ chối`.
    - Tích hợp gọi API ngoài để tự động điền thông tin người đề xuất, tra cứu nhân viên chọn người duyệt và lấy thành viên nhóm BGĐ.
    - Tự động gọi API sang **Module Tasks** để tạo công việc phê duyệt (gán người duyệt làm `assignees`, thành viên nhóm BGĐ làm `watchers`) và kích hoạt thông báo điều hướng tới công việc.
  - **Ngoài phạm vi (Out-of-Scope — Không xây dựng dư thừa):**
    - Không xây dựng lại hệ thống Đăng nhập/Quản lý nhân sự/Phòng ban.
    - Không xây dựng lại hệ thống Quản lý công việc (Task CRUD, Checklist, Comment) hay hệ thống Push Notification.
    - Không bao gồm các nghiệp vụ sau dự trù như: Đơn đặt hàng nhà cung cấp (PO), Nhập kho/Quản lý tồn kho, hay Hạch toán kế toán/Thanh toán.

---

## 2. Yêu cầu Nghiệp vụ Chi tiết (Original Requirements)
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
