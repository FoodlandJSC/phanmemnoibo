# CLAUDE.md — Hệ thống nội bộ Foodland

Phần mềm quản lý nội bộ cho Foodland JSC (thực phẩm, khoảng 55 nhân sự, khoảng 20 người dùng đồng thời) thay cho MISA SME 2022 bản offline. MISA vẫn chạy song song; hai hệ thống độc lập hoàn toàn, không tích hợp, không đối chiếu. Phần mềm mới phục vụ **quản trị nội bộ**, không nhằm ra số liệu kế toán chính thức.

Trả lời và viết tài liệu bằng **tiếng Việt**.

## Đọc trước khi làm việc
Thứ tự: `docs/ROADMAP.md` → `docs/PROJECT_OVERVIEW.md` → `docs/BUSINESS_REQUIREMENTS.md` → `docs/TECHNICAL_ARCHITECTURE.md`.

- `docs/ROADMAP.md` là nguồn mới nhất về lộ trình và quyết định công nghệ (cập nhật 20/11/2026). Khi mâu thuẫn với các file còn lại thì ROADMAP thắng.
- Các tài liệu dùng nhãn độ tin cậy: `[XÁC NHẬN]` (người dùng đã chốt), `[TỪ ẢNH]` (quan sát từ ảnh MISA), `[ĐỀ XUẤT]`, `[GIẢ ĐỊNH]`, `[CHƯA RÕ]`. Chỉ coi `[XÁC NHẬN]` là quyết định đã chốt.
- Tài liệu chưa đầy đủ: `BUSINESS_REQUIREMENTS.md` nhắc Phụ lục A và B (đặc tả hàng hóa, khách hàng) nhưng phụ lục **không có** trong file. Các file `CLAUDE_INSTRUCTIONS`, `CURRENT_STATUS_AND_TODO`, `DECISIONS_AND_CHANGELOG`, `DEVELOPMENT_HISTORY`, `CODING_CONVENTIONS`, thư mục `source/` (prototype HTML cũ) và `screenshots/` cũng chưa có trong repo. Đừng suy diễn nội dung của chúng.
- Danh sách chức năng đang dùng trên MISA nằm ở file PDF "Phân tích yêu cầu phần mềm" do người dùng lập (nếu có trong `docs/`). Đó là bảng module / chức năng con / thao tác, chưa có trường dữ liệu từng form.

## Giai đoạn hiện tại
Dựng **Frontend mã thật** (React + TypeScript) với dữ liệu giả sau một lớp API giả. Khi FE được chốt mới thiết kế dữ liệu (ERD) và viết Backend. Chưa có backend, chưa có database.

## Quyết định công nghệ đã chốt `[XÁC NHẬN]`
- Ant Design + AG Grid (bản Community, không dùng tính năng Enterprise), monorepo pnpm, kiến trúc FE feature-based.
- Giao diện bám sát MISA cũ (bố cục, thanh công cụ, lưới có hàng lọc theo cột, tab, phím tắt F9/F3) để nhân viên quen; hiện đại hơn về hình thức, tông xanh lá dịu; tối ưu chức năng và chỉnh bố cục sau theo yêu cầu.
- Trang danh sách: vào trang chỉ nút **Thêm** sáng; chọn một dòng thì Sửa, Xóa, Nhân bản sáng theo dòng đó.
- Mục "Chưa sử dụng" trong tài liệu chức năng không làm ở giai đoạn đầu; giữ toàn bộ danh sách báo cáo đã liệt kê.
- Thứ tự dựng FE: khung chung → danh mục → Mua → Bán → Kho → Quỹ/Ngân hàng → Báo cáo → Bàn làm việc → Hệ thống.

Các đề xuất chưa xác nhận (Vite, React Hook Form + Zod, TanStack Query, MSW, ECharts, cấu trúc thư mục hai cấp `features/<lĩnh vực>/<chức năng>/`, lớp `shared/ui` bọc Ant Design...) xem `docs/ROADMAP.md` mục 4. Hỏi người dùng trước khi coi là đã chốt.

## Cách làm việc với người dùng
- Người dùng là web developer (NodeJS, React, TypeScript).
- Người dùng gửi ảnh MISA đại diện và thông tin cho từng chức năng **khi làm tới chức năng đó**. Với chức năng chưa có ảnh hoặc thông tin về trường, tab, quy tắc: hỏi, không tự bịa.
- Quy trình: phân tích đầy đủ → nêu rõ điểm cần xác nhận → người dùng trả lời → chốt → sang phần kế tiếp.
- Mỗi phần làm dứt điểm, được người dùng duyệt rồi mới sang phần sau.
- Mã tổ chức theo module/feature ngay từ đầu, chia thư mục rõ ràng theo chức năng.
- Cập nhật tài liệu trong `docs/` khi có quyết định mới.

## Việc còn mở (xem chi tiết ở `docs/ROADMAP.md` mục 5)
- Xác nhận phạm vi mở rộng (báo cáo TSCĐ, Khế ước vay, Ngân sách khi chưa có menu nhập liệu).
- "Công nợ" ở Bàn làm việc có làm không.
- Cách khởi tạo danh mục và số dư đầu kỳ.
- Các câu hỏi nghiệp vụ mở khác: `docs/BUSINESS_REQUIREMENTS.md` mục 6.
