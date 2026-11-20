# BUSINESS_REQUIREMENTS.md

> Nhãn độ tin cậy: xem `PROJECT_OVERVIEW.md` mục 0. Phần "Quan sát từ ảnh MISA" là sự thật về MISA `[TỪ ẢNH]`, **không phải yêu cầu đã chốt** cho hệ thống mới trừ khi mục đó ghi `[XÁC NHẬN]`. Phụ lục A và B là bản đặc tả chi tiết (file spec 01 và 02) được nhúng nguyên văn từ `source/spec/`.

## 1. Yêu cầu nghiệp vụ đã xác nhận

### 1.1. Phạm vi và thứ tự `[XÁC NHẬN]`
- Trang cần xây: Bàn làm việc, Quỹ, Ngân hàng, Mua hàng, Bán hàng, Kho. Không cần: Khế ước vay, Hóa đơn điện tử, Quản lý hóa đơn, Công cụ dụng cụ, Tài sản cố định, Tiền lương.
- Thứ tự xây dựng: **Mua/Bán → Kho → Kế toán**.
- Hạch toán TK Nợ/TK Có đầy đủ như MISA (làm trong giai đoạn kế toán).
- Giữ **tất cả** trường/tab/chức năng MISA của danh mục hàng hóa "tạm thời", việc cắt bớt quyết định sau. (Cách diễn đạt gốc của người dùng: "tạm thời cứ giữ lại tất cả cho tôi" — phản hồi cho danh sách câu hỏi Claude nêu về các tab/trường có dùng hay không; Claude hiểu là giữ toàn bộ. `[XÁC NHẬN]` với cách hiểu này, người dùng không đính chính.)

### 1.2. Người dùng và quyền `[XÁC NHẬN]`
- Admin có toàn quyền; tài khoản khác được admin phân quyền được quản lý nghiệp vụ/chức năng nào.
- Chi tiết vai trò cụ thể (kế toán, thủ kho, nhân viên bán hàng...) và ma trận quyền: `[CHƯA RÕ]`, chưa làm.

### 1.3. Giao diện `[XÁC NHẬN]`
- Giữ bố cục như MISA; làm hiện đại hơn về hình thức; tông xanh lá dịu mặc định (không có màu thương hiệu cố định đã được cung cấp).
- Trang list: vào trang chỉ nút **Thêm** sáng; chọn một dòng thì Sửa, Xóa (và các nút liên quan như Nhân bản) sáng theo dòng đó.
- Wireframe phải là prototype web chạy thật (click trực tiếp), mỗi trang làm dứt điểm rồi mới sang trang sau.

### 1.4. Khởi tạo dữ liệu
- Người dùng: **không cần bê dữ liệu cũ của MISA sang** hệ thống mới. `[XÁC NHẬN]`
- Cách khởi tạo danh mục (hàng hóa ~11k, khách hàng ~14k, nhóm, kho, đơn vị tính...) và số dư đầu kỳ (tồn kho, công nợ): `[CHƯA RÕ]`. Claude từng đề xuất import danh mục từ file Excel xuất của MISA và nhập số dư tại ngày bắt đầu dùng; người dùng chưa trả lời. Dữ liệu cũ có chất lượng chưa tốt (đơn vị tính trùng khác hoa/thường/dấu; cột "Chức danh NLH" lẫn tên riêng) nên cần làm sạch nếu import.

### 1.5. Phi chức năng `[XÁC NHẬN]`
Xem `PROJECT_OVERVIEW.md` mục 6 và `TECHNICAL_ARCHITECTURE.md` mục 1 (2FA, dữ liệu không để người ngoài xem, API/token, backup, ~20 người dùng đồng thời, web trước).

## 2. Quyết định nghiệp vụ theo từng cụm

