# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Chu Thái Hòa`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `Chỉ gán xe bốn bánh khi có đủ dấu hiệu nhận dạng; không suy đoán xe bị che hoàn toàn hoặc chỉ còn vùng quá nhỏ, không xác định được.`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps) | Quỹ đạo trước và sau vùng che vẫn liên tục, giúp tránh tạo ID mới cho cùng một xe. |
| Xe bị che lâu hơn ngưỡng trên | tạo track mới sau khi xe xuất hiện lại, trừ khi có bằng chứng chắc chắn về vị trí và hình dạng liên tục | Sau thời gian che dài, không đủ bằng chứng để khẳng định identity cũ. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Không giả định xe quay lại là cùng identity nếu đã mất khỏi khung hình. |
| Hai xe cắt nhau / chồng lên nhau | giữ ID theo quỹ đạo, hướng di chuyển và vị trí trước khi cắt; không đổi ID chỉ vì bbox chồng nhau | Motion continuity đáng tin cậy hơn vị trí tức thời trong một frame. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **bbox khoảng từ 10 x 10 px trở lên và còn nhận ra được hình dạng xe** |
| Xe đang đỗ, không di chuyển | vẫn giữ cùng ID và bbox nếu xe còn nhìn thấy; chỉ kết thúc track khi xe rời khung hoặc bị che hoàn toàn quá ngưỡng | 
| Keyframe đặt dày ở đâu | đặt dày trước/sau khi xe đổi hướng, bị che, cắt nhau, đi sát mép ảnh, xuất hiện hoặc biến mất; đoạn chuyển động thẳng và ổn định có thể đặt thưa hơn |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 7 / ID 1`
- Tình huống: `Xe đi sát mép trái và bbox chạm rìa ảnh.`
- Quyết định: `Giữ ID 1, đặt x = 0 và chỉ bao quanh phần xe còn nhìn thấy.`
- Lý do: `Không suy đoán phần xe nằm ngoài khung; bbox phải phản ánh đúng vùng quan sát được.`

### Ca 2
- Clip / frame / ID: `clip_01 / frame 1 / ID 2`
- Tình huống: `Xe xuất hiện sát mép phải, dễ bị cắt hoặc bị nhầm là track mới.`
- Quyết định: `Giữ ID 2 và theo dõi tiếp theo quỹ đạo sang các frame sau.`
- Lý do: `Xe đã xác định được là xe bốn bánh và có chuyển động liên tục, nên không tách ID chỉ vì vị trí gần mép ảnh.`

### Ca 3
- Clip / frame / ID: `clip_02 / frame 1 / ID 2 và ID 3`
- Tình huống: `Hai xe bị cắt ở phía trên khung hình, bbox có y = 0 và phần thân xe nhìn thấy không đầy đủ.`
- Quyết định: `Giữ nguyên từng ID, bbox chạm đúng rìa trên và không mở rộng ra ngoài ảnh.`
- Lý do: `Mỗi xe vẫn có vị trí và hình dạng đủ để phân biệt; phần bị khuất ngoài ảnh không được suy đoán.`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Khi xe bị che ngắn hơn 25 frame, giữ ID nếu quỹ đạo trước và sau vùng che liên tục; nếu che lâu hơn hoặc rời khung rồi quay lại thì mặc định tạo ID mới. Kết quả eval_vs_gold có IDSW = 0 và không có track bị tách, nên không được đổi ID chỉ vì model hoặc bbox ở một frame khác biệt.`
- `Ở vùng xe cắt nhau, phải kiểm tra motion continuity và đặt keyframe dày trước/sau điểm giao; bbox chỉ bao quanh phần nhìn thấy và chạm rìa ảnh khi xe bị cắt.`
- `Phải kiểm tra frame bắt đầu/kết thúc của từng track bằng hình ảnh gốc. Các khoảng bị đánh dấu cần rà lại gồm frame 56-78, 82-100 và 149-151; nếu xe chưa xuất hiện hoặc đã rời khung thì bấm outside đúng frame, không để bbox treo.`
- `Khi IoU bbox thấp như frame 57 (track tham chiếu 4, pred ID 5, IoU = 0.503), phải xem lại frame gốc và keyframe lân cận trước khi sửa. Không sửa annotation chỉ vì tracker ReID khác ID; chỉ sửa khi hình ảnh và luật bbox xác nhận lỗi.`
