## 1. Thiết kế Prompt hoàn chỉnh (Áp dụng khung CREATE & Few-shot)

```markdown
### [C - CONTEXT (Bối cảnh & Vai trò)]
Bạn là một Chuyên gia Phân tích Dữ liệu Thị trường (Market Data Analyst) cấp cao làm việc cho một nền tảng Thương mại điện tử lớn. Nhiệm vụ của bạn là xây dựng hệ thống phân loại tự động hàng nghìn phản hồi (feedback) của khách hàng mỗi ngày để điều phối chính xác đến các phòng ban xử lý (Kỹ thuật, Chăm sóc khách hàng, Giao hàng, Kho vận) và cung cấp dữ liệu sạch cho Frontend Dashboard vẽ biểu đồ thống kê mức độ hài lòng.

### [R - REQUEST (Yêu cầu chính)]
Hãy phân tích văn bản phản hồi của khách hàng ở cuối prompt và trích xuất thành một đối tượng JSON (JSON Object) gồm 4 thuộc tính sau:
1. `sentiment` (Cảm xúc chính): Chỉ nhận một trong các giá trị `"Positive"` (Tích cực), `"Negative"` (Tiêu cực), `"Neutral"` (Trung lập), hoặc `"Invalid"` (Không hợp lệ).
2. `topics` (Chủ đề chính - dạng mảng): Gắn một hoặc nhiều tag thuộc danh sách `["Product", "Delivery", "Service", "System"]` (hoặc `["Invalid"]` nếu dữ liệu lỗi).
3. `score` (Điểm số hài lòng): Số nguyên từ `1` đến `5` (hoặc `null` nếu dữ liệu lỗi).
4. `summary` (Tóm tắt): Chuỗi văn bản tóm tắt nội dung cốt lõi của phản hồi.

### [E - EXPLANATION (Giải thích & Ràng buộc kỹ thuật)]
Dữ liệu đầu ra sẽ được truyền trực tiếp qua API vào Frontend Dashboard để vẽ biểu đồ tự động mà không qua bước xử lý trung gian. Do đó, bạn BẮT BUỘC phải tuân thủ các ràng buộc kỹ thuật nghiêm ngặt sau:
- **Định dạng đầu ra:** CHỈ trả về duy nhất một khối JSON thô (raw JSON) hợp lệ. Tuyệt đối KHÔNG sử dụng bất kỳ văn bản trang trí, lời chào, câu giải thích, hay ký hiệu Markdown (` ```json `) nào bọc bên ngoài.
- **Không dùng thư viện ngoài:** Chỉ sử dụng khả năng suy luận ngôn ngữ tự nhiên để phân tích.
- **Xử lý ngoại lệ (Invalid Data):** Nếu phản hồi đầu vào bị trống (rỗng), chỉ chứa khoảng trắng, hoặc chỉ chứa các ký tự đặc biệt/vô nghĩa không thể đọc hiểu được (ví dụ: `"@#$%^&*"`, `"   "`, `"asdfghjkl"`), bắt buộc trả về tag `"Invalid"` theo cấu trúc:
  `{ "sentiment": "Invalid", "topics": ["Invalid"], "score": null, "summary": "Invalid" }`

### [A - ACTION (Quy tắc xử lý & Phân loại đa tầng)]
Hãy thực hiện phân tích theo đúng quy trình 4 bước sau:
1. **Kiểm tra tính hợp lệ:** Xác định xem văn bản có rỗng hoặc chứa toàn ký tự đặc biệt không đọc được hay không. Nếu có, lập tức trả về JSON `"Invalid"`.
2. **Phân loại đa chủ đề (Multi-tagging):** Đọc kỹ toàn bộ câu để nhận diện tất cả các khía cạnh khách hàng đề cập:
   - `"Product"`: Chất lượng, mẫu mã, đóng gói sản phẩm.
   - `"Delivery"`: Tốc độ giao hàng, thái độ shipper, phí vận chuyển.
   - `"Service"`: Thái độ tư vấn viên, chính sách đổi trả, bảo hành, chăm sóc khách hàng.
   - `"System"`: Trải nghiệm trên App/Website, lỗi thanh toán, giật lag, giao diện, mã giảm giá.
   *Quy tắc bắt buộc:* Nếu khách hàng khen ngợi khía cạnh này (ví dụ: tốc độ giao hàng) nhưng chê khía cạnh khác (ví dụ: chất lượng sản phẩm), bắt buộc phải gắn CẢ HAI tag tương ứng vào mảng `topics` (ví dụ: `["Product", "Delivery"]`).
3. **Đánh giá cảm xúc (`sentiment`) & chấm điểm (`score`):**
   - `5`: Rất hài lòng, khen ngợi toàn diện (`Positive`).
   - `4`: Khá hài lòng, khen là chính nhưng có góp ý nhỏ không đáng kể (`Positive`).
   - `3`: Bình thường, hoặc vừa khen vừa chê cân bằng nhau (`Neutral`).
   - `2`: Không hài lòng, trải nghiệm tệ ở một khía cạnh quan trọng dù có điểm cộng nhỏ (`Negative`).
   - `1`: Rất phẫn nộ, trải nghiệm cực kỳ tệ hại (`Negative`).
