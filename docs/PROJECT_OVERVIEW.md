# PROJECT_OVERVIEW.md — Hệ thống nội bộ Foodland

> Bộ tài liệu bàn giao gồm 8 file Markdown + thư mục `source/` + thư mục `screenshots/`. Tài liệu này là điểm bắt đầu. Đọc theo thứ tự: PROJECT_OVERVIEW → CLAUDE_INSTRUCTIONS → CURRENT_STATUS_AND_TODO → BUSINESS_REQUIREMENTS → TECHNICAL_ARCHITECTURE → DECISIONS_AND_CHANGELOG → DEVELOPMENT_HISTORY → CODING_CONVENTIONS.
>
> Thời điểm lập bản bàn giao: 18/11/2026. Mọi mốc ngày trong bộ tài liệu suy ra từ dấu thời gian file và bộ nhớ dự án; hội thoại không có ngày giờ cho từng tin nhắn.

## 0. Chú giải nhãn độ tin cậy (dùng xuyên suốt 8 file)

| Nhãn | Nghĩa |
|---|---|
| `[XÁC NHẬN]` | Người dùng nói rõ hoặc đồng ý rõ ràng trong hội thoại. |
| `[TỪ ẢNH]` | Quan sát trực tiếp từ ảnh chụp phần mềm MISA do người dùng gửi. Là sự thật về MISA, chưa phải yêu cầu đã chốt cho hệ thống mới. |
| `[ĐỀ XUẤT]` | Claude đề xuất; người dùng chưa phản đối nhưng chưa xác nhận rõ. |
| `[GIẢ ĐỊNH]` | Claude tạm giả định để tiếp tục; chưa được hỏi hoặc chưa được trả lời. |
| `[CHƯA RÕ]` | Chưa có thông tin. |

Khi một thông tin đổi qua thời gian, tài liệu ghi trạng thái mới nhất và ghi lịch sử thay đổi trong `DECISIONS_AND_CHANGELOG.md`.

## 1. Dự án là gì

- Tên dự án (trong Claude Project): "hệ thống nội bộ foodland". `[XÁC NHẬN]`
- Mục tiêu: phát triển hệ thống phần mềm quản lý nội bộ cho công ty **Foodland JSC** để thay cho việc dùng phần mềm bên ngoài (MISA). `[XÁC NHẬN]`
- Vai trò của Claude: chuyên gia công nghệ kiêm trợ lý phát triển cho người dùng. `[XÁC NHẬN]`
- Người dùng: web developer, stack chính NodeJS, ReactJS, TypeScript. `[XÁC NHẬN]`

## 2. Bối cảnh doanh nghiệp

| Mục | Nội dung | Nhãn |
|---|---|---|
| Lĩnh vực | Thực phẩm | `[XÁC NHẬN]` |
| Quy mô nhân sự | Khoảng 55 người | `[XÁC NHẬN]` |
| Người dùng đồng thời | Khoảng 20 ở thời điểm hiện tại | `[XÁC NHẬN]` |
| Số đơn mỗi ngày | Người dùng nói 15–30 đơn/ngày. Dữ liệu trong ảnh MISA cho thấy khối lượng chứng từ lớn hơn (xem mục 4). Hai con số chưa được đối chiếu; "đơn" có thể được định nghĩa khác nhau. | `[XÁC NHẬN]` + `[CHƯA RÕ]` |
| Phần mềm hiện dùng | MISA, bản offline mua từ lâu, không nâng cấp, không có source; dữ liệu lớn nên phải xóa bớt dữ liệu liên tục; chức năng gần như vẫn đáp ứng nhu cầu | `[XÁC NHẬN]` |
| Phiên bản MISA trong ảnh | MISA SME 2022 R11 Standard | `[TỪ ẢNH]` |
| Công ty đang làm việc trong MISA | Thanh trạng thái ghi "CỬA HÀNG THỰC PHẨM TÂM AN"; quan hệ với "Foodland JSC" chưa được người dùng giải thích | `[TỪ ẢNH]` + `[CHƯA RÕ]` |
| Máy chủ / dữ liệu MISA | `DESKTOP-EFCI4LM\MISASME2022`, tên dữ liệu kế toán `KETOAN2026 2` | `[TỪ ẢNH]` |

## 3. Quan hệ giữa hệ thống mới và MISA (đã thay đổi trong quá trình làm việc)

