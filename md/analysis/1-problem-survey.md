# BƯỚC 1: KHẢO SÁT & HIỂU BÀI TOÁN — MODULE ĐỀ XUẤT MUA SẮM THIẾT BỊ (PURCHASE REQUISITION)

> **Tài liệu nguồn (Bước 0):** [`md/requirement.md`](../requirement.md)  
> **Quy chuẩn áp dụng:** [`.agents/rules/2-analysis/1_problem_survey_5_questions.md`](../../.agents/rules/2-analysis/1_problem_survey_5_questions.md)  
> **Đầu ra làm cơ sở cho:** Bước 2 — Truy vết & Bóc tách Tác nhân (`md/analysis/2-actor-identification.md`)

---

## 1. Câu 1: Tại sao cần xây dựng hệ thống này? *(Mục đích kinh doanh & Ranh giới hệ thống)*

### 1.1. Lý do ra đời & Lợi ích cốt lõi (Business Value)
1. **Số hóa và lưu trữ tập trung dự trù mua sắm:** Xây dựng module nhỏ **Quản lý Đề xuất mua sắm thiết bị** nằm trong hệ thống quản lý nội bộ **24h Group** (`https://24hgroup.com.vn/`) để chuẩn hóa biểu mẫu đề xuất và lưu trữ tập trung vào **1 bảng Đề xuất mua sắm riêng** phục vụ mục đích dự trù theo 3 trạng thái: `Chờ duyệt`, `Đã duyệt`, `Từ chối`.
2. **Đồng bộ thông tin nhân sự từ hệ sinh thái 24h:** Tận dụng hệ thống sẵn có bằng cách gọi **API bên ngoài** để tự động điền thông tin người đề xuất (Họ và tên, chức vụ, bộ phận/phòng ban) và tra cứu danh sách nhân viên khi chọn người duyệt đề xuất.
3. **Liên thông tự động với Module Tasks & Thông báo:** Ngay sau khi lập đề xuất dự trù, hệ thống tự động gọi API sang **Module Tasks sẵn có (`/api/v1/tasks`)** để tạo 1 Công việc mới (gán Người duyệt đề xuất làm người nhận việc, gửi `_id` cố định của nhóm **Ban Giám Đốc - BGĐ** làm người theo dõi) và kích hoạt thông báo trên App/Web để ấn vào nhảy thẳng tới đúng công việc đó.

### 1.2. Ranh giới hệ thống (System Boundary)
- **Trong phạm vi (In-Scope):**
  - Quản lý biểu mẫu và bảng **Đề xuất mua sắm** riêng để lưu thông tin dự trù (người đề xuất, danh sách sản phẩm kèm số lượng/hình ảnh/link, mục đích sử dụng, tổng giá trị dự kiến, nguồn kinh phí, danh sách nhà cung cấp, khoảng thời gian dự kiến giao hàng `từ ngày A -> đến ngày B`, người duyệt đề xuất).
  - Quản lý vòng đời 3 trạng thái của phiếu đề xuất mua sắm: `Chờ duyệt`, `Đã duyệt`, `Từ chối`.
  - Kết nối API bên ngoài để lấy thông tin nhân sự và gọi API sang **Module Tasks** để khởi tạo công việc phê duyệt kèm thông báo điều hướng.
- **Ngoài phạm vi (Out-of-Scope):**
  - Không xây dựng lại hệ thống Đăng nhập, Quản lý Nhân sự (`Employee`) hay Phòng ban/Chi nhánh (`Branch`).
  - Không xây dựng lại hệ thống Quản lý Công việc (`Tasks`), Nhóm công việc (`Groups`) hay hệ thống Push Notification (toàn bộ các vấn đề phía sau khi tạo Task như nhóm BGĐ theo dõi, thảo luận, tiến độ công việc và bắn thông báo điều hướng đều do **Module Tasks sẵn có** xử lý).
  - Không bao gồm các nghiệp vụ sau dự trù: Đơn đặt hàng nhà cung cấp (PO), Nhập kho/Quản lý tồn kho, Hạch toán kế toán/Thanh toán.

---

## 2. Câu 2: Hiện tại quy trình đang làm như thế nào? *(Quy trình thủ công hiện hữu — As-Is Process)*

