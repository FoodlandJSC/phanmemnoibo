# TECHNICAL_ARCHITECTURE.md

> Nhãn độ tin cậy: xem `PROJECT_OVERVIEW.md` mục 0. **Điểm then chốt: chưa có mã nguồn production.** Phần lớn nội dung dưới đây là đề xuất của Claude chưa được người dùng xác nhận rõ ràng; chỉ các ràng buộc ở mục 1 là `[XÁC NHẬN]`.
>
> **Cập nhật 20/11/2026:** các quyết định công nghệ Frontend (Ant Design, AG Grid, monorepo pnpm, feature-based) nằm ở `ROADMAP.md` mục 4 và có hiệu lực hơn các đề xuất FE trong tài liệu này (mục 4, 5).

## 1. Ràng buộc kỹ thuật đã xác nhận `[XÁC NHẬN]`

- Người dùng là web developer; stack chính: **NodeJS, ReactJS, TypeScript**.
- Chạy trên web trước; có thể đóng gói thành web app sau.
- Dùng chủ yếu trong mạng công ty; cần bảo trì, nâng cấp, sửa lỗi từ xa.
- Khoảng 20 người dùng đồng thời.
- Mã nguồn tổ chức theo module, chia thư mục rõ ràng cho từng chức năng, ngay từ ngày đầu.
- Phân quyền: admin full; tài khoản khác được admin cấp quyền theo nghiệp vụ/chức năng.
- Bảo mật: bảo vệ cấp 2 (2FA); người ngoài không xem được dữ liệu; API và token xác thực không để người ngoài biết.
- Có phương án backup khi triển khai lên server.
- Chưa có tích hợp; sẽ mở rộng sau.
- Hệ thống mới độc lập hoàn toàn với MISA.

## 2. Kiến trúc đề xuất `[ĐỀ XUẤT]` (người dùng chưa phản đối, chưa xác nhận rõ)

Các điểm dưới đây Claude đã nêu để trả lời các câu hỏi của người dùng ("monolith có cần server mạnh không", "module từ ngày đầu", "không dùng queue/Kafka/Redis có ổn không").

### 2.1. Modular monolith
- Một khối code triển khai một chỗ; tải thực tế thấp (xem `PROJECT_OVERVIEW.md` mục 2, 4) nên không cần microservice.
- Cấu hình server tham khảo: 2–4 vCPU, 4–8 GB RAM, SSD; Node.js và PostgreSQL có thể chạy chung một máy lúc đầu; tách database ra máy riêng sau nếu cần. Đầu tư kỹ hơn cho **backup** (đích riêng, tự động, có kiểm thử khôi phục).
- Ranh giới module: mỗi module sở hữu routes/controller, service, repository, schema, dto, events của riêng nó; **chỉ giao tiếp với module khác qua service công khai hoặc event nội bộ (in-process)**, không truy cập thẳng bảng của module khác.
- Cấu trúc thư mục đề xuất:

```
src/
├── modules/
│   ├── auth/            # đăng nhập, 2FA, phân quyền
│   ├── catalog/         # sản phẩm, khách hàng, NCC (danh mục dùng chung)
│   ├── purchasing/      # mua hàng
│   ├── sales/           # bán hàng
│   ├── inventory/       # kho
│   ├── accounting/      # kế toán
│   └── reports/
│       └── (mỗi module gồm: controller, service, repository, dto, events)
├── shared/              # tiện ích dùng chung, không chứa nghiệp vụ
└── main.ts
```
- Luồng ví dụ: `sales` xác nhận đơn → phát event `OrderConfirmed` → `inventory` và `accounting` lắng nghe để xuất kho và ghi bút toán.
- Framework chưa chọn. Claude nhắc NestJS hỗ trợ module sẵn nhưng nói Express/Fastify cũng được nếu kỷ luật đủ chặt. `[CHƯA RÕ]` lựa chọn cuối.

