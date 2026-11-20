# ROADMAP.md — Lộ trình phát triển hệ thống nội bộ Foodland

> Cập nhật 20/11/2026 (bản đầu chốt 18/11/2026). Nhãn độ tin cậy xem `PROJECT_OVERVIEW.md` mục 0.
> **Thay đổi 20/11:** do tiến độ gấp, người dùng bỏ bước Figma và chuyển sang **dựng Frontend (mã thật) trước, chốt FE rồi mới thiết kế dữ liệu và code Backend**. Cách làm "thiết kế ERD toàn bộ trước khi dựng giao diện" của bản 18/11 không còn áp dụng. `[XÁC NHẬN]`

## 1. Nguyên tắc `[XÁC NHẬN]`
- Phần mềm dùng để **phục vụ quản trị nội bộ**, không nhằm ra số liệu kế toán chính thức (thuế, báo cáo tài chính). Hệ thống vẫn độc lập hoàn toàn với MISA.
- Dựng FE bằng **mã dùng thật** (không làm prototype bỏ đi). Dữ liệu giai đoạn này là dữ liệu giả sau một lớp API giả, để sau thay bằng backend thật.
- Tài liệu chức năng (`Phân tích yêu cầu phần mềm`, PDF do người dùng lập 20/11/2026) là danh sách chức năng đang dùng trên MISA: module, chức năng con, thao tác. Chưa có trường dữ liệu từng form.
- Chi tiết từng chức năng (thuộc tính, tab phụ, component): người dùng cung cấp ảnh và thông tin **khi làm tới chức năng đó**. Khung giao diện các mục khá giống nhau nên chỉ gửi ảnh đại diện.
- Mục ghi "Chưa sử dụng" trong tài liệu (MISA có tính năng nhưng chưa có bản ghi) **không phát triển ở giai đoạn đầu**.
- Mục Báo cáo: giữ **tất cả** báo cáo đã liệt kê trong tài liệu.
- Phạm vi được **mở rộng** so với bản 18/11: những mục có trong tài liệu chức năng (ví dụ Công cụ dụng cụ, các nhóm danh mục Lương/TSCĐ/Mẫu hóa đơn, Xuất hóa đơn, phiếu Trả lương/Nộp thuế/Nộp bảo hiểm) được tính là trong phạm vi, trừ các mục "Chưa sử dụng". `[GIẢ ĐỊNH]` Claude hiểu như vậy; chờ người dùng xác nhận, đặc biệt phần báo cáo TSCĐ, Khế ước vay, Ngân sách vì tài liệu không có menu nhập liệu tương ứng.

## 2. Các bước

| # | Bước | Đầu ra |
|---|---|---|
| 1 | Nghiên cứu và chốt phạm vi | Tài liệu chức năng, các quyết định ở mục 1 |
| 2 | Chốt công nghệ Frontend | Xem mục 4 bên dưới |
| 3 | Dựng khung chung và component dùng chung | Layout kiểu MISA hiện đại, tông xanh lá dịu; lưới danh sách, lưới chi tiết, form chứng từ, thanh công cụ, bộ lọc kỳ, dropdown tìm kiếm nhiều cột, hộp thoại, cây nhóm; điều khiển bằng cấu hình |
| 4 | Dựng FE theo thứ tự | khung chung → danh mục → Mua → Bán → Kho → Quỹ/Ngân hàng → Báo cáo (dùng mẫu báo cáo chung) → Bàn làm việc → Hệ thống (người dùng, phân quyền, nhật ký) `[XÁC NHẬN]` |
| 5 | Duyệt và chốt FE | Người dùng duyệt từng phần, chốt toàn bộ FE |
| 6 | Thiết kế dữ liệu và nghiệp vụ | Luồng nghiệp vụ, ERD toàn hệ thống (gồm bút toán, sổ biến động kho), quy tắc hạch toán, phân quyền; chạy thử kịch bản trọn vẹn |
| 7 | Quyết định kỹ thuật Backend | Framework, ORM, auth/2FA, hạ tầng, backup |
| 8 | Lập trình Backend và nối với FE | Theo thứ tự nền → Mua/Bán → Kho → Kế toán; kiểm thử số liệu, cho nhân viên dùng thử |
| 9 | Khởi tạo dữ liệu, triển khai, backup, đào tạo, chuyển sang dùng chính thức | Rồi bảo trì, nâng cấp từ xa |

