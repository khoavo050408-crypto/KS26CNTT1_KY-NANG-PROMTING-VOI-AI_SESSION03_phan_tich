# SO SÁNH TRADE-OFF CÁC GIẢI PHÁP RA LỆNH CHO CẤU TRÚC PROMPT TIÊU CHUẨN & ĐỊNH DẠNG

## BƯỚC 1: ĐỀ XUẤT 2 PROMPT MẪU

Cả hai giải pháp dưới đây đều được thiết kế để giải quyết bài toán tiếp nhận đơn hàng thô của khách hàng và chuẩn hóa đầu ra, nhưng sử dụng hai trường phái ra lệnh khác nhau:

### 1. Giải pháp 1: Prompt Đơn giản, Ngắn gọn (Direct & Minimalist)

Phương án này đi thẳng vào yêu cầu cốt lõi, tiết kiệm token, lược bỏ toàn bộ các khung dài dòng, không dùng ví dụ mẫu (Few-shot) và ít ràng buộc định dạng ngặt nghèo.

```markdown
# Yêu cầu nhiệm vụ
Bạn là hệ thống xử lý đơn hàng của cửa hàng. Hãy đọc tin nhắn của khách hàng bên dưới và trích xuất thông tin thành 2 phần:
1. Một bảng Markdown Table tổng hợp các trường: Loại yêu cầu, Tên khách, SĐT, Địa chỉ, Sản phẩm, Xử lý khi hết hàng.
2. Một khối mã JSON chứa dữ liệu tương ứng. Nếu thiếu thông tin nào hãy ghi là "Trống".

# Dữ liệu đầu vào
"Alo shop ơi, giao gấp cho mình 2 ly trà sữa trân châu đường đen size L đến số 12 Nguyễn Trãi nhé, sđt 09000, nhớ bỏ ít đá nha. À mà nếu hết size L thì lấy size M cũng được."
```

### 2. Giải pháp 2: Prompt Chi tiết, Ràng buộc chặt chẽ (Khung CREATE + Few-shot + Ép Schema)

Phương án này áp dụng đầy đủ các kỹ thuật chuyên sâu gồm khung **CREATE**, kỹ thuật học máy **Few-shot** với các trường hợp bẫy dữ liệu, và quy tắc ép cấu trúc đầu ra nghiêm ngặt.

````markdown
# C - Context (Bối cảnh & Vai trò)
Bạn là hệ thống AI tự động hóa vận hành cho chuỗi cửa hàng bán lẻ, chịu trách nhiệm chuẩn hóa tin nhắn đặt hàng thô thành dữ liệu cấu trúc phục vụ hệ thống phần mềm backend.

# R - Request (Yêu cầu cốt lõi)
Phân tích tin nhắn đầu vào của khách hàng, trích xuất chính xác các trường thông tin và xuất ra đúng định dạng Bảng tóm tắt kết hợp khối mã JSON chuẩn.

# E - Explanation (Giải thích & Ràng buộc)
- Tuyệt đối không tự suy diễn hoặc bịa đặt thông tin nếu khách hàng không cung cấp.
- Nếu thiếu Số điện thoại hoặc Địa chỉ, điền giá trị "THIẾU_THÔNG_TIN".
- Nếu số điện thoại không đủ định dạng tiêu chuẩn (ví dụ quá ít chữ số như trường hợp 09000), bắt buộc phải phát hiện và ghi rõ cảnh báo vào trường `error_warning`.
- Đầu ra JSON phải là Valid JSON nguyên khối, không kèm văn bản chào hỏi hay giải thích ngoài luồng.

# A - Action (Các bước hành động)
1. Đọc hiểu và phân loại mục đích tin nhắn.
2. Trích xuất các thực thể: Loại yêu cầu, Tên khách hàng, Số điện thoại, Địa chỉ, Danh sách sản phẩm (Tên, Số lượng, Size, Ghi chú), và Phương án thay thế khi hết hàng.
3. Đối chiếu quy tắc kiểm tra lỗi biên (edge cases) để đưa ra cảnh báo chính xác.

# T - Tone (Giọng điệu văn phong)
Khách quan, chuẩn xác theo định dạng dữ liệu kỹ thuật.

# E - Extras (Định dạng đầu ra bắt buộc)
Phần 1: Bảng tóm tắt nhanh (Markdown Table)
| Trường thông tin | Giá trị trích xuất |
| :--- | :--- |
| Loại yêu cầu | ... |
| Khách hàng | ... |
| Số điện thoại | ... |
| Địa chỉ | ... |
| Chi tiết sản phẩm | ... |
| Xử lý khi hết hàng | ... |
| Cảnh báo lỗi | ... |

Phần 2: Khối dữ liệu JSON chuẩn
```json
{
  "request_type": "...",
  "customer_name": "...",
  "phone": "...",
  "address": "...",
  "items": [
    {
      "product_name": "...",
      "quantity": 0,
      "size": "...",
      "note": "..."
    }
  ],
  "fallback_action": "...",
  "error_warning": null
}
```