### 2.2. Cơ sở dữ liệu và xử lý nền
- Đề xuất **PostgreSQL**. `[ĐỀ XUẤT]` (người dùng không nêu DB cụ thể).
- Không dùng Kafka (quá mức cần thiết). Chưa dùng Redis/queue/cache ở giai đoạn đầu; PostgreSQL có index đúng là đủ nhanh. Nếu sau này cần tác vụ chạy lâu (gửi email hàng loạt, xuất báo cáo nặng) thì dùng thư viện **pg-boss** (queue chạy trên PostgreSQL, không cần hạ tầng mới). Chỉ nghĩ tới Redis khi có dấu hiệu rõ.
- Rủi ro thật cần xử lý: **giao dịch (transaction) và khóa dòng** ở kho và kế toán để tránh ghi đè/ghi dở dang (ví dụ hai người cùng xuất một mặt hàng tồn ít).
- Báo cáo lớn và xuất Excel/PDF: tối ưu truy vấn, đánh index, phân trang trước khi nghĩ tới queue.
- Danh sách lớn (11.224 hàng hóa, 14.073 khách hàng): phân trang, lọc, tìm kiếm **phía server**.

### 2.3. Bảo mật `[ĐỀ XUẤT]`
Claude đã đính chính một hiểu lầm: với web app, người dùng đã đăng nhập luôn xem được request/response trong DevTools nên không thể "giấu API khỏi người đang dùng"; bảo vệ thật sự là chặn người ngoài và chặn người trong làm việc vượt quyền. Đề xuất:
- Phân quyền kiểm tra **ở server** cho mọi API, không chỉ ẩn nút trên giao diện.
- Token trong cookie `httpOnly` + `Secure` + `SameSite`; access token ngắn hạn kèm refresh token.
- 2FA (TOTP qua ứng dụng Authenticator), áp dụng cho admin trước rồi mở rộng.
- Giới hạn mạng: chỉ cho truy cập qua mạng công ty hoặc VPN, hoặc IP whitelist.
- HTTPS bắt buộc; giới hạn tần suất đăng nhập; băm mật khẩu (argon2/bcrypt); nhật ký thao tác (audit log).
- File (ví dụ ảnh hàng hóa) lưu trên server/object storage, phục vụ qua endpoint **có xác thực**, không dùng URL công khai.
- Dữ liệu cá nhân (số điện thoại, địa chỉ, số CMND/CCCD của khách cá nhân): phân quyền xem, che một phần khi hiển thị, ghi nhật ký truy cập; Xuất khẩu và Nhập khẩu là quyền riêng. Người dùng được khuyên tự kiểm tra quy định bảo vệ dữ liệu cá nhân hiện hành (Claude không xác minh điều luật cụ thể).

## 3. Mô hình dữ liệu khái niệm

Đặc tả chi tiết từng bảng nằm ở **phụ lục của `BUSINESS_REQUIREMENTS.md`** (spec 01 và 02). Dưới đây là nguyên tắc chung và các bảng đã được nêu tên.

### 3.1. Nguyên tắc
- Khóa chính là `id` nội bộ; **mọi FK theo `id`**, nên sửa `code` không ảnh hưởng chứng từ. `[ĐỀ XUẤT]`
- `code` luôn là **chuỗi** (text), unique không phân biệt hoa thường: có mã vạch 13 số, có số 0 đứng đầu, có mã là số điện thoại hoặc mã số thuế kèm hậu tố chi nhánh. `[TỪ ẢNH]` + `[ĐỀ XUẤT]`
- Tiền, đơn giá, số lượng dùng kiểu `numeric`, không dùng float. `[ĐỀ XUẤT]`
- Không lưu các giá trị tính toán (ví dụ `stock_qty`, `stock_value`); lấy từ sổ biến động tồn kho. `[ĐỀ XUẤT]`
- Dùng cờ `is_inactive` (Ngừng theo dõi); đề xuất chặn xóa khi đã có chứng từ tham chiếu. `[ĐỀ XUẤT]` (một phần `[GIẢ ĐỊNH]`, chưa được xác nhận)
- Thuế suất lưu ở bảng `tax_rates`, không viết cứng; 0% khác KCT (không chịu thuế). `[ĐỀ XUẤT]`
- Hạch toán kế toán nằm trong từng chứng từ (cột TK Nợ/TK Có ở Quỹ, Ngân hàng, Kho; trạng thái Ghi sổ/Bỏ ghi): cần hệ thống tài khoản + bảng bút toán (mỗi bút toán gồm các dòng TK Nợ, TK Có, số tiền, gắn chứng từ gốc) ngay từ giai đoạn nền; engine tự sinh bút toán và báo cáo làm ở giai đoạn kế toán; chứng từ lưu đủ thông tin để tính lại bút toán. Quy tắc hạch toán dự kiến lưu thành **bảng cấu hình** (loại nghiệp vụ → TK Nợ/TK Có) giống "Định khoản tự động" của MISA. `[ĐỀ XUẤT]`
- Sổ biến động tồn kho (stock movement ledger): mỗi dòng mua/bán ghi một biến động nhập/xuất; giai đoạn Kho dựng thêm nghiệp vụ trên dữ liệu đó. `[ĐỀ XUẤT]`

