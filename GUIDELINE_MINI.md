# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Do Thanh Long — 2A202602199`
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

Bổ sung của nhóm (nếu có): Không có.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps) |
| Xe bị che lâu hơn ngưỡng trên | bấm **Outside** tại frame cuối còn thấy; xuất hiện lại → **track ID mới** |
| Xe rời khung hình rồi quay lại | **track mới** |
| Hai xe cắt nhau / chồng lên nhau | giữ ID theo hướng chuyển động liên tục, không hoán đổi ID cho nhau |

**Các lần bbox chồng lấn trong dữ liệu (kiểm lại trong CVAT):**

| Clip | Frame | Track ID | IoU |
| --- | ---: | --- | ---: |
| clip_01 | 119 | 3 & 5 | 0.06 |
| clip_01 | 134 | 3 & 6 | 0.08 |
| clip_01 | 186 | 3 & 7 | 0.08 |
| clip_02 | 22 | 2 & 4 | 0.05 |
| clip_02 | 35 | 3 & 4 | 0.06 |
| clip_02 | 39 | 2 & 6 | 0.08 |
| clip_02 | 51 | 3 & 6 | 0.05 |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | ngưỡng: **~1600 px²** (mốc nhỏ nhất quan sát được: track 6/clip_01, frame 110, 89×18) |
| Xe đang đỗ, không di chuyển | vẫn tạo track bình thường (ca thật: track 3/clip_01, đứng yên tại (208,252) suốt 190 frame) |
| Keyframe đặt dày ở đâu | dày quanh entry/exit và lúc cắt nhau; thưa ở đoạn xe đi thẳng đều |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01, frame 1–190, ID 3`
- Tình huống: xe đứng yên tại (208,252) suốt cả clip
- Quyết định: vẫn giữ một track `vehicle` xuyên suốt, không xoá/bỏ qua vì xe không di chuyển
- Lý do: Xe đang đỗ vẫn là vehicle và vẫn cần track suốt thời gian nó trong khung

### Ca 2
- Clip / frame / ID: `clip_02, frame 60, ID 5`
- Tình huống: bbox teo còn 3×4 px, chạm rìa phải/dưới đúng lúc clip kết thúc
- Quyết định: giữ bbox ôm sát phần còn nhìn thấy đến hết frame, không cần bấm `outside` vì đây là frame cuối clip chứ không phải xe rời khung giữa chừng
- Lý do: outside chỉ bấm ở frame xe rời khung; ở đây bbox vẫn hợp lệ (chạm rìa, không đoán ngoài ảnh) theo luật mục 3

### Ca 3
- Clip / frame / ID: `clip_01, frame 134, ID 3 & 6`
- Tình huống: bbox hai xe chồng lấn nhẹ (IoU 0.08) khi đi ngang qua nhau
- Quyết định: giữ nguyên ID 3 và ID 6, không hoán đổi cho nhau
- Lý do: hai xe cắt nhau không đổi ID cho nhau; quỹ đạo mỗi track liên tục trước/sau frame 134 nên không có căn cứ để gộp/đổi ID

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- "Xe rời khung hình rồi quay lại" cần thêm mốc chính xác: theo `outputs/eval_vs_gold.json`, ID 4 (frame 149–151) và ID 8 (frame 169–171) còn giữ bbox 3 frame sau khi gold đã coi xe rời khung — luật mới: bấm `outside` ngay khi bbox chạm rìa lần cuối, không đợi thêm frame để "chắc chắn".
- Ngưỡng bắt đầu track cho xe nhỏ/mờ (~1600 px²) vẫn hơi trễ so với gold: track gold 5 chỉ được phủ 78% (47/60 frame) và track gold 8 chỉ 76% (25/33 frame) — luật mới: hạ ngưỡng bắt đầu track xuống ngay khi nhận ra hình dạng xe, kể cả khi bbox còn dưới 1600 px².
