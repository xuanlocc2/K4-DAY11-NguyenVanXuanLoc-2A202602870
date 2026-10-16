MSSV: 2A202602870

# Lab 11 · Scenario Forge — báo cáo

Họ tên: Nguyễn Văn Xuân Lộc. Mã dùng để sinh thị trấn và seed: `02870` (chạy với `LAB11_MSSV=02870`; `evidence/*.json: _run.mssv` = "02870"). Thị trấn (`meta.json: _run.town`): **C**. Mọi số lấy từ `evidence/` chạy đầy đủ (không `--quick`); số nào tính thêm ngoài `evidence/` được ghi "(tự tính)".

## 1. Phát lại log có trung thực không? (bước A)

`replay.json`: n = 400, `dropped_human_crash_logs` = 0, `n_used` = 400.

| | va chạm | theo loại tiếp xúc |
|---|---|---|
| phát lại (`replay`) | **119** | rear 105, side 14 |
| thế giới phản ứng (`reactive`) | **0** | – |

`crash_in_both` = 0: không có va chạm nào ở cả hai. 105/119 là `rear`: xe bám đuôi TG trong log chỉ lặp lại hành vi đã ghi theo người lái, không "thấy" planner. Planner phanh sớm hơn người lái nên TG cứ chạy như log và tông vào đuôi. Đó là va chạm **giả**: ngoài đời TG là người thật sẽ phanh theo. Ví dụ `replay_only[0]` (scenario 3, `contact = rear`, đường khô μ = 0,8, `min_ttc_reactive` = 2,16 s: ở thế giới phản ứng cùng kịch bản còn dư 2 s): ảnh `evidence/shots/R-01.png` cho va chạm xe bám đuôi lúc 16,5 s.

## 2. Kịch bản bất khả thi (bước B)

`is_feasible` loại nếu: (a) gia tốc ngang khi chuyển làn (π/cutDur)²·3,5/2 > μg: bánh xe không bám nổi; (b) `cutBrake` > μg: không xe nào phanh mạnh hơn ma sát; (c) `pedV` > 2,5 m/s: người đi bộ không chạy nhanh vậy ở đây; (d) `cutV` ∉ (0, 22] m/s: xe đứng yên hoặc quá nhanh so với phố 50 km/h.

`hunt.json: infeasible`: bus_bypass 0, cut_in 2, crosswalk_block 5, rear_ended 7 (log hunt: "infeasible skipped"); `mc.json: infeasible` = 22/3000. Ở cuộc săn có điểm "nguy hiểm nhất" (cắt làn ngắn, phanh vượt ma sát, người đi bộ quá nhanh) CEM bị kéo về đúng những kịch bản bất khả thi, vì chúng có điểm cao nhất. Nếu đếm chúng là va chạm, ước lượng bị thổi phồng bằng thất bại không thể xảy ra và q co về vùng vô nghĩa. Ở đây chúng bị bỏ ra, không chạy và không đếm.

## 3. Monte Carlo cho bạn biết gì (bước C1)

`mc.json`: n = 3000, `crashes` = **6**, p̂ = 2,00·10⁻³, Wilson 95 % [**9,17·10⁻⁴**, **4,36·10⁻³**]; theo kiểu `modes`: bus_bypass 5, crosswalk_block 1.
Để khoảng hẹp còn ±20 % của p cần n ≈ 96/p̂ = 96/2,0·10⁻³ ≈ **48 000** lần chạy (tự tính), gấp 16 lần ngân sách MC (3000).
Chưa thấy trong 3000 lần: `cut_in`, `rear_ended`, `other` đều k = 0 (`mc.json: per_mode`). Không báo 0: cận trên Wilson mỗi kiểu là **1,28·10⁻³** (`per_mode.*.hi`). Trong `estimate.json`, `cut_in` và `rear_ended` có `upper_zero_hits` = 0,025 (cận trên cho ước lượng IS khi 0 hit).