### 3.2. Bảng/thực thể đã được nêu tên (chi tiết cột ở phụ lục `BUSINESS_REQUIREMENTS.md`)
`units`, `warehouses`, `item_groups`, `item_group_items` (nhiều-nhiều), `items`, `item_unit_conversions`, `item_quantity_discounts`, `item_spec_definitions`, `tax_rates`, `files`, `parties` (khách hàng/nhà cung cấp/nhân viên dùng chung, cờ `is_customer`, `is_supplier`, `is_employee`), `party_groups`, `party_group_members`, `party_bank_accounts`, `party_delivery_locations`, `payment_terms`.
Thực thể cần có nhưng **chưa được đặc tả**: `accounts` (hệ thống tài khoản), bảng bút toán, sổ biến động tồn kho, chứng từ mua/bán/kho/quỹ/ngân hàng, người dùng/vai trò/quyền, nhật ký thao tác. `[CHƯA RÕ]`

### 3.3. Nhận xét về cột `Mã quy cách`
Dòng chứng từ (ví dụ Đơn mua hàng) có cột "Mã quy cách" `[TỪ ẢNH]`; đề xuất dòng chứng từ có thêm `spec_1..spec_5` nullable; độ ưu tiên thấp. `[ĐỀ XUẤT]`

## 4. Kiến trúc giao diện

### 4.1. Hướng đã chốt `[XÁC NHẬN]`
- Giữ **bố cục giống MISA** vì nhân viên đã quen; làm mới phần hình thức theo hướng hiện đại.
- Cấu trúc màn hình MISA lặp lại ở hầu hết trang: thanh trên (menu, tìm tính năng, công ty, Báo cáo/Tìm chứng từ/Ngày hạch toán) → menu con và danh sách module bên trái → tab ngang → thanh công cụ (Thêm, Nhân bản, Sửa, Xóa, Nạp, Nhập khẩu, Xuất khẩu...) → bộ lọc (Kỳ, Từ, Đến, Trạng thái, Lấy dữ liệu) → lưới danh sách có hàng lọc theo cột và dòng tổng → lưới "Chi tiết" bên dưới.
- Giao diện hiện đại: tông xanh lá dịu (không có màu thương hiệu cố định; có thể đổi), font Inter, icon nét mảnh, tab gạch chân, nhãn trạng thái màu, góc bo nhẹ, mật độ hàng gọn. Gom màu/font/khoảng cách vào biến thiết kế (design tokens).
- Quy tắc bật/tắt nút: vào trang chỉ **Thêm** sáng; chọn một dòng thì Nhân bản, Sửa, Xóa sáng theo dòng đó.
- Giữ phím tắt kiểu MISA: **F9 = Thêm nhanh**, **F3 = Tìm nhanh** (MISA hiển thị gợi ý ở chân form). `[ĐỀ XUẤT]` giữ nguyên để người dùng quen tay (người dùng chưa phản hồi riêng điểm này).

### 4.2. Đề xuất triển khai `[ĐỀ XUẤT]`
- Bộ component dùng chung điều khiển bằng cấu hình: lưới danh sách + lưới chi tiết + thanh công cụ + bộ lọc kỳ; mỗi loại chứng từ/danh mục chỉ cấu hình cột, trường, nút.
- Component `LookupSelect`: dropdown nhiều cột (mã + tên), gõ để tìm, nút "+" tạo nhanh bản ghi liên quan trong hộp thoại nhỏ.
- Hộp thoại chọn nhóm: cây có checkbox chọn nhiều; nút Thêm nhanh | Đồng ý | Hủy bỏ.
- Có tùy chọn mật độ hiển thị (gọn / thoải mái), mặc định gọn.

## 5. Mã nguồn và tài sản hiện có

Không có repository. Các file thực sự tồn tại (bản sao nằm trong `source/`):

> **Cập nhật 20/11/2026:** đã có repo `FoodlandJSC/phanmemnoibo` (xem `ROADMAP.md`). Các file trong bảng dưới thuộc bản bàn giao cũ, hiện **chưa có trong repo này**.