Trạng thái mới nhất `[XÁC NHẬN]`: MISA vẫn tiếp tục được dùng song song, **hai hệ thống độc lập hoàn toàn, không liên quan, không tích hợp, không đối chiếu số liệu**. Hệ thống mới chỉ để quản lý nội bộ thay vì phải dùng phần mềm bên ngoài.

Người dùng cũng nêu rằng có thể bê nguyên chức năng MISA sang làm lại; chỉ tham khảo giao diện MISA vì bản offline không có source. Người dùng không yêu cầu bê dữ liệu cũ của MISA sang. `[XÁC NHẬN]`

Lịch sử: Claude từng giả định hai hệ thống có liên hệ (đối chiếu, xuất Excel cho MISA, nguồn chuẩn) và đã bị người dùng đính chính. Chi tiết trong `DECISIONS_AND_CHANGELOG.md`.

## 4. Quy mô dữ liệu quan sát được từ ảnh MISA `[TỪ ẢNH]`

| Đối tượng | Số liệu | Ghi chú |
|---|---|---|
| Vật tư hàng hóa | 11.224 dòng | |
| Nhóm VTHH | 217 dòng | dạng cây |
| Kho | 44 dòng (45 sau khi thử thêm) | |
| Đơn vị tính | 72 dòng | |
| Khách hàng | 14.073 dòng | |
| Chứng từ bán hàng | 7.844 dòng | kỳ 01/09/2026 đến khoảng 16/11/2026 |
| Phiếu xuất kho | 2.212 dòng | kỳ 01/11/2026 đến 16/11/2026 |

## 5. Phạm vi chức năng

### Trang cần xây dựng `[XÁC NHẬN]`
Dựa trên ảnh MISA do người dùng gửi: **Bàn làm việc, Quỹ, Ngân hàng, Mua hàng, Bán hàng, Kho**. Các menu MISA không được chụp thì không cần làm: Khế ước vay, Hóa đơn điện tử, Quản lý hóa đơn, Công cụ dụng cụ, Tài sản cố định, Tiền lương.

> **Cập nhật 20/11/2026:** phạm vi đã được người dùng cho biết sẽ **mở rộng** theo tài liệu "Phân tích yêu cầu phần mềm" (xem `ROADMAP.md`). Danh sách "không cần làm" ở trên không còn là danh sách cuối cùng.

Mỗi trang có nhiều tab con và nhiều form. Người dùng sẽ gửi chi tiết từng tab cần thiết, từng menu một.

### Module nghiệp vụ người dùng liệt kê `[XÁC NHẬN]`
Mua/bán hàng, quản lý kho, kế toán, báo cáo; mỗi module có rất nhiều module nhỏ.

### Hạch toán kế toán `[XÁC NHẬN]`
Người dùng xác nhận hệ thống mới cần hạch toán TK Nợ/TK Có đầy đủ như MISA.

### Thứ tự ưu tiên xây dựng `[XÁC NHẬN]`
1. Mua hàng / Bán hàng
2. Kho
3. Kế toán (bao gồm hạch toán, Quỹ, Ngân hàng... — việc xếp Quỹ/Ngân hàng/Bàn làm việc vào giai đoạn nào là `[ĐỀ XUẤT]`, xem `CURRENT_STATUS_AND_TODO.md`)

## 6. Truy cập, bảo mật, vận hành `[XÁC NHẬN]`

- Chủ yếu dùng trong công ty; chạy trên **web trước** (để bảo trì, nâng cấp, sửa lỗi từ xa); sau này nếu cần mới đóng gói thành web app.
- Chưa có tích hợp với hệ thống khác; sẽ mở rộng sau.
- Vai trò: admin full quyền; các tài khoản khác được admin phân quyền theo từng nghiệp vụ/chức năng.
- Bảo mật: có bảo vệ cấp 2 (2FA); không để người ngoài xem được dữ liệu; việc gọi API và lưu token xác thực cũng không nên để người ngoài biết.
- Sao lưu: sẽ có phương án backup khi triển khai lên server.
- Thiết kế mã nguồn dạng module (chia thư mục rõ ràng theo chức năng) ngay từ ngày đầu.

## 7. Giai đoạn hiện tại

