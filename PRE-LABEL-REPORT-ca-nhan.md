# Báo cáo thực hành PointPillars — Day 13 (bản cá nhân)

> Bản đã điền, thu riêng cho LC. Không commit lên repo public.
> Trạng thái: **`executed-by-self`** — tự tải gói Student, tự chạy đủ A/B/C và tạo ca QC trên máy cá nhân.
>
> Bài này được giao **cá nhân** (không theo nhóm 3–4 người như bản gốc của lab), nên không có `TEAMMATES.md` và không chia vai vận hành.

## 1. Nhóm và provenance

- **Họ tên:** Thân Vĩnh Trọng
- **MSSV:** 2A202602262
- **Mã ca:** 2B-Lab-D305
- Hình thức: **cá nhân** (bài giao đã đổi từ nhóm 3–4 người sang cá nhân)
- Trạng thái thực thi: **`executed-by-self`** (không phải `provided-results`; không chạy trên máy LC)
- Người thực sự chạy: **Thân Vĩnh Trọng** — tự tải gói, tự chạy lệnh, tự đọc kết quả
- Ngày/giờ chạy: 2026-10-02, ~03:23–03:25 UTC (10:23–10:25 giờ VN)
- Hệ máy / architecture: **MacBook Pro Apple Silicon (M1) — arm64**, macOS, Docker Desktop `linux/aarch64`, server 29.7.2
- Mạng: VPN 1.1.1.1 (đường quốc tế bị nghẽn nếu không VPN; không ảnh hưởng kết quả tính toán)
- Image tag: `day13-pointpillars:lc-20261001-arm64`
- Image ID: `sha256:dd6999ad5dd67962fdca18eb526132c08ce105980c9d89475d66cb1a5193f8a1`
- Phiên bản repo trong gói: `0831856d921609312d42c7582c366e5a311bb7b1` (`working_tree_dirty: true`, đúng như manifest ghi)
- PCD được cấp: `input/demo.pcd` (KITTI 000008 đã chuyển đổi), `frame_id=demo`
  - SHA256: `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60` (khớp `VALIDATION.md`)
  - 17.238 điểm; `z_ground` ước lượng = **0,075 m**
- Checkpoint: `/opt/PointPillars/pretrained/epoch_160.pth`
  - SHA256: `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`
- Phạm vi: **front-window**; score threshold: **0,3**; giới hạn container 4 CPU / 4 GB
- Giả định kênh thứ tư/intensity: PCD bài chủ đích **bỏ reflectance gốc**, `rgb=0` là placeholder → script dùng kênh hằng số. Đây **không phải** benchmark KITTI dùng intensity thật.
- Nguồn `z_ground`: ước lượng từ chính PCD (không phải mặt đường đo chính xác). Đường `z=0` trên ảnh Side chỉ là đường tham chiếu của plot.

Gói tải về: `student-prelabel-arm64.zip`, SHA256 `f52d6c55eb8cb2570e79160862ae4ed4c6d82a3c53f90d5edf298f55fb984be8` — khớp `SHA256SUMS.txt` chính thức.

## 2. Kết quả chạy (đọc từ output, không tự tính)

`smoke.json` → **`status: passed`**, đủ 5 bước: `docker-load` → `run-A` → `run-B` → `run-C` → `qc-cases`.

| Bước | Thời gian (giây) | Số hộp | prediction SHA256 |
|---|---:|---:|---|
| docker-load | 50,97 | — | — |
| run-A | 13,92 | 1 | `f5b84cb9…5272` |
| run-B | 8,50 | 13 | `51f49ac6…016d` |
| run-C | 5,54 | 6 | `777c4706…3be2` |
| qc-cases | 1,54 | 3 ca | — |

### Ba lượt inference thật

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | ---: | ---: | --- | --- |
| A | 0 | 0,16 | **1** | **0,330** | `run-A/boxes-demo-delta-0-voxel-0.16.json` · `side-demo-delta-0-voxel-0.16.png` · `summary.csv` | Ảnh Side: một hộp đỏ ở x≈13 m, **chìm xuống dưới đường z=0** (đáy ≈ −0,4 m, nắp ≈ +1,05 m). Điểm nằm sát z≈0. |
| B | 1,73 | 0,16 | **13** | **1,034** | `run-B/boxes-demo-delta-1.73-voxel-0.16.json` · `side-demo-delta-1.73-voxel-0.16.png` · `summary.csv` | Ảnh Side: 13 hộp, đáy gần đúng z=0. Phân bố: 10 `vehicles`, 2 `pedestrian`, 1 `two-wheels`. Hộp xa nhất x≈55,6 m. |
| C | 1,73 | 0,32 | **6** | **1,091** | `run-C/boxes-demo-delta-1.73-voxel-0.32.json` · `side-demo-delta-1.73-voxel-0.32.png` · `summary.csv` | Ảnh Side: 6 hộp, **toàn bộ là `pedestrian`**, mảnh và cao (h≈1,7 m, w≈0,6–1,06 m). Không còn hộp `vehicles` nào. |

