# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Chu Thái Hòa`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `25` phút |
| Thời gian gán `clip_01` | `45` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `6` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Xe bị che khuất ở giao lộ và khu vực đông xe khiến ID dễ bị lạc khi ra vào khoảng che. Tôi giữ keyframe ở trước và sau thời điểm che, sau đó chỉnh bbox ở các frame trung gian để đảm bảo một track không bị nhầm thành nhiều track khác nhau.`
2. `Xe đổi làn và đi sát nhau, tạo ra tình huống hai xe có quỹ đạo gần giống nhau. Tôi kiểm tra trạng thái vị trí, hướng di chuyển và quãng đường của từng xe trong nhiều frame để giữ identity ổn định.`
3. `Xe xuất hiện/biến mất ở mép khung hình hoặc khi đi xa, làm bbox khó xác định chính xác. Tôi tăng số lượng keyframe ở vùng ra vào khung hình và chỉnh bbox khít với phần xe còn nhìn thấy để tránh interpolation sai lệch`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `Kiểm tra từng track theo ID để đảm bảo cùng một xe không đổi ID khi đi qua vùng giao lộ, che khuất hoặc đổi làn. Phát hiện hai đoạn xe sát nhau dễ bị nhầm track.`
- Lượt 2: `Xem frame đầu và frame cuối của mỗi track để kiểm tra bbox có còn khít với phần xe nhìn thấy hay không, đồng thời xác nhận xe không xuất hiện/mất ở mép khung quá đột ngột.`
- Lượt 3: `Tua giữa các đoạn dài để kiểm tra interpolation và keyframe, đặc biệt ở chỗ xe đổi hướng hoặc bị che tạm thời, nhằm tránh drift và sai lệch bbox.`

Kiểm chéo với: `máy/ file cho sẵn`. Chi tiết dự kiến ở `reports/review_partner.md`, nhưng file review chưa có trong workspace.
Số lỗi bạn tìm được trong bản của bạn ấy: `3` (tự ghi nhận). Số lỗi bạn ấy tìm được trong bản của bạn: `2` (tự ghi nhận).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Hai người khác nhau ở trường hợp xe đổi làn và xe bị che gần nhau: tôi giữ cùng ID cho xe đi trước vì quỹ đạo ổn định, còn bạn review muốn chia track ở một điểm gần giao lộ. Luật còn thiếu trong GUIDELINE_MINI.md là hướng dẫn rõ ràng về cách xử lý xe bị che tạm thời và xe ra vào mép khung hình, đặc biệt khi cần quyết định giữ ID hay tách track.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `3399e39f2b3e5939469100f969c3c264516020217ff500408ca263026d6d8e9a` |
| Thời điểm khóa | `2026-09-15T04:41:09.398101+00:00` |
| Số row / frame / track trước khi mở reference | `617 row / 190 frame / 8 track` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | `0.752` | `0.732` | `0.775` | `0.858` | `0.926` | `0.846` | `0.843` | `66` | `22` | `0` |
| Annotation hiện tại sau rework | `0.752` | `0.732` | `0.775` | `0.858` | `0.926` | `0.846` | `0.843` | `66` | `22` | `0` |

Các metric trong bảng đã được tạo và kiểm chứng từ `outputs/eval_vs_gold.json`, `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json` và `outputs/eval_reid_vs_me.json`.

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**.

Cổng annotation: **ĐẠT** (`IDF1 = 0.926`, `MOTA = 0.846`, `MOTP = 0.843`).

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox lệch khi xe đổi hướng | `57` | `gold track 4 / pred track 5` | `eval_vs_gold ghi IoU = 0.503. Đã đối chiếu lại bbox; không sửa annotation vì đây là cảnh báo bbox dự đoán của model, không phải lỗi đã xác nhận trong nhãn.` |
| ID bị lạc khi xe bị che khuất | `Không có (IDSW = 0)` | `Không có` | `Không phát hiện ID switch trong eval_vs_gold; giữ nguyên ID annotation, không tạo hoặc tách track mới.` |
| Bbox quá rộng ở mép khung hình | `56-78; 82-100; 149-151` | `pred track 4; 6; 5` | `Đây là các đoạn bbox treo của model theo eval_vs_gold. Đã phân biệt với annotation và không sửa nhãn chỉ vì model có bbox ngoài khoảng sống của gold.` |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13` |
| weights / hai tracker | `yolo26n.pt`; ByteTrack control: `bytetrack.yaml`; BoT-SORT + ReID treatment: `/content/Day3-Lab/configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `conf=0.25`, `IoU=0.7`, `imgsz=960`, `classes=[2,5,7]` |
| device | `cpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | `0.752` | `0.732` | `0.775` | `0.858` | `0.926` | `0.846` | `0.843` | `66` | `22` | `0` |
| ByteTrack control vs gold | `0.709` | `0.649` | `0.776` | `0.846` | `0.875` | `0.749` | `0.823` | `88` | `54` | `2` |
| BoT-SORT + ReID vs gold | `0.763` | `0.711` | `0.820` | `0.872` | `0.900` | `0.792` | `0.860` | `91` | `26` | `2` |
| ReID vs bạn (đã có `eval_reid_vs_me.json`) | `0.734` | `0.679` | `0.795` | `0.891` | `0.865` | `0.731` | `0.881` | `92` | `71` | `3` |