### 2.1. Danh mục hàng hóa (Vật tư hàng hóa, Nhóm VTHH, Kho, Đơn vị tính)
- Giữ toàn bộ như mục 1.1. Đặc tả chi tiết ở **Phụ lục A**.
- Giả định mặc định Claude đã ghi để không chặn tiến độ (chưa được người dùng trả lời từng điểm) `[GIẢ ĐỊNH]`: cây nhóm nhiều cấp không giới hạn cứng; nút Sửa hàng loạt cần chọn nhiều dòng; Nạp/Nhập khẩu/Xuất khẩu/In luôn sáng; mã hàng nhập tay hoặc để trống cho hệ thống tự sinh; Thời hạn BH là trường tùy chọn; Xóa bị chặn khi đã có tham chiếu.

### 2.2. Đối tượng — Khách hàng
- Quyết định đã xác nhận `[XÁC NHẬN]`:
  1. Mã khách hàng mới tự sinh **ngẫu nhiên 10 chữ số** (kiểm tra không trùng), người dùng sửa được (thường sửa thành số điện thoại sau khi thêm). Trước đó người dùng nói "tự sinh, có thể sửa theo ý thích"; sau đó chốt "cứ sinh ngẫu nhiên 10 số".
  2. Chọn Điều khoản thanh toán thì **tự điền Số ngày được nợ**.
  3. Ô "Nhà cung cấp" chỉ tích khi khách hàng đồng thời là nhà cung cấp vật tư; khách mua bán thông thường không tích.
  4. Mã số thuế trùng: **chỉ cảnh báo**, không chặn.
  5. Hành vi khi vượt "Số nợ tối đa" (cảnh báo hay chặn): **để sau**, chỉ lưu trường.
- Đặc tả chi tiết ở **Phụ lục B**. Một số dòng trong Phụ lục B còn nhãn `[?]` cũ đã được trả lời sau đó; **ưu tiên các mục "Quyết định" ở trên và ở cuối Phụ lục B**.
- Chưa phân tích: Nhà cung cấp, Nhân viên, danh sách Nhóm KH/NCC, danh sách Điều khoản thanh toán. Mẫu mã nhà cung cấp `[CHƯA RÕ]`.

## 3. Quan sát từ ảnh MISA — sáu trang chính `[TỪ ẢNH]`

Đây là bản ghi nhận từ **một ảnh tổng quan mỗi trang**; chưa có ảnh chi tiết từng tab.

### 3.1. Khuôn chung
Thanh trên: menu Tệp, Danh mục, Nghiệp vụ, Hệ thống, Tiện ích, Trợ giúp; ô tìm tính năng (Ctrl + Q); "Đang làm việc trên: [công ty]"; Báo cáo, Tìm chứng từ, Ngày hạch toán; ADMIN. Bên trái: menu con của module và danh sách module (Bàn làm việc, Quỹ, Ngân hàng, Khế ước vay, Mua hàng, Bán hàng, Hóa đơn điện tử, Quản lý hóa đơn, Kho, Công cụ dụng cụ, Tài sản cố định, Tiền lương). Tab ngang trong module. Thanh công cụ: Thêm, Xem, Xóa, Ghi sổ, Bỏ ghi, Xuất khẩu, Góp ý, Giúp (một số trang có Gửi email/Zalo, Nhập khẩu). Bộ lọc: Kỳ, Từ, Đến, Trạng thái, Loại, nút Lấy dữ liệu. Lưới danh sách có hàng lọc theo cột, footer "Số dòng". Lưới "Chi tiết" bên dưới. Có ngày hạch toán khác ngày chứng từ; trạng thái Ghi sổ/Bỏ ghi.

### 3.2. Bàn làm việc
Tab: Tổng quan tài chính, Doanh thu, Chi phí, Lãi lỗ, Công nợ, Tồn kho, Quy trình nghiệp vụ, Thông báo. Widget: Tình hình tài chính (tổng tiền, tiền mặt, tiền gửi, phải thu, phải trả, doanh thu, chi phí, lợi nhuận, hàng tồn kho), Lợi nhuận, Tiền, Phải thu và Phải trả (trong hạn/quá hạn), Doanh thu-chi phí-lợi nhuận theo tháng, Mặt hàng có doanh thu cao nhất, Tình hình sức khỏe doanh nghiệp, Chi phí theo khoản mục chi phí, Phân tích công nợ theo tuổi nợ, Dòng tiền. Bộ lọc kỳ "Đầu năm đến hiện tại"; đơn vị hiển thị Triệu. Phụ thuộc dữ liệu từ các module khác nên dự kiến làm sau cùng `[ĐỀ XUẤT]`.