## 3. Quy tắc trong quá trình làm
- Sau khi FE đã chốt, nếu cần thay đổi thì cập nhật tài liệu trước rồi mới sửa code.
- Rủi ro đã biết khi chốt FE trước ERD: khi thiết kế dữ liệu có thể phát hiện trường hoặc luồng còn thiếu và phải sửa lại FE. Giảm bằng cách ghi rõ nghiệp vụ và quy tắc tính toán của từng form khi dựng FE.

## 4. Công nghệ FE
Ràng buộc đã biết: NodeJS, ReactJS, TypeScript; mã theo module ngay từ đầu.

Đã chọn (người dùng xác nhận 20/11/2026) `[XÁC NHẬN]`:
- Thư viện giao diện **Ant Design** + lưới dữ liệu **AG Grid** (bản Community, không dùng tính năng Enterprise).
- **Monorepo pnpm**.
- Kiến trúc FE **feature-based**.
- Giao diện bám sát MISA cũ để nhân viên quen; tối ưu chức năng và chỉnh bố cục sau theo yêu cầu.
- Repo: `https://github.com/FoodlandJSC/phanmemnoibo`.

Claude đề xuất, chờ người dùng xác nhận `[ĐỀ XUẤT]`:
- React + TypeScript strict, Vite, SPA (không cần Next.js).
- React Hook Form + Zod; TanStack Query; React Router; MSW cho API giả; Apache ECharts cho biểu đồ; react-hotkeys-hook (F9, F3); Vitest + Testing Library; ESLint + Prettier.
- Feature-based hai cấp: `features/<lĩnh vực>/<chức năng>/` (pages, components, api, hooks, schemas, config, index.ts). Luật: `shared` không import `features`; các chức năng chỉ import nhau qua `index.ts`; ESLint chặn vi phạm.
- Lớp `shared/ui` bọc Ant Design; ESLint chặn import `antd` trực tiếp ngoài lớp này để dễ đổi thư viện hoặc chỉnh bố cục sau này.
- Dựng khung chung và một trang danh sách, một form chứng từ trước, rồi người dùng so với ảnh MISA và chỉnh theme (mật độ, màu, vị trí nút) trước khi làm tiếp.
- Cấu trúc thư mục dự kiến:

```
apps/web/src/
  app/        (provider, router, layout khung, theme)
  features/
    catalog/    (items, item-groups, warehouses, units, customers, suppliers, ...)
    purchasing/ (purchase-orders, purchase-vouchers, purchase-returns, payables)
    sales/  inventory/  cash/  bank/  reports/  dashboard/  system/
  shared/     (ui, grid, form, api-client, hooks, lib)
  mocks/      (MSW handlers, dữ liệu giả)
packages/shared/   (kiểu dữ liệu, schema Zod dùng chung với backend)
```

## 5. Còn chưa chốt
- Các điểm ở mục 4 đánh dấu `[ĐỀ XUẤT]`.
- Phạm vi mở rộng: xác nhận cách hiểu ở mục 1, và các báo cáo TSCĐ/Khế ước vay/Ngân sách khi chưa có nhập liệu.
- "Công nợ" ở Bàn làm việc (MISA không xem được): có làm không.
- Cách khởi tạo danh mục và số dư đầu kỳ.
- Các câu hỏi nghiệp vụ mở khác: xem `BUSINESS_REQUIREMENTS.md` mục 6.
- Việc dùng Figma, prototype HTML cũ (4 trang danh mục), canvas và script patch Khách hàng: không còn nằm trên đường chính; chỉ còn là tài liệu tham khảo.