# Few-shot Examples (Ví dụ mẫu huấn luyện)
## Ví dụ mẫu (Xử lý bẫy dữ liệu thiếu/sai SĐT):
- Input: "Shop ơi ship cho mình 2 phần cơm gà giòn, sđt 0912..."
- Output Bảng:
| Trường thông tin | Giá trị trích xuất |
| :--- | :--- |
| Loại yêu cầu | DAT_HANG |
| Khách hàng | Khách lẻ |
| Số điện thoại | 0912... |
| Địa chỉ | THIẾU_THÔNG_TIN |
| Chi tiết sản phẩm | 2x Cơm gà giòn |
| Xử lý khi hết hàng | Không có |
| Cảnh báo lỗi | Thiếu địa chỉ giao hàng; Số điện thoại chưa đầy đủ |
- Output JSON:
```json
{
  "request_type": "DAT_HANG",
  "customer_name": "Khách lẻ",
  "phone": "0912...",
  "address": "THIẾU_THÔNG_TIN",
  "items": [{"product_name": "Cơm gà giòn", "quantity": 2, "size": "Tiêu chuẩn", "note": ""}],
  "fallback_action": "Không có",
  "error_warning": "Thiếu địa chỉ giao hàng; Số điện thoại chưa đầy đủ"
}
```

# Input Data (Dữ liệu thực tế cần xử lý)
"Alo shop ơi, giao gấp cho mình 2 ly trà sữa trân châu đường đen size L đến số 12 Nguyễn Trãi nhé, sđt 09000, nhớ bỏ ít đá nha. À mà nếu hết size L thì lấy size M cũng được."
````

---

## BƯỚC 2: LẬP BẢNG SO SÁNH TRADE-OFF

Bảng đánh giá ưu và nhược điểm giữa hai giải pháp dựa trên các tiêu chí vận hành thực tế:

| Tiêu chí | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết) |
| :--- | :--- | :--- |
| **Độ dễ hiểu** | **Cao:** Cấu trúc ngắn gọn, dễ viết và tiếp cận ngay lập tức mà không cần quy chuẩn phức tạp. | **Trung bình:** Yêu cầu tư duy thiết kế cấu trúc chặt chẽ, phân chia rõ ràng các module logic. |
| **Tốn token** | **Ít (~80 tokens):** Tiết kiệm chi phí gọi API và thời gian phản hồi cực nhanh. | **Nhiều hơn (~500 tokens):** Tốn lượng token cao hơn do chứa đầy đủ khung hướng dẫn và ví dụ mẫu. |
| **Chặn lỗi biên** | **Trung bình:** Dễ bị bỏ sót các trường hợp ngoại lệ (như SĐT giả định `09000`), AI dễ tự ý thêm câu chào làm hỏng cấu trúc phân tích. | **Cực tốt:** Kiểm soát chặt chẽ các trường hợp dữ liệu bẫy, bắt lỗi thiếu thông tin và ép buộc cố định định dạng JSON thuần túy. |
| **Tính ổn định Schema** | **Thấp:** Tên biến (key) trong JSON dễ thay đổi ngẫu nhiên qua mỗi lần chạy. | **Tuyệt đối:** Cố định 100% cấu trúc dữ liệu đầu ra, tương thích trực tiếp với hệ thống phần mềm. |

---

## BƯỚC 3: LỰA CHỌN VÀ CHẠY THỬ

### 1. Phương án lựa chọn tối ưu

* **Giải pháp được chọn:** **Giải pháp 2 (Chi tiết, có ràng buộc chặt chẽ theo khung CREATE kết hợp Few-shot).**
* **Lý do lựa chọn:** Trong các bài toán tự động hóa quy trình nghiệp vụ (Business Automation), dữ liệu đầu ra của AI không chỉ để con người đọc mà còn được kết nối trực tiếp vào các hệ thống phía sau (như cơ sở dữ liệu hoặc phần mềm quản lý đơn hàng POS/CRM). Nếu sử dụng Giải pháp 1, cấu trúc JSON rất dễ bị biến đổi qua mỗi lần gọi (lúc thì đổi tên key, lúc thì chèn thêm chữ nghĩa ngoài lề), gây lỗi cú pháp (Parsing Error) cho hệ thống. Do đó, việc đầu tư thêm token ở Giải pháp 2 để đổi lấy tính ổn định tuyệt đối và khả năng chặn lỗi biên (như phát hiện số điện thoại sai `09000`) là sự đánh đổi hoàn toàn xứng đáng.

### 2. Kết quả chạy thử thực tế (Thực thi Giải pháp 2)

Khi đưa đoạn dữ liệu mẫu chứa bẫy số điện thoại (`sđt 09000`) vào Giải pháp 2, hệ thống AI trả về kết quả chính xác như sau:

**Phần 1: Bảng tóm tắt nhanh**

| Trường thông tin | Giá trị trích xuất |
| :--- | :--- |
| Loại yêu cầu | DAT_HANG |
| Khách hàng | Khách lẻ |
| Số điện thoại | 09000 |
| Địa chỉ | Số 12 Nguyễn Trãi |
| Chi tiết sản phẩm | 2x Trà sữa trân châu đường đen (Size: L, Ghi chú: Ít đá; giao gấp) |
| Xử lý khi hết hàng | Đổi sang size M nếu hết size L |
| Cảnh báo lỗi | Số điện thoại không đủ số chuẩn (chỉ có 5 chữ số), có dấu hiệu nhập thiếu |

**Phần 2: Khối dữ liệu JSON chuẩn**

```json
{
  "request_type": "DAT_HANG",
  "customer_name": "Khách lẻ",
  "phone": "09000",
  "address": "Số 12 Nguyễn Trãi",
  "items": [
    {
      "product_name": "Trà sữa trân châu đường đen",
      "quantity": 2,
      "size": "L",
      "note": "Ít đá; giao gấp"
    }
  ],
  "fallback_action": "Đổi sang size M nếu hết size L",
  "error_warning": "Số điện thoại không đủ số chuẩn (chỉ có 5 chữ số), có dấu hiệu nhập thiếu"
}
```