## 4. Importance sampling, ESS và độ suy biến (bước C3)

`estimate.json` (`budget` = **16 800** lần chạy, 3 seed, n = 600 mỗi kiểu):

| seed | kiểu | p | hits | ess | top_share |
|---|---|---|---|---|---|
| 0 | bus_bypass | 8,33·10⁻⁴ | 28 | 147 | 0,582 |
| 0 | crosswalk_block | 1,40·10⁻⁵ | 89 | 110 | **0,921** |
| 0 | cut_in / rear_ended | 0 (≤ 0,025) | 0 | 138 / 114 | 0 |
| 1 | bus_bypass | 2,17·10⁻³ | 34 | 112 | 0,431 |
| 1 | crosswalk_block | 2,13·10⁻⁶ | 79 | 108 | 0,525 |
| 1 | cut_in / rear_ended | 0 (≤ 0,025) | 0 | 134 / 112 | 0 |
| 2 | bus_bypass | 2,40·10⁻³ | 32 | 110 | 0,473 |
| 2 | crosswalk_block | 9,17·10⁻⁷ | 77 | 131 | 0,550 |
| 2 | cut_in / rear_ended | 0 (≤ 0,025) | 0 | 119 / 132 | 0 |

`total` theo seed: 8,47·10⁻⁴ [0, 3,10·10⁻³]; 2,17·10⁻³ [6,1·10⁻⁵, 5,57·10⁻³]; 2,40·10⁻³ [3,8·10⁻⁵, 6,04·10⁻³].
- Các seed **không** đồng ý chặt: bus_bypass chênh ×2,9 (8,3·10⁻⁴ so với 2,4·10⁻³), `crosswalk_block` chênh gần ×15 (1,4·10⁻⁵ so với 9,2·10⁻⁷). So với MC (p̂ = 2,0·10⁻³, Wilson [9,2·10⁻⁴, 4,4·10⁻³]): seed 1 và 2 nằm trong khoảng, seed 0 (8,5·10⁻⁴) hơi thấp hơn mép dưới.
- **Suy biến:** seed 0 `crosswalk_block` có `top_share` = 0,921, tức 92 % ước lượng đến từ **một** mẫu dù có 89 hit, nên con số 1,4·10⁻⁵ gần như là trọng số của một kịch bản, không đáng tin. Seed 0 `bus_bypass` (0,582), seed 1 và 2 `crosswalk_block` (0,525; 0,550) cũng > 0,5. `ess` ≈ 110–147 trên 600 mẫu: trọng số lệch.
- **Mâu thuẫn với MC (tôi chưa giải được):** MC thấy 1 va chạm `crosswalk_block` trong 3000 → p ∈ [5,9·10⁻⁵, 1,9·10⁻³] (`mc.json: per_mode.crosswalk_block`). IS cho 10⁻⁶–10⁻⁵ với cận trên cao nhất 3,9·10⁻⁵ (seed 0), tức **thấp hơn cận dưới của MC**. Một hit không đủ để kết luận MC đúng, nhưng IS có vẻ đánh giá thấp: giả thuyết (chưa kiểm) là q của CEM chỉ phủ một phần vùng thất bại crosswalk_block nên bỏ sót phần còn lại. Vì vậy tôi không dùng con số IS của crosswalk_block làm "xác suất"; nó chỉ là cận dưới thô.
- **Ngân sách:** IS dùng 16 800 lần cho cả 4 kiểu; MC cần ≈ 48 000 lần (tự tính) chỉ để `total` đạt ±20 %, tức IS rẻ hơn ≈ 2,9× nhưng chưa đạt ±20 % (khoảng ở trên rộng). Với `crosswalk_block` ở mức 10⁻⁶, MC cần ≈ 96/10⁻⁶ ≈ 10⁸ lần (tự tính): không khả thi, đó là chỗ IS có ích nếu nó đúng.
- `other` lấy từ `mc.json` (k = 0, n = 3000, p = 0, `hi` = 1,28·10⁻³; `estimate.json: other`) vì kiểu này không được săn nên không có q cho nó.

