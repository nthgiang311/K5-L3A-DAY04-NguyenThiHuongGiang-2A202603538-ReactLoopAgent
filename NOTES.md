# Ghi chú phân tích · Day 04

**Họ và tên:** Nguyễn Thị Hương Giang 
**MSSV:** 2A202603538

## 1. Case được chọn

- **ID case:** A6
- **Câu trả lời của agent:** Agent điều tra incident GitHub `h02n9572mx9y`, phát hiện incident này có liên quan đến incident tổng hợp `xr3myzj7ms51`. Agent cần tổng hợp thông tin từ cả hai incident, bao gồm trạng thái, ảnh hưởng, nguyên nhân và các URL nguồn.
- **Điểm ở lần chạy 1:** 79/100.

## 2. Bằng chứng trong trace

Trace cho thấy agent đã gọi `get_incident_details` để lấy thông tin của hai incident. Sau đó, bước validation ghi nhận các URL nguồn quan sát được:

```text
{
  'event': 'validation_missing_source',
  'step': 3,
  'urls': [
    'https://stspg.io/7pwv28hvkmz8',
    'https://stspg.io/pjgptg3cfc6n'
  ]
}
```

**Nguyên nhân có thể xảy ra:**

Ở phiên bản trước, hướng dẫn validation tập trung vào việc bổ sung URL còn thiếu, nhưng chưa nhấn mạnh rõ việc phải bảo toàn toàn bộ URL nguồn đã được quan sát từ các tool call.

Với câu hỏi cần tổng hợp nhiều incident có liên quan, việc chỉ chú ý đến URL còn thiếu có thể khiến agent bỏ sót nguồn của một incident khi tạo câu trả lời cuối cùng.

Lưu ý: Trong kết quả A6 hiện tại, cả hai phiên bản đều có hai URL trong câu trả lời cuối. Vì vậy, trace cho thấy điểm cần cải thiện trong logic validation, nhưng chưa đủ để khẳng định phiên bản trước thực sự làm mất URL ở case này.

## 3. Thay đổi đã thực hiện

- **Sửa gì, ở đâu:** Sửa phần validation prompt trong file `agent.py`.

  - Trước khi sửa: Prompt tập trung vào việc bổ sung các URL nguồn còn thiếu.
  - Sau khi sửa: Prompt yêu cầu agent sử dụng và bảo toàn tất cả URL nguồn đã quan sát được, đồng thời bổ sung URL còn thiếu nếu cần.

- **Vì sao chọn thay đổi này:**

  Case A6 yêu cầu tổng hợp thông tin từ nhiều incident liên quan. Do đó, việc bảo toàn đầy đủ nguồn tham chiếu giúp câu trả lời có thể đối chiếu với dữ liệu từ từng incident, hạn chế tình trạng bỏ sót nguồn khi tổng hợp thông tin.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---:|---:|
| Điểm tổng (trung bình 22 case) | 89.14 | 91.59 |
| Điểm case A6 | 79 | 98 |
| Số model calls (A6) | 4 | 4 |
| Token (A6) | 7,310 | 7,751 |
| Chi phí | Không có dữ liệu | Không có dữ liệu |

- **Lỗi đã giải quyết:** Phiên bản sau có điểm A6 tăng từ 79 lên 98. Câu trả lời thể hiện rõ mối liên hệ giữa hai incident, tổng hợp được thông tin nguyên nhân và ảnh hưởng, đồng thời phân biệt thông tin đã xác nhận với thông tin chưa đủ dữ liệu. Đây là dấu hiệu cho thấy chất lượng câu trả lời được cải thiện.

- **Lỗi còn lại hoặc mới xuất hiện:**
  - Tổng số token của A6 tăng từ 7,310 lên 7,751, cho thấy phiên bản sau sử dụng nhiều token hơn.
  - Điểm tổng thể tăng từ 89.14 lên 91.59, nhưng không phải tất cả case đều cải thiện: có 11 case tăng điểm, 4 case giữ nguyên và 7 case giảm điểm.
  - Chưa thể kết luận thay đổi prompt là nguyên nhân duy nhất làm A6 tăng điểm, vì cả hai phiên bản đều có đủ hai URL trong câu trả lời cuối.

## 5. Giới hạn

- **Điểm Judge:** Điểm được đánh giá bởi một mô hình LLM nên có thể chịu ảnh hưởng bởi cách diễn giải tiêu chí, độ ổn định của mô hình và cách viết câu trả lời. Điểm số không hoàn toàn tương đương với đánh giá chất lượng thực tế của người dùng.
- **Snapshot:** Agent chỉ phân tích dữ liệu incident có trong snapshot tại thời điểm chạy. Nếu trạng thái hoặc nội dung incident thay đổi sau đó, kết quả có thể khác. Ngoài ra, một số nội dung trong incident bị cắt ngắn (`truncated=True`), nên agent không thể xác nhận các chi tiết không xuất hiện trong dữ liệu.
- **Số lần chạy:** Mỗi phiên bản chỉ được chạy một lần cho mỗi case. Kết quả có thể chịu ảnh hưởng bởi tính không xác định của LLM và sự biến động của API. Vì vậy, chưa đủ cơ sở để kết luận mức tăng điểm sẽ ổn định nếu chạy lại nhiều lần.

**Kết luận:** Sau khi điều chỉnh validation prompt, điểm trung bình của bộ 22 case tăng 2.45 điểm và riêng A6 tăng 19 điểm. Thay đổi hướng tới việc bảo toàn đầy đủ nguồn tham chiếu khi tổng hợp nhiều incident. Tuy nhiên, cần chạy lặp lại và kiểm tra thêm các case khác để đánh giá độ ổn định cũng như xác định rõ tác động của thay đổi này.