### 3.3. Quỹ
Menu con: Thu tiền, Thu tiền khách hàng, Thu tiền khách hàng hàng loạt, Chi tiền, Trả tiền nhà cung cấp, Nộp thuế, Nộp bảo hiểm, Trả lương, Kiểm kê. Tab: Thu/chi tiền, Kiểm kê, Dự báo dòng tiền, Báo cáo phân tích, Quy trình. Cột danh sách: Ngày hạch toán, Ngày chứng từ, Số chứng từ (dạng `PT03483`), Đối tượng, Diễn giải, Số tiền, Lý do thu/chi, Ngày ghi sổ quỹ, Loại chứng từ (Phiếu thu...), Số chứng từ CUKCUK; hiển thị "Tồn quỹ đến hiện tại". Lưới chi tiết: Diễn giải, TK Nợ, TK Có, Số tiền, Đối tượng, Tên đối tượng, Mục thu/chi, Số khế ước, Mã thống kê.

### 3.4. Ngân hàng
Menu con: như Quỹ, thêm Chuyển tiền nội bộ; nhóm "Ngân hàng điện tử" (Kết nối ngân hàng điện tử, Thỏa thuận sử dụng dịch vụ). Tab: Thu/chi tiền, Đối chiếu ngân hàng, Ngân hàng điện tử, Dự báo dòng tiền, Báo cáo phân tích, Quy trình. Cột: Ngày hạch toán, Số chứng từ (tiền tố `UNC`, `NTTK`, `CTNB`), Số tài khoản NH, Đối tượng, Diễn giải, Số tiền, Lý do thu/chi, Loại chứng từ (Ủy nhiệm chi, Thu tiền gửi, Chuyển tiền nội bộ); bộ lọc có TK ngân hàng; hiển thị "Tổng số dư tiền gửi". Lưới chi tiết thêm: Khoản mục CP, Đơn vị, Đối tượng THCP, Công trình, Đơn đặt hàng, Đơn mua hàng, Hợp đồng mua, Hợp đồng bán.

### 3.5. Mua hàng
Menu con: Đơn mua hàng, Hợp đồng mua hàng, Chứng từ mua hàng hóa, Chứng từ mua hàng nhiều..., Chứng từ mua dịch vụ, Trả lại hàng mua, Giảm giá hàng mua, Nhận hóa đơn, Trả tiền nhà cung cấp, Đối trừ chứng từ, Đối trừ chứng từ nhiều đối tượng, Bỏ đối trừ, Bù trừ công nợ. Tab: Đơn mua hàng, Hợp đồng mua hàng, Hóa đơn đầu vào, Mua hàng hóa dịch vụ, Nhận hóa đơn, Trả lại hàng mua, Giảm giá hàng mua, Công nợ, Báo cáo phân tích, Quy trình. Đơn mua hàng: Tình trạng (Chưa thực hiện / Hoàn thành), Ngày đơn hàng, Số đơn hàng (dạng `DMH16343-27`), Ngày giao hàng, Nhà cung cấp, Diễn giải, Giá trị đơn hàng. Chi tiết: Mã hàng, Tên hàng, Mã quy cách, ĐVT, Số lượng, Số lượng nhận, Đơn giá sau thuế, Đơn giá, Thành tiền, Tỷ lệ CK (%), Tiền chiết khấu, % thuế GTGT, Tiền thuế GTGT.