| File | Vai trò | Ghi chú |
|---|---|---|
| `source/foodland-prototype.html` | Prototype web tương tác một file (4 trang danh mục) | Xem 5.1. Đã đăng làm Artifact trong tài khoản cũ |
| `source/canvas-project/*.dc.html`, `canvas.json` | Dự án canvas thiết kế (loại Design của Claude) | Xem 5.2 |
| `source/Foodland_Lo_trinh_thiet_ke.xlsx` | Bảng tính lộ trình 7 bước + tóm tắt phân tích | Tạo sớm; một phần đã lỗi thời |
| `source/spec/01_danh_muc_hang_hoa.md`, `02_doi_tuong.md` | Đặc tả phân tích đang tích lũy | Cũng được nhúng ở phụ lục `BUSINESS_REQUIREMENTS.md` |
| `source/patch_khach_hang.py.txt` | Script đã soạn để thêm trang Khách hàng vào prototype | **Chưa chạy thành công**, xem 5.3 |

### 5.1. `foodland-prototype.html`
- Một file HTML tự chứa, JavaScript thuần (không framework), CSS biến (tokens) có chế độ sáng/tối; chỉ dùng Google Fonts. Dữ liệu chỉ lưu trong bộ nhớ trình duyệt (tải lại là về dữ liệu mẫu). Dữ liệu mẫu là giá trị giả, không phải dữ liệu thật.
- Trang (đường dẫn hash): `#vat-tu` (Vật tư hàng hóa), `#nhom` (Nhóm vật tư, hàng hóa, dịch vụ), `#kho` (Kho), `#don-vi-tinh` (Đơn vị tính). Menu **Danh mục ▾** ở thanh trên để chuyển trang; nút Back của trình duyệt hoạt động (listener `hashchange`).
- Cấu trúc mã (các tên thật trong file):
  - Hằng/biến: `$`, `esc`, `num`, `NS` (kind không có tồn kho: `['Dịch vụ','Chỉ là diễn giải']`), `uid`, `D` (dữ liệu: `units`, `wh`, `groups`, `items`), `specs`, `itemDef`, `groupOpts`, `P` (cấu hình trang), `S` (trạng thái), `O`, `clone`, `act` (bảng hành động).
  - Hàm: `desc(id)` (nhóm con cháu), `toast`, `rowsOf` (lọc + duyệt cây), `renderView`, `renderGrid`, `detail`, `setBtns`, `fld`, `tbl`, `renderModal`, `blank`, `openForm`, `save`, `go`.
  - `P[page]` có các khóa: `d` (tên mảng trong `D`), `t` (tiêu đề), `key` (trường khóa duy nhất: `code` hoặc `name`), `cols` (`[khóa, nhãn, độ rộng, 'r' nếu canh phải]`), `f` (trường chính của form), `tabs` (các tab với `n`, `f`, `note`, `tb`), `tb` (lưới con), `def` (giá trị mặc định), `used` (hàm trả thông báo chặn xóa), `tree` (hiển thị dạng cây).
  - `S`: `page`, `sel{}` (dòng chọn theo trang), `q`, `gf`, `open{}` (cây), `modal`, `form`, `mtab`, `ptab`, `err`, `conf`.
  - `act`: `soon`, `menu`, `go`, `pick`, `tog`, `togall`, `reload`, `ptab`, `add`, `clone`, `edit`, `del`, `delok`, `cancel`, `save`, `savenew`, `mtab`, `addrow`.
  - Sự kiện: `click`, `dblclick` (mở Sửa), `input`, `change`, `keydown` (Esc đóng hộp thoại), `hashchange`; nhận diện hành động bằng thuộc tính `data-a`, `data-id`, `data-k`, `data-t`...