## 5. Các kiểu thất bại (bước C2 + 3D)

`hunt.json: found`: bus_bypass → bus_bypass 111; cut_in → chỉ bus_bypass 27 (không có cut_in); crosswalk_block → crosswalk_block 32; rear_ended → bus_bypass 16, crosswalk_block 5 (không có rear_ended).
- **bus_bypass** (`evidence/shots/B-01.png`, B-02..B-04): đường ướt (μ 0,5), người đi bộ không ngó qua xe buýt, khe chấp nhận của xe cắt làn rất hẹp (B-01: 4,0 m; B-02: 4,7 m; B-03: 3,0 m; B-04: 4,3 m). Chuỗi sự kiện ở B-01: xe C2 lách khỏi hàng sau xe buýt vào khe hẹp, ego chỉ coi C2 là xe phía trước khi nó đã lấn gần hơn `cutinLat` = 2,15 m nên phanh muộn → va chạm xe cắt làn lúc 12,5 s. Ở khung đọc của viewer, run này hiện "không va chạm" cho policy đã sửa (cutinLat 3,3).
- **crosswalk_block** (`C-01.png`, C-02..C-04): đường ướt, người đi bộ không ngó qua xe buýt, khe 3,2–5,9 m. Ở C-01 ego dừng với thân xe nằm trên vạch và chặn người đi bộ lúc 19,6 s (`kind: block`); không có cú tông. Quan trọng: C-01, C-03, C-04 đều bị chặn ở **cả** policy gốc và policy đã sửa (bản sửa cutinLat không đụng đến kiểu này); chỉ C-02 được sửa.
- **Không tìm thấy:** `cut_in` và `rear_ended` (0 hit ở hunt, MC, IS). Chỉ biết cận trên 0,025 (IS) và 1,28·10⁻³ (MC); "không thấy" ≠ "không có".
- **Nếu chỉ một cuộc săn chung:** hunt cut_in và rear_ended cho thấy CEM bị kéo về kiểu dễ nhất (cả 27 va chạm tìm thấy ở hunt cut_in là bus_bypass; 16/21 ở hunt rear_ended), nên một cuộc săn chung sẽ gần như chỉ ra bus_bypass và bỏ sót kiểu hiếm hơn. Mỗi kiểu cần cuộc săn và hàm điểm riêng.

## 6. Sửa planner (bước E + 3D)

`fix.json` (policy chọn trên TRAIN: `{cutinLat: 3.3}`; `cutinLat: 3.0` có cùng `fail`, 3,3 thắng ở `progress`):

| | fail | fail_hi | fail_hi_seen | unseen | progress (m) |
|---|---|---|---|---|---|
| baseline, TRAIN | 8,59·10⁻⁴ | 1,67·10⁻³ | 1,67·10⁻³ | – | 216,6 |
| chọn, TRAIN | 3,6·10⁻⁷ | 6,9·10⁻⁷ | 6,9·10⁻⁷ | bus_bypass | 216,2 |
| baseline, TEST | 1,48·10⁻³ | 4,18·10⁻³ | 2,90·10⁻³ | – | 211,1 |
| chọn, TEST | 1,82·10⁻⁶ | **3,88·10⁻²** | 3,8·10⁻⁶ | bus_bypass | 210,8 |