### 3.6. Bán hàng
Menu con: Báo giá, Đơn đặt hàng, Chứng từ bán hàng, Hóa đơn, Giảm giá hàng bán, Trả lại hàng bán, Doanh thu nhận trước, Thu tiền khách hàng, Thu tiền khách hàng hàng loạt, Đối trừ chứng từ, Đối trừ chứng từ nhiều đối tượng, Bỏ đối trừ, Bù trừ công nợ, Đợt thu nợ. Tab: Báo giá, Đơn đặt hàng, Đề nghị ghi nhận doanh thu, Bán hàng, Xuất hóa đơn, Trả lại hàng bán, Giảm giá hàng bán, Thu nợ, Công nợ, Báo cáo phân tích, Quy trình. Danh sách bán hàng: Ngày hạch toán, Ngày chứng từ, Số chứng từ (dạng `B059933`), Số hóa đơn, Khách hàng, Diễn giải, Tổng tiền hàng, Tiền chiết khấu, Tiền thuế GTGT, Tổng tiền thanh toán, Kiêm phiếu xuất kho, Đã xuất hóa đơn, Mã nhân viên, Nhân viên, Người tạo; bộ lọc có "Lập HĐ". Chi tiết: Mã hàng, Tên hàng, Kho, Hàng khuyến mại, ĐVT, Số lượng, Đơn giá, Thành tiền, Tỷ lệ CK, Tiền chiết khấu, % thuế, Tiền thuế GTGT.

### 3.7. Kho
Menu con: Nhập kho, Xuất kho, Chuyển kho, Lệnh sản xuất, Lệnh lắp ráp tháo dỡ, Tính giá xuất kho, Kiểm kê. Tab (một phần bị che): Chuyển kho, Lệnh sản xuất, Lắp ráp tháo dỡ, Kiểm kê, Báo cáo phân tích, Quy trình. Danh sách: Ngày hạch toán, Ngày chứng từ, Số chứng từ (dạng `XK063602`), Diễn giải, Tổng tiền, Người giao/Người nhận, Đối tượng, Đã lập CT bán hàng, Ngày ghi sổ kho, Loại chứng từ (Xuất kho bán hàng, Xuất kho khác, Xuất kho từ kiểm kê). Chi tiết: Mã hàng, Tên hàng, Kho, TK Nợ, TK Có, ĐVT, Số lượng, Đơn giá, Thành tiền, **Số lô**, **Hạn sử dụng**, Số CT đối trừ, Đơn đặt hàng, Số khế ước.

### 3.8. Liên kết giữa module nhìn thấy trong ảnh
Đơn mua hàng → chứng từ mua → nhập kho (cột Số lượng nhận). Chứng từ bán hàng có tùy chọn **kiêm phiếu xuất kho**; phiếu xuất có cờ "Đã lập CT bán hàng". Thu/trả tiền gắn công nợ qua đối trừ chứng từ và bù trừ công nợ. Kho có số lô, hạn sử dụng và chức năng tính giá xuất kho.

## 4. Quan sát từ ảnh MISA — menu Danh mục `[TỪ ẢNH]`

- Tài khoản: Hệ thống tài khoản, Tài khoản kết chuyển, Tài khoản ngầm định, Định khoản tự động.
- Cơ cấu tổ chức.
- Đối tượng: Khách hàng, Nhà cung cấp, Nhân viên, Nhóm khách hàng/nhà cung cấp.
- Lương nhân viên: Ký hiệu chấm công, Biểu tính thuế thu nhập.
- Loại công cụ dụng cụ; Loại tài sản cố định.
- Vật tư hàng hóa: Vật tư hàng hóa, Kho, Nhóm vật tư hàng hóa dịch vụ, Đơn vị tính.
- Khoản mục chi phí.
- Ngân hàng: Ngân hàng, Tài khoản ngân hàng.
- Khác: Điều khoản thanh toán, Mục thu/chi, Mã thống kê, Biểu thuế tiêu thụ đặc biệt, Biểu thuế tài nguyên, Loại tiền, Hình thức thanh toán, Mẫu số hóa đơn.