Trước khi có module Đề xuất mua sắm, quy trình đang được thực hiện thủ công qua 3 bước rời rạc:
1. **Bước 1 (Lập file rời rạc):** Nhân sự khi có nhu cầu mua sắm thiết bị tự lập phiếu đề xuất trên file **Excel / Google Sheets** (tự gõ thông tin cá nhân, phòng ban, danh sách sản phẩm, số lượng, link/hình ảnh, nhà cung cấp, giá trị dự kiến, nguồn kinh phí, thời gian giao hàng dự kiến).
2. **Bước 2 (Xin duyệt qua chat):** Nhân sự gửi file Excel/Google Sheets và nhắn tin qua **nhóm chat nội bộ** để trình xin ý kiến phê duyệt từ Quản lý / Người duyệt và Ban Giám Đốc.
3. **Bước 3 (Tạo việc thủ công):** Sau khi được đồng ý qua tin nhắn chat, nhân sự mới tự mở **Module Tasks** để tạo công việc thủ công, nhập lại nội dung và tự chọn người nhận việc, người theo dõi.

---

## 3. Câu 3: Vấn đề (Nỗi đau) hiện tại là gì? *(Chuyển hóa Pain Points ➔ Mục tiêu hệ thống)*

| Vấn đề hiện tại (Pain Points) | Hệ quả | Mục tiêu hệ thống tương ứng (Business Goals) |
|---|---|---|
| **1. Lập đề xuất trên file Excel/Google Sheets rời rạc:** Nhân sự phải tự nhập lại thông tin cá nhân/phòng ban và thiếu cấu trúc biểu mẫu chuẩn cho nhiều sản phẩm (tên, số lượng, hình ảnh/link), nhiều nhà cung cấp, nguồn kinh phí, khoảng ngày giao hàng dự kiến (`từ ngày A -> đến ngày B`). | Dữ liệu dự trù mua sắm bị phân tán ở nhiều file, dễ nhập thiếu trường thông tin quan trọng; không có bảng dữ liệu Đề xuất mua sắm riêng trên hệ thống để lưu trữ và tra cứu dự trù tập trung. | **Mục tiêu 1 — Chuẩn hóa biểu mẫu & Lưu bảng Đề xuất dự trù riêng:** Tự động điền thông tin người đề xuất (qua API ngoài) + `ngày đề xuất = ngày tạo`; chuẩn hóa nhập danh sách sản phẩm, nhà cung cấp, mục đích sử dụng, tổng giá trị dự kiến, nguồn kinh phí, khoảng ngày giao hàng và lưu vào bảng Đề xuất mua sắm riêng. |
| **2. Xin phê duyệt qua tin nhắn nhóm chat thủ công:** Việc gửi đề xuất qua nhóm chat dễ bị trôi tin nhắn, thiếu cơ chế ghi nhận quyết định phê duyệt tập trung. | Chậm trễ trong khâu duyệt đề xuất; khó tra cứu và thống kê phiếu đề xuất nào đang `Chờ duyệt`, `Đã duyệt` hay `Từ chối`. | **Mục tiêu 2 — Quản lý luồng chọn Người duyệt & Trạng thái phê duyệt:** Cho phép chọn Người duyệt đề xuất từ danh sách nhân viên (qua API ngoài) và quản lý rõ ràng 3 trạng thái trên bảng Đề xuất mua sắm (`Chờ duyệt`, `Đã duyệt`, `Từ chối`). |
| **3. Phải tự vào Module Tasks tạo công việc thủ công sau khi chat:** Nhân sự tốn công nhập liệu 2 lần (từ Excel/Chat sang Tasks), dễ quên gắn nhóm Ban Giám Đốc (BGĐ) vào theo dõi và người liên quan không nhận được thông báo gắn trực tiếp với công việc ngay khi lập đề xuất. | Đứt gãy giữa bước lập dự trù mua sắm và bước giao việc/giám sát trên hệ thống 24h; tốn thời gian thao tác tay và dễ bỏ sót người theo dõi trong BGĐ. | **Mục tiêu 3 — Tự động gọi API tạo Công việc trên Module Tasks & Bắn thông báo điều hướng:** Ngay sau khi lưu phiếu đề xuất, tự động gọi API sang Module Tasks tạo 1 Công việc (Người duyệt là người nhận việc, gửi `_id` cố định của nhóm BGĐ làm người theo dõi) và bắn thông báo trên App/Web để ấn vào nhảy thẳng tới công việc đó. |