### A/B — chỉ đổi `delta` (0 → 1,73 m)

A có **1** hộp; B có **13** hộp. `mean_z` 0,330 → 1,034.

Quan sát có bằng chứng:

- Về **hình học hộp duy nhất** ở A và hộp `[1]` của B: A có tâm (13,15; −0,45; 0,330), yaw 2,671; B có hộp `[1]` tâm (14,77; −1,08; 0,900), yaw −0,303. Hai hộp này **không cùng đối tượng** — yaw lệch gần 180° (2,671 vs −0,303, chênh ≈ 2,97 rad ≈ 170°), và vị trí cũng khác. Không thể coi là “cùng một hộp được dịch”.
- Ảnh `side-demo-delta-0-voxel-0.16.png`: hộp A **nằm dưới mặt đường** (đáy dưới z=0), trong khi cụm điểm thật nằm phía trên z=0. Ảnh B: các hộp **bám đáy z≈0** — đúng mặt đường.
- Đây là **model chạy lại trên input đã dịch**, không phải dịch hộp cũ: số hộp đổi từ 1 → 13, class xuất hiện đủ 3 loại, tọa độ và yaw của các hộp không bảo toàn.
- **Điều em còn chưa chắc:** không có ground truth nên không kết luận được B “đúng hơn” A chỉ vì hộp bám đất và nhiều hộp hơn. Về giả định, `delta=1,73` là cao sensor của checkpoint KITTI; hộp A chìm dưới z=0 là dấu hiệu giả định sai cho PCD này, nhưng em chưa có ảnh camera để đối chiếu đầu xe/hướng.

### B/C — chỉ đổi pillar XY (0,16 → 0,32 m)

B có **13** hộp; C có **6** hộp. `mean_z` 1,034 → 1,091.

Quan sát có bằng chứng:

- **Đổi hoàn toàn phân bố class**: B = 10 `vehicles` + 2 `pedestrian` + 1 `two-wheels`; C = **6 `pedestrian`**, không còn `vehicles` hay `two-wheels`.
- **Vị trí không bảo toàn**: hộp B `[8]` `vehicles` tại (9,38; 4,23; 1,426) không có hộp C tương ứng; hộp C `[0]` tại (19,43; −8,07; 0,841) là vùng B không có hộp nào. Chỉ có vài vị trí *gần giống*, ví dụ C `[1]` (13,15; 4,20) vs B `[8]` (9,38; 4,23) — khác 3,8 m về x.
- **Kích thước đổi theo hướng "mảnh và cao"**: C đều có width 0,55–1,06 m, height 1,68–1,79 m; B có width 1,51–1,72 m cho `vehicles` (bề ngang xe ≈ 1,6 m là hợp lý). C làm mất đặc trưng bề ngang của xe.
- Ảnh Side C: các hộp thon, kéo dài theo z, phủ lên vùng điểm không có cụm xe rõ.
- **Có đủ bằng chứng để kết luận tốt hơn không?** **Không.** C có **ít hộp hơn** (6 vs 13), mất toàn bộ nhóm `vehicles` mà ảnh Side cho thấy có cụm điểm hình xe ở x≈8–20 m, và kích thước bề ngang không khớp thực tế. C dùng lại checkpoint pretrained cho pillar 0,16 m; đây là **thay biểu diễn đầu vào**, không phải model được train riêng cho pillar 0,32 m. B là mốc so sánh hợp lý hơn cho bài này, nhưng “hợp lý hơn” ≠ “đã chứng minh đúng”.

### Giới hạn ROI và góc Side ảnh

- Side là hình chiếu toàn scene x–z, **nhiều đối tượng ở khoảng cách khác nhau chồng lên nhau**. Ví dụ vùng x≈8–10 m có 3–4 hộp B chồng nhau, không thể kết luận hộp nào ở đâu.
- Vùng **x > 30 m** điểm rất thưa, hộp B `[6]` (40,98 m) và `[9]` (55,58 m) nằm ở vùng gần như không có điểm → có thể là nhiễu/false positive, cần Top/Side/Front + camera mới kết luận.
- `mean_z` là trung bình cao độ **tâm hộp**, không phải điểm chất lượng; không dùng để so sánh A với B về độ đúng.
- Hộp A chỉ có 1 nên `mean_z` của A là của chính hộp đó — so trực tiếp mean_z A vs B/C về mặt thống kê là không cùng ý nghĩa.