Hệ thống tài khoản `[TỪ ẢNH]`: phân cấp (ví dụ 1121 chia thành 1121.1, 1121.2...), có cột Tính chất (Dư Nợ) và tên tiếng Anh. "Định khoản tự động" là bảng quy tắc hạch toán: mỗi dòng gồm loại nghiệp vụ (Chi tiền mặt, Thu tiền gửi, Nghiệp vụ khác), tên định khoản và tài khoản (ví dụ chi tiền điện cho quản lý doanh nghiệp → 6427; chi trả tiền nhà cung cấp → 331; chi tạm ứng cho nhân viên → 141; hạch toán thuế TNDN phải nộp → 8211; thu lãi tiền gửi ngân hàng → 1121.1).

### 4.1. Phân loại danh mục Claude đề xuất `[ĐỀ XUẤT]` (người dùng chưa xác nhận từng dòng)

| Nhóm | Danh mục |
|---|---|
| Cần ngay (giai đoạn nền/Mua-Bán) | Vật tư hàng hóa, Kho, Nhóm VTHH, Đơn vị tính; Khách hàng, Nhà cung cấp, Nhân viên, Nhóm KH/NCC; Hệ thống tài khoản; Loại tiền; Điều khoản thanh toán; Hình thức thanh toán |
| Giai đoạn kế toán | Tài khoản ngầm định, Tài khoản kết chuyển, Định khoản tự động; Ngân hàng, Tài khoản ngân hàng; Mục thu/chi; Khoản mục chi phí; Mã thống kê |
| Có thể bỏ | Lương nhân viên, Loại công cụ dụng cụ, Loại tài sản cố định, Biểu thuế tiêu thụ đặc biệt, Biểu thuế tài nguyên, Mẫu số hóa đơn |
| Cần cân nhắc | Cơ cấu tổ chức (phòng ban, chi nhánh): `[CHƯA RÕ]`, người dùng chưa trả lời |

## 5. Quy mô và hiệu năng cần đáp ứng
- Danh sách lớn phải phân trang/lọc/tìm kiếm phía server (11.224 hàng hóa, 14.073 khách hàng, 7.844 chứng từ bán hàng trong ~2,5 tháng). `[TỪ ẢNH]` + `[ĐỀ XUẤT]`
- Kiến trúc một server + một database là đủ so với khối lượng này. `[ĐỀ XUẤT]`

## 6. Câu hỏi nghiệp vụ còn mở (tổng hợp, chi tiết hơn ở `CURRENT_STATUS_AND_TODO.md`)
1. Thu tiền khách hàng / trả tiền nhà cung cấp có đưa vào giai đoạn Mua/Bán để theo dõi công nợ thanh toán, hay để cùng Quỹ/Ngân hàng ở giai đoạn kế toán? (Claude hỏi, chưa được trả lời.)
2. Module kế toán của hệ thống mới phục vụ quản trị nội bộ hay phải đảm bảo số liệu kế toán chính thức (thuế, báo cáo tài chính)? (Claude hỏi nhiều lần, chưa được trả lời rõ; vì MISA vẫn giữ song song nên có thể chỉ phục vụ quản trị nội bộ, nhưng người dùng chưa khẳng định.) `[CHƯA RÕ]`
3. Cơ cấu tổ chức (phòng ban/chi nhánh) có cần không?
4. Cách khởi tạo danh mục và số dư đầu kỳ.
5. Các trường liên quan hóa đơn trên chứng từ bán hàng (Số hóa đơn, Đã xuất hóa đơn, Lập HĐ) có giữ không, khi menu Hóa đơn điện tử/Quản lý hóa đơn không làm?
6. Mẫu mã tự sinh của nhà cung cấp, nhân viên, hàng hóa.
7. Tính chất "Thời hạn BH" của hàng hóa là bảo hành hay hạn dùng, và hạn sử dụng trên phiếu nhập kho có tự tính từ đó không.
8. Hạn mức công nợ ("Số nợ tối đa") khi vượt: cảnh báo hay chặn (người dùng: để sau).
9. Vai trò và ma trận phân quyền cụ thể.

---

<!-- Phụ lục A và B được nối tự động bên dưới -->