- Đang ở giai đoạn **phân tích yêu cầu và thiết kế giao diện prototype**. `[XÁC NHẬN]` (giai đoạn do người dùng nêu ở đầu dự án).
- **Chưa có mã nguồn backend, chưa có repository, chưa có database thật, chưa có ERD chính thức.** Những gì đã tồn tại: các bản đặc tả phân tích (file spec), một prototype giao diện bằng HTML (4 trang danh mục), một dự án canvas thiết kế, một bảng tính lộ trình. Xem `TECHNICAL_ARCHITECTURE.md` mục 5 và thư mục `source/`.
- Quy trình làm việc đã thống nhất: người dùng gửi ảnh MISA từng tab; Claude phân tích đầy đủ và nêu điểm cần xác nhận; sau khi người dùng trả lời thì chốt, rồi mới sang tab kế tiếp. Cuối cùng gom thành một bản đặc tả `.md` duy nhất để dùng cho lập trình, cho AI khác hoặc VS Code. `[XÁC NHẬN]`

> **Cập nhật 20/11/2026:** đã có repository GitHub `FoodlandJSC/phanmemnoibo` (mới tạo, trống trước khi thêm tài liệu). Hướng đi hiện tại: dựng Frontend mã thật trước, chốt FE rồi mới thiết kế dữ liệu và code Backend. Xem `ROADMAP.md`.

## 8. Tiến độ phân tích theo cụm danh mục (tóm tắt)

| Cụm | Trạng thái |
|---|---|
| Khung giao diện chung | Đã chốt hướng (giữ bố cục MISA, giao diện hiện đại tông xanh lá dịu) |
| Danh mục hàng hóa (Vật tư hàng hóa, Nhóm VTHH, Kho, Đơn vị tính) | Đã phân tích, đã có prototype web tương tác, người dùng đã chấp nhận |
| Danh mục Đối tượng — Khách hàng | Đã phân tích xong và có quyết định; **chưa dựng prototype** (lần thêm vào prototype bị lỗi, chưa áp dụng) |
| Đối tượng — Nhà cung cấp, Nhân viên, danh sách Nhóm KH/NCC, danh sách Điều khoản thanh toán | Chưa nhận ảnh |
| Các danh mục khác (Hệ thống tài khoản, Loại tiền, Hình thức thanh toán...) | Chưa nhận ảnh |
| Chi tiết các tab của Mua hàng, Bán hàng, Kho, Quỹ, Ngân hàng, Bàn làm việc | Mới có một ảnh tổng quan mỗi trang; chưa nhận ảnh chi tiết từng tab |

## 9. Tài liệu và tài sản trong bộ bàn giao

| Mục | Nội dung |
|---|---|
| `PROJECT_OVERVIEW.md` | Tài liệu này |
| `TECHNICAL_ARCHITECTURE.md` | Kiến trúc đề xuất, mô hình dữ liệu khái niệm, cấu trúc prototype hiện có |
| `BUSINESS_REQUIREMENTS.md` | Yêu cầu nghiệp vụ, quan sát từ ảnh MISA, phụ lục đặc tả dữ liệu chi tiết |
| `DECISIONS_AND_CHANGELOG.md` | Nhật ký quyết định, thay đổi yêu cầu, phương án bị loại |
| `DEVELOPMENT_HISTORY.md` | Lịch sử các bước đã làm, lỗi gặp phải và cách xử lý, chỉ mục ảnh chụp |
| `CURRENT_STATUS_AND_TODO.md` | Trạng thái hiện tại, việc còn tồn đọng, kế hoạch tiếp theo, thông tin không khôi phục được |
| `CODING_CONVENTIONS.md` | Quy ước đã xác nhận và quy ước đề xuất |
| `CLAUDE_INSTRUCTIONS.md` | Cách người dùng muốn được hỗ trợ; hướng dẫn cho AI tiếp quản |
| `source/` | Prototype HTML, dự án canvas, bảng tính lộ trình, file spec, script patch chưa thành công |
| `screenshots/` | Toàn bộ ảnh MISA người dùng đã gửi, chia theo lần gửi |

**Lưu ý quan trọng:** các liên kết Artifact/Doc tạo trong tài khoản cũ (xem `DEVELOPMENT_HISTORY.md`) là riêng tư của tài khoản đó. Tài khoản mới không mở được; hãy dùng các file trong `source/`.

> **Lưu ý khi dùng repo này (20/11/2026):** trong repo hiện chỉ có 4 file tài liệu (`PROJECT_OVERVIEW.md`, `BUSINESS_REQUIREMENTS.md`, `TECHNICAL_ARCHITECTURE.md`, `ROADMAP.md`). Các file còn lại trong danh sách trên (`CLAUDE_INSTRUCTIONS`, `CURRENT_STATUS_AND_TODO`, `DECISIONS_AND_CHANGELOG`, `DEVELOPMENT_HISTORY`, `CODING_CONVENTIONS`), thư mục `source/` và `screenshots/` chưa có trong repo.