Tiến độ giữ ≈ 99,8 % baseline; phanh gắt trung bình `harsh`: 7,68 → 7,19 (TRAIN), 9,02 → 8,09 (TEST).
- **Cận trên TEST của bản sửa (3,88·10⁻²) lớn hơn `fail` của baseline (1,48·10⁻³):** bus_bypass dưới policy chọn có 0 hit nên cộng `upper_zero_hits` = 3/(0,2·400) = 0,0375. Đó là giới hạn của n = 400, không phải bản sửa tệ hơn: `fail_hi_seen` = 3,8·10⁻⁶ ≪ 2,90·10⁻³. Không kết luận được "an toàn hơn 10³ lần", chỉ "chưa thấy bus_bypass".
- **crosswalk_block không đổi:** ở TRAIN, baseline và bản chọn cùng 48 hit, 3,55·10⁻⁷. TEST baseline có `top_share` crosswalk 0,92 (một mẫu), chọn 0,54: số này không so sánh được.
- **Ảnh có bóng mờ:** B-01 và B-02..B-04 (ego gốc va chạm; policy sửa cho "không va chạm" trong khung đọc của viewer). Nhãn khung đọc ở các run của hunt có vẻ đảo ("chính sách của bạn" hiện kết quả của policy gốc), nên mình dựa vào `fix.json` làm nguồn chính, ảnh để minh họa.
- **Núm nghe an toàn hơn nhưng không tốt hơn:** `cautionV` = 5,0 (chậm hẳn): `fail` 1,25·10⁻³ **cao hơn** baseline 8,59·10⁻⁴, mất 44 m quãng đường (172,2 m) và có `unseen: crosswalk_block`; `cautionV` = 6,5 cho 2,74·10⁻³, còn 6,0 cho 3,52·10⁻⁴: không đơn điệu. Giả thuyết (chưa tách riêng): chậm hơn làm đổi thời điểm đến vạch và khe cắt làn của xe khác, tạo tương tác khác chứ không chỉ "ít nguy hiểm hơn".

## 7. Từ mô phỏng ra đường thật

1. **Khe chấp nhận của xe cắt làn** (`cutGap`, log-chuẩn, trung vị 10 m ở thị trấn C): thay bằng khe đo từ quỹ đạo thật (NGSIM, highD, Waymo Open Motion). Nếu khe thật lớn hơn 10 m, bus_bypass bị tôi **ước lượng cao** (cả 4 ảnh B-xx có khe 3,0–4,7 m).
2. **Tỉ lệ người đi bộ ngó qua xe buýt** (`pedLook` = 0,80): thay bằng quan sát hành vi người đi bộ ở trạm xe buýt (camera giao lộ, nghiên cứu hành vi). Nếu ít người ngó hơn 80 % thì crosswalk_block bị **ước lượng thấp** (cả 4 ảnh C-xx đều là trường hợp không ngó).
3. **Ma sát đường ướt** (`wet` = 0,20; μ = 0,5): thay bằng tỉ lệ giờ mưa của thành phố (khí tượng) và ma sát đo trên mặt đường thật. Nếu mưa nhiều hơn hoặc μ thấp hơn, mọi kiểu thất bại bị **ước lượng thấp** (tất cả ảnh B-xx và C-xx đều ướt); nếu ít mưa thì cao.

## 8. Giới hạn

Xác suất của tôi **không** nói: (1) về mọi con phố: thế giới chỉ có một đoạn phố, một xe buýt, một vạch qua đường, không có ngã tư, xe máy hay đèn giao thông; (2) về người lái thật: các xe khác là IDM + khe chấp nhận, không có người phanh bất ngờ hay phân tâm; (3) về cảm biến: hoàn hảo trừ phần bị xe buýt che, không nhiễu hay bỏ sót. Ngoài ra con số IS của crosswalk_block mâu thuẫn với MC (mục 4) và các seed chênh nhau ×2,9 (bus_bypass). Nếu có thêm một tuần: kiểm lại q của crosswalk_block (ví dụ tăng `final_n`), tăng `fix_n` để cận trên TEST của bus_bypass xuống dưới 10⁻², và thử cut_in / rear_ended với hàm điểm riêng.

## 9. Rắc rối của bạn (bước G + chương 10)