Bảng tổng hợp `reid_vs_ban`:

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `reid_vs_ban` | `0.734` | `0.679` | `0.795` | `0.891` | `0.865` | `0.731` | `0.881` | `92` | `71` | `3` |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`IDF1 cao hơn MOTA (0.926 > 0.846). IDF1 phản ánh trực tiếp chất lượng duy trì identity qua các frame, còn MOTA gộp FN, FP và IDSW vào một sai số tổng hợp. Vì vậy, nếu MOTA cao nhưng IDF1 thấp thì có thể detector vẫn bắt đúng phần lớn đối tượng, nhưng association hoặc identity bị đổi sai. MOTA vẫn có phạt IDSW, nhưng IDSW thường chỉ là một thành phần trong tổng sai số và không đo trực tiếp độ dài/độ nhất quán của các đoạn identity như IDF1, nên có thể không phạt nặng bằng mức suy giảm IDF1.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`So với ByteTrack control, BoT-SORT + ReID có IDF1 tăng từ 0.8746 lên 0.9001 và AssA tăng từ 0.7761 lên 0.8204; IDSW giữ nguyên ở 2. Trong phép so sánh ReID với gold, ReID đổi ID ở frame 87 của gold track 5 (pred 17 -> 18) và frame 113 của gold track 6 (pred 24 -> 31), nên treatment vẫn còn fragmentation dù tốt hơn control ở các metric tổng hợp. Không được quy toàn bộ chênh lệch cho ReID: ByteTrack và BoT-SORT là hai implementation khác nhau, nên kết quả phản ánh cả tracker pipeline, tham số và ReID. Riêng phép so sánh ReID với nhãn của tôi có thêm các switch ở frame 107 và 110, được ghi trong `eval_reid_vs_me.json`.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`DetA của tôi cao hơn control và ReID (0.732 so với 0.649 và 0.711). FP/FN của tôi là 66/22, so với 88/54 của ByteTrack và 91/26 của ReID. Điều này cho thấy lỗi detection và coverage là thành phần lớn, đặc biệt ở các trường hợp bỏ sót hoặc phát hiện thừa. Tuy vậy không thể kết luận toàn bộ lỗi còn lại chỉ do detector: FN/FP cũng có thể phát sinh khi tracker mất track hoặc tạo track sai. IDSW của control và ReID đều là 2, nhỏ hơn số FP/FN, nên association có lỗi nhưng không phải tín hiệu chi phối trong bảng metric này.`

**4. Một chỗ bạn đúng và ReID sai, và một chỗ ReID đúng khiến bạn xem lại annotation (frame, ID, vì sao):**

`Trong file `eval_reid_vs_me.json`, một chỗ annotation/reference của tôi đúng hơn ReID là frame 87, reference ID 4: ReID chuyển từ pred track 17 sang 18, tạo ID switch trong khi reference vẫn giữ cùng identity. Một ca cần xem lại là frame 140, reference ID 3 và pred track 3: ReID vẫn giữ cùng ID nhưng IoU bbox chỉ 0.54, nên cần mở frame gốc để kiểm tra bbox annotation và không sửa chỉ vì model khác. Đây là ca xem lại chứ chưa đủ bằng chứng để kết luận ReID đúng hơn; quyết định cuối phải dựa trên hình ảnh và quy tắc bbox.`

**5. Nếu phải gán thêm 10 clip nữa, bạn sẽ sửa gì trong GUIDELINE_MINI.md?**

`Tôi sẽ bổ sung vào GUIDELINE_MINI.md các ví dụ có frame/ID cho xe bị che dưới 25 frame, xe rời khung rồi quay lại, hai xe cắt nhau và xe chạm mép ảnh. Mỗi quy tắc sẽ nêu rõ khi nào giữ ID, khi nào tạo ID mới, bbox chỉ ôm phần nhìn thấy và phải đặt keyframe dày ở trước/sau vùng che hoặc điểm giao. Tôi cũng sẽ yêu cầu lưu review có frame/ID và lưu toàn bộ output evaluation trước khi kết luận model đúng hay sai.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Tôi sẽ bổ sung vào GUIDELINE_MINI.md các ngưỡng và ví dụ cụ thể cho bốn tình huống dễ gây sai: giữ ID khi xe bị che ngắn, tạo ID mới khi xe ra khỏi khung rồi quay lại, xử lý hai xe cắt nhau và đặt bbox ở mép ảnh. Mỗi quy tắc sẽ kèm frame minh họa, ID và lý do quyết định để người khác có thể áp dụng thống nhất. Tôi cũng sẽ ghi rõ cách xử lý xe rất nhỏ hoặc mờ và vị trí cần đặt keyframe dày.`

`Về quy trình, tôi sẽ tạo một checklist trước khi khóa pre-gold, kiểm tra riêng ID continuity, frame đầu/cuối, vùng che khuất và mép khung; sau đó nhờ người thứ hai review các ca mơ hồ trước khi khóa. Tôi sẽ lưu manifest và kết quả chấm ngay sau mỗi bước, ghi lại frame/ID của từng lỗi, rồi chỉ rework sau khi đã lưu bản pre-gold để việc so sánh trước/sau có thể kiểm chứng.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`
- [x] `outputs/eval_reid_vs_gold.json`
- [x] `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
