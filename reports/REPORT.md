# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Do Thanh Long — 2A202602199` (làm cá nhân)
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `15` phút |
| Thời gian gán `clip_01` | `45` phút |
| Số track đã vẽ trong `clip_01` | `20` |
| Số keyframe trung bình mỗi track | `10` (không suy ra được từ `gt.txt` đã export vì file là per-frame sau interpolation, không phải danh sách keyframe gốc trong CVAT) |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `clip_01`, ID 3, frame 1–190: xe đỗ đứng yên suốt clip — vẫn giữ một track `vehicle` xuyên suốt, không xoá/bỏ qua.
2. `clip_02`, ID 5, frame 60: bbox teo còn 3×4 px đúng lúc clip kết thúc — giữ bbox ôm phần còn nhìn thấy, không bấm `outside` vì đây là frame cuối clip chứ không phải xe rời khung giữa chừng.
3. `clip_01`, ID 3 & 6, frame 134: bbox hai xe chồng lấn nhẹ (IoU 0.08) khi đi ngang qua nhau — giữ nguyên từng ID, không hoán đổi cho nhau.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `0 ID switch, 8/8 track khớp gold, không có ID nào bị trùng/tách.`
- Lượt 2: `ID 4 (frame 149–151) và ID 8 (frame 169–171) còn bbox treo sau khi gold coi xe đã rời khung.`
- Lượt 3: `cảnh báo track 3 gần như đứng im frame 1–15 (đứng yên thật hay quên bấm outside)`

Kiểm chéo với: Không có — làm cá nhân, chưa có reviewer. `reports/review_partner.md` chưa tồn tại.
Số lỗi bạn tìm được trong bản của bạn ấy: `N/A`. Số lỗi bạn ấy tìm được trong bản của bạn: `N/A`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`N/A — chưa có kiểm chéo`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `54d6a9272c6923b7e178721411cbe1bef8966a5d17548c83270cace600ec25cf` |
| Thời điểm khóa | `2026-09-15T04:54:54Z` |
| Số row / frame / track trước khi mở reference | 531 row / 190 frame / 8 track |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.796 | 0.774 | 0.821 | 0.892 | 0.938 | 0.881 | 0.881 | 13 | 55 | 0 |
| Sau rework | | | | | | | | | | |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Danh sách lỗi từ `outputs/eval_vs_gold.json` (gate đã đạt, ghi lại để cân nhắc rework tuỳ chọn):

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo (còn box sau khi xe rời khung) | 149–151 | 4 | `remove tại sau 5-10 frame xe rời khung do số frame ít` |
| Bbox treo (còn box sau khi xe rời khung) | 169–171 | 8 | `remove tại sau 5-10 frame xe rời khung do số frame ít` |
| Thiếu đoạn (track gold 8 chỉ phủ 76%) | — | 8 | `...` |
| Thiếu đoạn (track gold 5 chỉ phủ 78%) | — | 5 | `...` |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.12.7 / 8.4.145 / 2.14.0 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` (control), `configs/trackers/botsort-reid.yaml` (treatment) |
| conf / IoU / imgsz / classes | 0.25 / 0.7 / 960 / [2, 5, 7] |
| device | cpu |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.796 | 0.774 | 0.821 | 0.892 | 0.938 | 0.881 | 0.881 | 13 | 55 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.859 | 91 | 26 | 2 |
| ReID vs bạn | 0.756 | 0.701 | 0.817 | 0.885 | 0.891 | 0.761 | 0.873 | 117 | 10 | 0 |

Cổng qua bài của hai run model (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): ByteTrack control **KHÔNG đạt** (MOTA 0.749 < 0.75); BoT-SORT + ReID **đạt** cả ba.

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Ở cả 4 hàng trong bảng trên, IDF1 luôn cao hơn MOTA (0.938 vs 0.881; 0.875 vs 0.749; 0.900 vs 0.792; 0.891 vs 0.761) — không có hàng nào ngược lại. MOTA = 1 − (FP+FN+IDSW)/GT_boxes nên bị FP/FN chi phối gần như hoàn toàn (IDSW chỉ 0–2 trên 531–638 box, đóng góp dưới 0.4% vào công thức); IDF1 dùng khớp theo track dài nhất nên vẫn cao dù một số frame lẻ bị FP/FN, miễn ID không đổi. Đây là lý do MOTA không phạt nặng lỗi ID — IDSW chỉ là một số hạng nhỏ trong tử số so với FP/FN.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

IDF1: 0.875 (control) → 0.900 (treatment), AssA: 0.776 → 0.820, IDSW: 2 cả hai (khác vị trí switch). Không cô lập được causal effect của ReID vì BoT-SORT và ByteTrack là hai tracker implementation khác nhau, không chỉ khác mỗi ReID.

- Tốt hơn: track gold 4 (frame 70–151) bị `fragmented` thành 2 ID dưới control (pred 14→15 tại frame 59) nhưng liền mạch dưới treatment (không còn trong danh sách fragmented).
- Không đổi đáng kể: track gold 7 (frame 113–190) vẫn `fragmented` ở cả hai (control: pred 52→71 tại một frame lẻ; treatment: pred 29→39), tỉ lệ 1 frame lẻ trên >78 frame ở cả hai run.
- Tệ hơn: track gold 6 không có trong danh sách fragmented của control nhưng bị fragment dưới treatment tại frame 113 (pred 24→31).

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA: 0.649 (control) → 0.711 (treatment). FP: 88 → 91 (gần như không đổi). FN: 54 → 26 (giảm gần một nửa). Cả hai run đều có đúng 5 `ghost_pred_tracks` không khớp track tham chiếu nào (ví dụ frame 106–121, frame 163/167–178) — cùng vị trí ở cả hai run, cho thấy đây là lỗi detector (model detect vật không phải `vehicle` hoặc detect trùng vùng nền) chứ không phải do đổi tracker. Phần fragmented track (track 4, 5, 6, 7 bị tách ID giữa chừng ở các frame lẻ) là lỗi association.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`clip_01, frame 55–69, pred track 9 (theo eval_reid_vs_me.json)`: ReID mở một track ở đây trước khi track tham chiếu của bạn (ID 4, bắt đầu frame 70) xuất hiện. Bạn bắt đầu track 4 đúng frame 70 theo luật "bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh" (ngưỡng ~1600 px² trong `GUIDELINE_MINI.md`), còn model detect sớm hơn trên một vật thể chưa đủ rõ.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`clip_01, frame 138–140 và 190, ID 3 (loose_boxes trong eval_reid_vs_gold.json, IoU 0.50–0.54 với gold)`: đây đúng track xe đỗ đứng yên (Ca 1 trong `GUIDELINE_MINI.md`). IoU với gold thấp hơn hẳn so với các frame khác của cùng track — đáng xem lại xem bbox của bạn có bị lệch/không cập nhật kích thước theo gold ở đúng các frame này không.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Làm rõ ngưỡng IoU tính là "hai xe cắt nhau" (dữ liệu hiện tại chỉ có các lần chồng nhẹ IoU 0.05–0.08, chưa có ca chồng nặng để chốt luật); ghi số phút thực tế cho mỗi lượt tự kiểm; và thu xếp có reviewer thật trước khi khoá pre-gold thay vì để trống `reports/review_partner.md`.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