---

## 4. Câu 4: Ai là người tham gia sử dụng? *(Các nhóm đối tượng trong phạm vi module)*

| Nhóm đối tượng (thô) | Căn cứ trong đề (`md/requirement.md`) | Nhu cầu chính trên Module Đề xuất mua sắm |
|---|---|---|
| **1. Nhân sự (Người đề xuất)** | *"Nhân sự sẽ nhập các trường thông tin: Thông tin người đề xuất... mặc định tự động điền thông tin của mình... Nội dung đề xuất mua sắm... Chọn người duyệt đề xuất: từ danh sách nhân viên"* | Lập phiếu đề xuất mua sắm dự trù (tự động điền thông tin cá nhân, nhập danh sách sản phẩm, nhà cung cấp, mục đích sử dụng, tổng giá trị dự kiến, nguồn kinh phí, thời gian dự kiến giao hàng `từ ngày A -> đến ngày B`, chọn người duyệt), chỉnh sửa phiếu khi đang ở trạng thái `Chờ duyệt` và theo dõi trạng thái phiếu (`Chờ duyệt`, `Đã duyệt`, `Từ chối`). |
| **2. Người phê duyệt (Người được chọn duyệt đề xuất)** | *"Chọn người duyệt đề xuất: từ danh sách nhân viên... Tạo ra 1 bảng đề xuất mua sắm riêng -> có trạng thái (Chờ duyệt, Đã duyệt, Từ chối)... người nhận việc là người đc đề xuất"* | Xem danh sách và chi tiết phiếu đề xuất mua sắm dự trù mà mình được chọn duyệt; thực hiện quyết định chuyển trạng thái phê duyệt sang `Đã duyệt` hoặc `Từ chối`. |

> **Ghi chú phân định phạm vi đối tượng:**
> - Toàn bộ các vấn đề phía sau khi tạo Công việc — bao gồm việc các thành viên thuộc **nhóm Ban Giám Đốc (BGĐ)** theo dõi công việc, bắn thông báo đẩy trên App/Web để ấn vào nhảy đúng công việc đó, thảo luận và cập nhật tiến độ công việc — đều do **Module Tasks sẵn có** xử lý.
> - **Nhà cung cấp** chỉ là thông tin văn bản (`Tên, mô tả`) do Người đề xuất nhập vào phiếu để phục vụ dự trù, không trực tiếp đăng nhập hay thao tác trên hệ thống.

---

## 5. Câu 5: Kết quả mong muốn là gì? *(Tiêu chí nghiệm thu — Acceptance Criteria)*

