# CNTT1-K-N-ng-Promting-v-i-AI-Session04Dưới đây là bản báo cáo ngắn gọn, súc tích, bám sát các tiêu chí và cấu trúc của đề bài.

---

# BÁO CÁO SO SÁNH TRADE-OFF: KỸ THUẬT PROMPTING NÂNG CAO (LOGIC & CHUỖI)

## Bước 1: Đề xuất 2 Prompt mẫu

**1. Giải pháp 1: Prompt đơn giản, ngắn gọn**


Bạn là nhân viên bán hàng. Hãy kiểm tra đơn hàng sau của khách: "{{INPUT}}"
Kho hiện có: Áo thun (200k, tồn 5), Quần jean (400k, tồn 1), Mũ (100k, tồn 0).
Quy định: Đơn từ 500k miễn phí ship, dưới 500k phí ship 30k.

Hãy suy luận từng bước (Chain-of-Thought) để kiểm tra tồn kho, tính tổng tiền các món hợp lệ và xuất báo cáo gồm: Trạng thái đơn, Lỗi phát hiện và Tổng tiền cần thanh toán.



**2. Giải pháp 2: Prompt chi tiết, nhiều ràng buộc**


Vai trò: Nhân viên điều phối đơn hàng. Xử lý tin nhắn khách hàng theo 3 bước tuần tự sau: "{{INPUT}}"

DỮ LIỆU HỆ THỐNG:
- SP01 | Áo thun | 200.000đ | Tồn kho: 5
- SP02 | Quần jean | 400.000đ | Tồn kho: 1
- SP03 | Mũ lưỡi trai | 100.000đ | Tồn kho: 0 (Hết hàng)
- Vận chuyển: Đơn >= 500.000đ freeship; đơn < 500.000đ ship 30.000đ.

QUY TRÌNH XỬ LÝ (BẮT BUỘC):
- Bước 1 (Bẫy lỗi & Xử lý đứt gãy): Kiểm tra thiếu thông tin liên hệ (Tên, SĐT, Địa chỉ) hoặc đặt vượt tồn kho / hết hàng. Nếu có, ghi cảnh báo [THIẾU THÔNG TIN] hoặc [LỖI KHO]. Tuyệt đối không tự bịa dữ liệu.
- Bước 2 (Chain-of-Thought): Liệt kê từng bước tính tiền cho phần sản phẩm hợp lệ, cộng phí ship. Nếu gặp sản phẩm hết hàng, ngắt phần tính tiền món đó và chỉ báo giá tạm tính phần còn lại.
- Bước 3 (Định dạng đầu ra): Chỉ trả về đúng mẫu sau:
--- PHIẾU ĐƠN HÀNG ---
Trạng thái: [Hợp lệ / Cần bổ sung / Lỗi kho]
Cảnh báo: [Chi tiết lỗi hoặc "Không"]
Chi tiết tính toán: [Các bước tính]
Tổng thanh toán: [Số tiền] VNĐ
----------------------



## Bước 2: Lập bảng so sánh Trade-off

| Tiêu chí | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết) |
| --- | --- | --- |
| **Độ dễ hiểu** | Cao (Ngắn gọn, dễ viết và chỉnh sửa nhanh). | Trung bình (Cấu trúc dài, nhiều điều kiện rẽ nhánh). |
| **Tốn token** | Ít (Tiết kiệm chi phí xử lý, tốc độ phản hồi nhanh). | Nhiều hơn (Tốn nhiều token đầu vào do có các quy tắc ràng buộc). |
| **Chặn lỗi biên** | Trung bình (Dễ bỏ sót khi khách nhắn thiếu địa chỉ hoặc tự đoán giá). | Cực tốt (Bắt lỗi chính xác dữ liệu trống, lệch kho và chuẩn hóa đầu ra). |



## Bước 3: Lựa chọn phương án và Chạy thử

* **Phương án được chọn:** **Giải pháp 2 (Chi tiết, nhiều ràng buộc).**
* **Lý do lựa chọn:** Trong quy trình xử lý đơn hàng thực tế, việc để sót thông tin giao hàng hoặc báo sai tồn kho gây thiệt hại trực tiếp cho cửa hàng. Giải pháp 2 tuy tốn nhiều token hơn nhưng đảm bảo tính chính xác tuyệt đối, kiểm soát tốt các điểm đứt gãy dữ liệu và giữ định dạng kết quả đồng nhất.

**Kết quả chạy thử (với Giải pháp 2):**

* **Dữ liệu đầu vào (`INPUT`):** *"Shop gửi cho mình 2 quần jean (SP02) và 1 mũ lưỡi trai (SP03). Mình tên Minh, sđt 0909111222."*
* **Phản hồi thu được:**


--- PHIẾU ĐƠN HÀNG ---
Trạng thái: Cần bổ sung & Lỗi kho
Cảnh báo: 
- [THIẾU THÔNG TIN]: Khách hàng chưa cung cấp địa chỉ giao hàng.
- [LỖI KHO]: SP02 (Quần jean) khách đặt 2 nhưng tồn kho chỉ còn 1.
- [LỖI KHO]: SP03 (Mũ lưỡi trai) đã hết hàng (tồn kho 0).
Chi tiết tính toán:
- Tính tạm tính cho 1 quần jean (SP02) hợp lệ còn trong kho: 1 x 400.000đ = 400.000đ. (Ngắt tính tiền SP03 do hết hàng).
- Vì 400.000đ < 500.000đ nên áp dụng phí ship 30.000đ.
- Tổng cộng: 400.000đ + 30.000đ = 430.000đ.
Tổng thanh toán: 430.000 VNĐ (Tạm tính phần hàng hợp lệ)