### JSON nào còn chưa đủ cơ sở để import?

**Cả ba JSON A/B/C đều chưa đủ cơ sở import vào CVAT Robotaxi**, vì:

1. **Không cùng frame**: đây là KITTI `000008` demo, khác 30 job Robotaxi.
2. **Nhãn không đúng schema ca**: KITTI adapter chỉ sinh `vehicles` / `two-wheels` / `pedestrian`. Schema ca Day13 có **5 class** (`vehicles`, `two-wheels`, `pedestrian`, `Animal`, `Obstacle`) — thiếu 2 class.
3. **Domain/checkpoint khác**: pretrained KITTI, không phải model Robotaxi.

Riêng `case-correct.json` chỉ giữ nguyên phép chuyển z của prediction B — dùng để học cách đọc QC, không phải cuboid đúng.

## 3. Ca QC có kiểm soát — không import CVAT

Nguồn: `run-B/boxes-demo-delta-1.73-voxel-0.16.json` (SHA256 `51f49ac6…016d`), 13 hộp, `z_ground=0,075`, `delta=1,73` → lượng lệch chuẩn = **1,73 + 0,075 = 1,805 m**.

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Quyết định | Bằng chứng |
| --- | --- | ---: | --- | --- | --- |
| `case-correct` | **0 / 13** | 0 | Không đổi | **Không có lỗi** — giữ nguyên | `qc-cases/case-correct.json`; `side-correct.png`. Hộp [0] z=0,921 khớp B. Chỉ chứng minh helper giữ đúng phép chuyển nguồn, **không** chứng minh cuboid đúng. |
| `case-batch-z` | **13 / 13** | **−1,8050 m** (tất cả) | Class/kích thước/score giữ nguyên; Δx=Δy=Δyaw=0,0000 | **DỪNG BATCH — lỗi pipeline** | `side-batch-z.png`: toàn bộ hộp rơi **dưới đường z=0** (z từ +0,698…+1,426 xuống −0,527…−1,107). Cùng một lượng trừ ở cả 13 hộp = quên cộng ngược `z_ground+delta`. → Báo LC kiểm transform, yêu cầu tạo lại prediction; **không sửa tay từng hộp**. |
| `case-one-box-z` | **1 / 13** (hộp [0]) | −1,8050 m (chỉ hộp 0) | Hộp 0 lệch; 12 hộp sau **không đổi** (Δz=0) | **KIỂM TỪNG HỘP** — lỗi đối tượng | `side-one-box-z.png`: chỉ hộp [0] (x=8,09 m) chìm xuống z=−0,884, còn 12 hộp vẫn bám z≈0. 1 hộp lệch **không** đủ cơ sở kết luận lỗi pipeline; cần xem Top/Side/Front + camera cho đối tượng đó. |

Ghi rõ: helper `pipeline-qc-cases.py` tạo biến đổi có chủ đích từ prediction thật của lượt B (đọc `source_prediction_sha256` trong `qc-cases/manifest.json`). Đây **không phải** kết quả inference riêng, **không phải** nhãn đúng, và **không** import vào CVAT.

## 4. Nhận xét cá nhân — Thân Vĩnh Trọng (2A202602262) · ca 2B-Lab-D305

Vì bài giao **cá nhân**, mục này gộp vai trò + nhận xét + điều chưa chắc của một người thay vì chia theo thành viên nhóm.