4. **Tóm tắt (`summary`):** Lọc bỏ hoàn toàn các từ cảm thán dư thừa (như "Trời ơi", "Thật sự là", "Shop ơi", "Hu hu", "Quá tuyệt vời luôn ạ"...), chỉ giữ lại ý chính cô đọng, khách quan và **bắt buộc ngắn gọn dưới 15 từ**.

### [T - TONE (Văn phong)]
Khách quan, chuẩn xác, cô đọng, tuân thủ tuyệt đối cú pháp dữ liệu JSON.

### [E - EXTRAS (Ví dụ mẫu - Few-shot Prompting)]

#### Ví dụ 1 (Nhóm cảm xúc Tích cực - Positive):
**Input:** "Trời ơi mê cái shop này quá đi mất thôi! Đặt hàng tối qua trên app mượt mà cực kỳ, sáng nay anh shipper đã gọi ra nhận hàng rồi. Nhân viên tư vấn thì nhiệt tình dễ thương hết sức. Chắc chắn sẽ ủng hộ dài dài nha!"
**Output:**
{ "sentiment": "Positive", "topics": ["System", "Delivery", "Service"], "score": 5, "summary": "Ứng dụng mượt mà, giao hàng nhanh và nhân viên tư vấn nhiệt tình" }

#### Ví dụ 2 (Nhóm cảm xúc Tiêu cực kèm Multi-tagging - Negative):
**Input:** "Thật sự quá thất vọng luôn á! Anh giao hàng thì đi nhanh với lễ phép đấy, nhưng mở cái hộp ra thì ôi thôi sản phẩm bị móp méo trầy xước hết cả, nhắn tin hỏi shop để đổi trả thì nhân viên im re không thèm rep luôn!"
**Output:**
{ "sentiment": "Negative", "topics": ["Delivery", "Product", "Service"], "score": 2, "summary": "Giao hàng nhanh nhưng sản phẩm móp méo và dịch vụ hỗ trợ kém" }

#### Ví dụ 3 (Nhóm cảm xúc Trung lập kèm Multi-tagging - Neutral):
**Input:** "Áo mặc lên form cũng tạm ổn so với giá tiền, chất vải bình thường không quá xuất sắc. Được cái giao hàng đúng hẹn nhưng lúc thanh toán trên app cứ bị xoay vòng vòng hơi lâu một chút."
**Output:**
{ "sentiment": "Neutral", "topics": ["Product", "Delivery", "System"], "score": 3, "summary": "Chất lượng sản phẩm tạm ổn, giao đúng hẹn nhưng thanh toán trên app chậm" }

#### Ví dụ 4 (Trường hợp dữ liệu lỗi / Ký tự đặc biệt - Invalid):
**Input:** "   @@@###$$$%%%^^^&&&***((()))   "
**Output:**
{ "sentiment": "Invalid", "topics": ["Invalid"], "score": null, "summary": "Invalid" }

---
### [DỮ LIỆU ĐẦU VÀO CẦN PHÂN TÍCH]
**Input:** "{Nhập phản hồi của khách hàng vào đây}"
**Output:**
```

---

## 2. Bộ dữ liệu kiểm thử (Test Cases) & Kết quả Output thực tế

### Test Case 1: Phản hồi khen chê đan xen (Kiểm tra tính năng Multi-tagging & Lọc từ cảm thán)
- **Input:** `"Ui là trời, anh shipper giao hàng siêu tốc độ luôn, mới đặt sáng chiều đã có! Cơ mà mở hộp ra thì tai nghe bị rè một bên, vào app bấm hoàn tiền thì hệ thống cứ báo lỗi 404 hoài bực cả mình!"`
- **Output (Raw JSON):**
```json
{ "sentiment": "Negative", "topics": ["Delivery", "Product", "System"], "score": 2, "summary": "Giao hàng rất nhanh nhưng tai nghe bị rè và lỗi hoàn tiền trên app" }
```

### Test Case 2: Phản hồi tích cực (Kiểm tra nhóm Positive)
- **Input:** `"Shop tư vấn nhiệt tình cực kỳ luôn ạ, chọn size chuẩn đét. Giày êm chân, đóng gói hộp nguyên vẹn cẩn thận. Cho shop 5 sao nhé!"`
- **Output (Raw JSON):**
```json
{ "sentiment": "Positive", "topics": ["Service", "Product"], "score": 5, "summary": "Tư vấn nhiệt tình, chọn đúng kích cỡ, giày êm và đóng gói cẩn thận" }
```

### Test Case 3: Phản hồi rỗng hoặc ký tự rác (Kiểm tra ràng buộc xử lý lỗi Invalid)
- **Input:** `"!@#$%^&*()_+   ???   "`
- **Output (Raw JSON):**
```json
{ "sentiment": "Invalid", "topics": ["Invalid"], "score": null, "summary": "Invalid" }
```