- Hành vi đã có: chỉ Thêm sáng lúc đầu, chọn dòng mới bật Nhân bản/Sửa/Xóa; tìm kiếm; lọc theo nhóm (trang Vật tư); panel chi tiết 2 tab (trang Vật tư); form 5 tab của Vật tư hàng hóa (Ngầm định, Chiết khấu, Đơn vị chuyển đổi, Mã quy cách, Đặc tính/hình ảnh); hàng Dịch vụ/Chỉ là diễn giải khóa ô Kho ngầm định và Tài khoản kho; cây Nhóm với mở/thu gọn và lưới hàng hóa trong form; Cất / Cất & Thêm / Hủy bỏ; báo lỗi thiếu hoặc trùng mã; xác nhận xóa.
- Ràng buộc liên trang (**đây là giả định của Claude, người dùng chưa xác nhận**): xóa nhóm còn nhóm con hoặc đang có hàng hóa bị chặn; xóa đơn vị tính hoặc kho đang được hàng hóa dùng bị chặn; thêm đơn vị tính mới thì xuất hiện trong ô ĐVT của form hàng hóa.
- Chưa có: Nhập khẩu, Xuất khẩu, In, Sửa hàng loạt (nút chỉ báo "chưa có trong prototype"); lưu trữ bền vững; phân quyền; trang Khách hàng.
- Giới hạn của môi trường đăng Artifact: trang chỉ tải script từ vài host cho phép và không gọi được mạng ra ngoài; vì vậy prototype dùng JS thuần, không thư viện. Không áp dụng cho sản phẩm thật.

### 5.2. Dự án canvas (`source/canvas-project/`)
Tạo bằng loại artifact "Design" của Claude. `canvas.json` khai báo 6 board:

| File board | Nội dung | Trạng thái |
|---|---|---|
| `Main.dc.html` | Wireframe khung chung (xám) | Tĩnh |
| `Modern.dc.html` | Cùng bố cục, giao diện hiện đại tông xanh lá (bản được duyệt) | Tĩnh |
| `DanhMucHangHoa.dc.html` | Danh sách Vật tư hàng hóa | Tĩnh, **thừa** (đã có bản tương tác) |
| `FormHangHoa.dc.html` | Form Thêm vật tư hàng hóa | Tĩnh, **thừa** |
| `NhomVTHH.dc.html` | Nhóm VTHH | Tương tác (cần bấm Play trong canvas) |
| `VatTuHangHoa.dc.html` | Vật tư hàng hóa | Tương tác (cần bấm Play) |

Lưu ý: định dạng `.dc.html` cần `support.js` do môi trường Design cung cấp; file không tự chạy khi mở rời. Các board này **chỉ dùng được trong môi trường Claude có loại Design**. Sau khi người dùng yêu cầu "tương tác như web thật", hướng chính đã chuyển sang `foodland-prototype.html`; canvas chỉ còn là tham khảo.

### 5.3. Script patch chưa thành công
`patch_khach_hang.py.txt` (bản gốc ở `/tmp/patch.py`) là script Python dùng `str.replace` với kiểm tra mỗi mẫu phải xuất hiện đúng 1 lần để thêm trang Khách hàng vào `foodland-prototype.html`. Nó **thất bại 2 lần và không ghi gì vào file** (xem `DEVELOPMENT_HISTORY.md`). Lỗi đã biết: dòng thay thế `let v=c[0]==='off'` bị gõ thiếu một dấu `=` (phải là `===`). Chưa kiểm tra các mẫu còn lại, chưa chạy `node --check` thành công. Script cho thấy thiết kế dự kiến: thêm dữ liệu `D.terms`, `D.kg`, `D.emp`, `D.cust`; cấu hình trang `'khach-hang'` với các khóa mới `autocode`, `warnTax`, `kf`, `head`, `hint`, `chg`, `detail`, trường có điều kiện `if:'org'|'ind'`, kiểu trường `t:'m'` (chọn nhiều nhóm) và `qa` (nút "+" thêm nhanh); hộp thoại phụ chọn nhóm, thêm Điều khoản thanh toán và cảnh báo trùng mã số thuế; phím F9.

## 6. Hạ tầng và triển khai `[CHƯA RÕ]`
Chưa có quyết định về máy chủ cụ thể, hệ điều hành, container, CI/CD, môi trường dev/prod, chiến lược backup chi tiết, giám sát. Người dùng chỉ nói "sẽ có phương án backup khi triển khai lên server".

## 7. Khoảng trống kỹ thuật chưa giải quyết
- Chưa chọn framework backend, ORM, thư viện xác thực/2FA, thư viện giao diện React.
- Chưa có ERD tổng, đặc tả API, ma trận phân quyền.
- Chưa có thiết kế quy tắc hạch toán tự động (cần ảnh cấu hình "Định khoản tự động"/"Tài khoản ngầm định" của MISA; mới thấy một phần).
- Chưa quyết định cách khởi tạo dữ liệu danh mục ban đầu (xem `BUSINESS_REQUIREMENTS.md` mục "Khởi tạo dữ liệu").