- **Rắc rối:** một người đi bộ băng ngang giữa đoạn đường, chỗ **không có vạch** (`hazard.json`: một hazard `ped`, x ≈ 175,9 m, v ≈ 1,93 m/s, `trig` ≈ 41,9 m). Ngoài đời là người băng đường ẩu (jaywalking).
- **Căn cứ:** planner chỉ có logic người đi bộ cho vạch qua đường; người băng ngoài vạch không được dự đoán nên phanh muộn. `why` trong hazard.json dẫn NHTSA Traffic Safety Facts (Pedestrians, 2023): ~74 % người đi bộ tử vong xảy ra ngoài giao lộ.
- **Giả định:** `space` = x ∈ [110, 220] m, trig ∈ [30, 70] m, v ∈ [1,2; 2,2] m/s (dưới giới hạn 2,5 m/s của `is_feasible`). `exposure` = 0,02 là **giả định bậc độ lớn của tôi, không có số liệu tần suất**; `per_trip` tỉ lệ thuận với nó.
- **Kết quả** (`mine.json`, 1494/1500 hợp lệ → `accepted` = 0,996; `infeasible` = 6; `exact`: ego va chạm H0 lúc 21,7 s):

| policy | va chạm | avoidable | unavoidable | unattributed | low_speed | P(lỗi planner \| rắc rối) [Wilson] | per_trip |
|---|---|---|---|---|---|---|---|
| gốc (`baseline`) | 65 | **59** | 0 | 6 | 0 | 3,95·10⁻² [3,07·10⁻², 5,06·10⁻²] | 7,90·10⁻⁴ (hi 1,01·10⁻³) |
| chọn cutinLat 3,3 (`chosen`) | 59 | **59** | 0 | 0 | 0 | 3,95·10⁻² [3,07·10⁻², 5,06·10⁻²] | 7,90·10⁻⁴ |

  `per_trip` = exposure × p = 0,02 × 3,95·10⁻². Đặt cạnh lách xe buýt: MC p̂ = 2,0·10⁻³ (`mc.json`), IS 8,5·10⁻⁴–2,4·10⁻³ (`estimate.json: seeds[*].total`): cùng bậc độ lớn, nhưng `per_trip` này phụ thuộc exposure giả định nên chỉ so được về bậc độ lớn.
- **Hình:** `evidence/shots/GB-01..03.png` (policy gốc: va chạm người băng ngang; GB-01 lúc 23,3 s, đường khô, có ngó) và `GC-01..03.png` (policy chọn): bản sửa cũng va chạm cùng các kịch bản nên không có ảnh "bản sửa tránh được".
- **Tránh được hay không:** 59/59 vụ là `avoidable`: phanh hết cỡ ngay khi thấy sẽ tránh được (`avoidable_foreknown` = `avoidable` = 59: tầm nhìn không bị che). Chuẩn "tránh được" lạc quan theo hai hướng: người phanh hết cỡ biết trước nên đếm **nhiều hơn** số vụ một planner thật tránh được, còn chỉ thử phanh (không đánh lái) nên đếm **ít đi**.
- **Bản sửa bước 6 không cứu rắc rối này:** `chosen` vẫn 59 vụ avoidable (6 vụ unattributed của baseline là va chạm bus_bypass bình thường, bản sửa loại bỏ đúng chúng). Đúng mục tiêu "tránh được mà bản sửa không cứu".
- **Bạn tự lái thì sao:** tôi chưa tự lái trong chương 10, không có số so sánh.
- **Đề xuất sửa:** đây là thiếu logic chứ không phải núm chỉnh sai: planner cần theo dõi mọi người đi bộ gần mép đường (không chỉ P1 ở vạch), dự đoán họ băng qua làn ego và giảm tốc trước (pre-brake theo thời gian tới va chạm). Tôi chỉ chạy `mine` với policy gốc và policy chọn, chưa thử các ứng viên khác trong `fix.json: table` (núm gần nhất là `pedBuffer`).