- **Vai trò đã làm:** tự tải và xác minh gói Student arm64 (SHA256 khớp `SHA256SUMS.txt`), chạy `student-bundle.py run`, đọc `summary.csv` / `boxes-*.json` / `side-*.png`, và đo lệch z của cả 13 hộp trong từng ca QC bằng cách so trực tiếp với JSON của lượt B.
- **Quan sát A/B/C có dẫn file:** A có 1 hộp chìm dưới z=0 (`run-A/side-demo-delta-0-voxel-0.16.png`); B có 13 hộp bám đáy, đủ 3 class (`run-B/side-…-0.16.png`); C chỉ còn 6 hộp `pedestrian` mảnh cao, mất hết `vehicles` (`run-C/side-…-0.32.png`).
- **Diễn giải phép z thuận/ngược:** `z_model = z_source − z_ground − delta` và `z_source = z_model + z_ground + delta`. Với bộ dữ liệu này `z_ground=0,075`, `delta=1,73` nên lượng phải cộng ngược là **1,805 m**. `delta` là lượng dịch **trước** khi đưa điểm vào model; nó **khác** việc dịch một lượng cố định cho mọi hộp **sau** inference. Bằng chứng: `case-one-box-z` chỉ lệch đúng 1 hộp, còn `case-batch-z` lệch đồng loạt 13 hộp — một ca là lỗi đối tượng, ca kia là lỗi chuyển hệ tọa độ.
- **Một quan sát về pillar:** pillar XY không phải kích thước hộp. Khi tăng 0,16 → 0,32 m, ô gom điểm rộng gấp đôi nên nhiều điểm rơi chung một cột, thông tin bị trộn; model dùng checkpoint pretrained cho 0,16 m nên hỏng class: 10 `vehicles` + 2 `pedestrian` + 1 `two-wheels` (B) → 6 `pedestrian` (C). Kích thước bề ngang giảm từ ≈1,6 m (xe) xuống 0,55–1,06 m.
- **Một quyết định lỗi batch:** khi thấy **cùng một lượng lệch ở toàn bộ batch** (case `batch-z`: 13/13 hộp, Δz = −1,8050 m, Δx/Δy/Δyaw = 0) thì **dừng sửa tay** và báo LC kiểm phép chuyển frame / tạo lại prediction từ pipeline đúng. Khi chỉ **một hộp** lệch (case `one-box-z`: 1/13, 12 hộp giữ nguyên) thì đó là lỗi đối tượng — kiểm bằng nhiều góc nhìn và ảnh camera, không kết luận lỗi pipeline chỉ vì một hộp nổi/chìm.
- **Điều chưa chắc:**
  1. Không có ground truth cho KITTI demo nên **không** kết luận B là cấu hình "đúng"; chỉ nói B nhất quán hơn với quan sát bằng mắt trên ảnh Side.
  2. Ảnh Side **không đủ** để duyệt hình học từng hộp (chồng nhau theo phương x–z, không có chiều y). Cần Top/Front và ảnh camera cùng frame mà bộ Student **không có**.
  3. Vùng x > 30 m rất thưa; chưa kết luận được hộp B `[6]` (40,98 m) và `[9]` (55,58 m) là thật hay nhiễu.
  4. PCD dùng kênh hằng số (`rgb=0`, không có reflectance thật) → chưa đánh giá được ảnh hưởng của thiếu intensity.
  5. `z_ground=0,075` chỉ là ước lượng từ dữ liệu, không phải mặt đường đo; đường z=0 trên plot là đường tham chiếu.

## 5. File kèm theo

Thư mục output: `../ket-qua-ca-nhan-01/` (đặt cạnh gói, ngoài gói)

```
ket-qua-ca-nhan-01/
├── smoke.json                     status: passed
├── run-A/  boxes-demo-delta-0-voxel-0.16.json · side-…png · summary.csv
├── run-B/  boxes-demo-delta-1.73-voxel-0.16.json · side-…png · summary.csv
├── run-C/  boxes-demo-delta-1.73-voxel-0.32.json · side-…png · summary.csv
└── qc-cases/  case-correct · case-batch-z · case-one-box-z (+ side PNG, manifest.json)
```

## 6. Tự kiểm trước khi nộp LC

- [x] Chạy thật, không đọc kết quả có sẵn → ghi `executed-by-self`
- [x] Máy, image ID, checkpoint hash, input hash, cấu hình A/B/C đều có trong báo cáo
- [x] Đủ output: `smoke.json` (`passed`), `run-A/B/C` (JSON + Side PNG + CSV), `qc-cases` (3 ca + manifest)
- [x] So **A/B** và **B/C** bằng số liệu và dẫn tên file; **không** so A với C
- [x] Phân biệt rõ **dừng batch** (`batch-z`) với **kiểm từng hộp** (`one-box-z`)
- [x] Ghi đủ điều chưa chắc (5 mục)
- [x] **Không** import `case-*.json` hay prediction KITTI vào CVAT Robotaxi
- [x] Báo cáo + output giữ ngoài repo public

## 7. LC ghi nhận riêng

*(Phần này để LC điền — không tự điền.)*

- Quyền dù PCD/image và đúng ca:
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:
- Nhận xét của Thân Vĩnh Trọng và quyết định dừng pipeline:
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:

---