| Mục tiêu hệ thống | Tiêu chí đạt cụ thể (Kiểm chứng được) |
|---|---|
| **Mục tiêu 1: Chuẩn hóa biểu mẫu & Lưu bảng Đề xuất mua sắm dự trù riêng** | • Khi mở form tạo đề xuất, hệ thống tự động lấy và điền thông tin Người đề xuất (Họ và tên, chức vụ, bộ phận/phòng ban) từ API bên ngoài và gán `ngày đề xuất = ngày tạo`.<br>• Cho phép thêm nhiều sản phẩm (mỗi sản phẩm có: Tên, Số lượng, Hình ảnh tải lên qua `FilesService` hoặc URL link đính kèm) và nhiều nhà cung cấp (Tên, mô tả).<br>• Nhập đầy đủ Mục đích sử dụng (đoạn văn bản), Tổng giá trị dự kiến (số tiền), Nguồn kinh phí (đoạn text), Thời gian dự kiến giao hàng (`từ ngày A -> đến ngày B`), và chọn Người duyệt đề xuất từ danh sách nhân viên (qua API ngoài).<br>• Lưu thành công bản ghi vào bảng **Đề xuất mua sắm** riêng với trạng thái mặc định ban đầu là `Chờ duyệt`. |
| **Mục tiêu 2: Quản lý phân quyền xem, chỉnh sửa & cập nhật trạng thái phê duyệt** | • Mỗi nhân sự xem được danh sách phiếu do mình đề xuất và phiếu mình được chọn làm Người phê duyệt (thành viên nhóm BGĐ xem được toàn bộ phiếu dự trù).<br>• Khi phiếu đang ở trạng thái `Chờ duyệt`, Người đề xuất được phép chỉnh sửa thông tin phiếu.<br>• Người phê duyệt được chọn trên phiếu có thể chuyển trạng thái phiếu từ `Chờ duyệt` sang `Đã duyệt` hoặc `Từ chối`; ngay sau khi chuyển sang `Đã duyệt` hoặc `Từ chối`, phiếu bị khóa hoàn toàn (không cho sửa/xóa). |
| **Mục tiêu 3: Tự động tích hợp sang Module Tasks & Thông báo điều hướng** | • Ngay sau khi tạo phiếu Đề xuất mua sắm thành công, hệ thống tự động gọi API sang **Module Tasks sẵn có** để tạo 1 Công việc mới: gán Người duyệt đề xuất làm người nhận việc, gửi `_id` cố định (hardcode) của nhóm **BGĐ** làm người theo dõi, và lưu lại `task_id` liên kết vào bản ghi Đề xuất mua sắm.<br>• Module Tasks tự động bắn thông báo trên App/Web cho người nhận việc và người theo dõi; khi ấn vào thông báo sẽ điều hướng nhảy đúng vào chi tiết công việc (`/tasks?task_id=<task_id>`) bên Module Tasks. |

---

## 6. Bảng Các Quyết định Nghiệp vụ & Kiến trúc Đã Chốt (`[Đã xác nhận]`)

| Mã QĐ | Hạng mục quyết định | Nội dung quyết định nghiệp vụ & kiến trúc đã chốt | Trạng thái |
|:---:|---|---|:---:|
| **QĐ-01** | **Ranh giới Module & Tích hợp hệ sinh thái 24h** | Module Đề xuất mua sắm chỉ quản lý bảng Đề xuất dự trù riêng và vòng đời 3 trạng thái (`Chờ duyệt`, `Đã duyệt`, `Từ chối`). Thông tin người đề xuất & danh sách nhân viên lấy qua API ngoài. Sau khi tạo phiếu và gọi API tạo Task, mọi luồng theo dõi của nhóm BGĐ, thông báo nhảy vào việc và xử lý công việc phía sau đều do **Module Tasks sẵn có** đảm nhiệm. | `[Đã xác nhận]` |
| **QĐ-02** | **Quy tắc chỉnh sửa & khóa phiếu theo trạng thái** | Chỉ cho phép Người đề xuất chỉnh sửa phiếu khi trạng thái đang là `Chờ duyệt`. Ngay khi Người phê duyệt chuyển trạng thái sang `Đã duyệt` hoặc `Từ chối`, phiếu đề xuất bị khóa hoàn toàn (không cho phép sửa hoặc xóa) để bảo toàn lịch sử dự trù. | `[Đã xác nhận]` |
| **QĐ-03** | **Cơ chế gắn nhóm Ban Giám Đốc (BGĐ) khi tạo Task** | Bên Module Tasks có sẵn 1 nhóm riêng là nhóm **BGĐ** (danh sách thành viên trong nhóm được cấu hình động qua Module Tasks). Khi Module Đề xuất mua sắm gọi API tạo công việc sang Module Tasks, chỉ cần gửi `_id` cố định (hardcode hằng số) của nhóm BGĐ. | `[Đã xác nhận]` |
| **QĐ-04** | **Phạm vi xem danh sách đề xuất & Đính kèm trên sản phẩm** | Mỗi nhân sự xem được danh sách phiếu do mình đề xuất và phiếu mình được chọn làm Người phê duyệt (thành viên nhóm BGĐ xem được toàn bộ phiếu dự trù). Trường hình ảnh/link của mỗi sản phẩm hỗ trợ cả tải file ảnh lên (`FilesService`, tối đa 5MB `jpeg/png/webp`) lẫn nhập URL link đính kèm. | `[Đã xác nhận]` |
