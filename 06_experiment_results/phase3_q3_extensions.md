# Giai đoạn 3 — ba việc bổ sung cho bản tạp chí (Q3)

Ngày 13/10/2026. Căn cứ: bản duyệt nội bộ của `08_final_submission/ICIIT2027_paper_v3.docx`.

| # | Việc | Trạng thái |
|---|---|---|
| 1 | Đọc toàn văn Theodorou et al. (2025) và định vị lại | ⏳ chờ người dùng lấy PDF |
| 2 | Quy tắc chọn phương pháp theo dự đoán tăng trưởng | ✅ xong (mục 2) |
| 3 | Độ nhạy theo siêu tham số | ✅ xong (mục 3) |

Mọi phân tích dùng 7 phương pháp. Phần diễn giải gắn nhãn [Nhận định nhóm].

## 1. Theodorou et al. (2025)

- DOI: https://doi.org/10.1016/j.ejor.2024.12.033
- ScienceDirect: https://www.sciencedirect.com/science/article/pii/S0377221724009755
- Thông tin: E. Theodorou, E. Spiliotis, V. Assimakopoulos. Forecast accuracy and inventory performance: Insights on their relationship from the M5 competition data. *European Journal of Operational Research* 322(2), 414–426.

Khi có toàn văn, cần đối chiếu:

1. các phương pháp dự báo họ dùng;
2. chính sách tồn kho, lead time, chu kỳ xem xét;
3. cách chia kiểu nhu cầu;
4. số giai đoạn kiểm thử;
5. kết luận chính và các con số;
6. họ có so sánh ở cùng mức phục vụ hay theo chi phí.

Sau đó sửa mục 1, 2 và 6 của bài, và bỏ câu "we had access to the abstract of [7] only".

## 2. Quy tắc chọn phương pháp theo dự đoán tăng trưởng

Nguồn: `code/outputs/growth_rule.md` (script `code/growth_rule.py`).

### Thiết kế

Không có thông tin nào của cửa sổ đánh giá được dùng trước khi đánh giá.

1. Trên cửa sổ huấn luyện, gán nhãn "tăng mạnh" cho chuỗi có nhu cầu kiểm thử > 1,5 lần 26 tuần trước đó.
2. Huấn luyện bộ phân loại trên 9 đặc trưng trước kiểm thử.
3. Cũng trên cửa sổ huấn luyện, chọn:
   - A = phương pháp hiệu quả nhất cho chuỗi tăng mạnh;
   - B = phương pháp hiệu quả nhất cho các chuỗi còn lại;
   - tỷ lệ chuỗi giao cho A, sao cho tồn kho của chính sách ghép nhỏ nhất ở fill rate 0,92 (dùng xác suất dự đoán ngoài mẫu).
4. Áp dụng nguyên trạng cho cửa sổ kế tiếp; so với việc luôn dùng LGB-Q ở cùng fill rate, với bootstrap theo chuỗi.

Có ba lần chuyển: w52 → w26; w26 → main; (w52 + w26) → main.

### Những gì được học và chất lượng dự đoán tăng trưởng

| Dataset | Lần chuyển | A (chuỗi dự đoán tăng) | B (còn lại) | Tỷ lệ giao cho A | AUC ở cửa sổ kế tiếp |
|---|---|---|---|---|---|
| M5 | w52 → w26 | HGB-Q | LGB-Q | 30% | 0,77 |
| M5 | w26 → main | LGB-Q | LGB-Q | 5% | 0,81 |
| M5 | w52 + w26 → main | LGB-Q | LGB-Q | 5% | 0,83 |
| VN1 | w52 → w26 | LGB-Q | TSB-NB | 70% | 0,66 |
| VN1 | w26 → main | LGB-C | TSB-NB | 5% | 0,67 |
| VN1 | w52 + w26 → main | LGB-C | TSB-NB | 70% | 0,70 |

- Trên VN1, ở cả ba lần học, phương pháp cho chuỗi ổn định là **TSB-NB** và phương pháp cho chuỗi tăng mạnh là một mô hình ML. Điều này khớp với phân tích dịch chuyển nhu cầu.
- Trên M5, quy tắc tự thu về LGB-Q, tức không có gì để chuyển đổi.

### Kết quả trên VN1

Chênh lệch tồn kho so với luôn dùng LGB-Q (%, khoảng tin cậy 95%). Số âm = cần ít tồn kho hơn.

| Lần chuyển | Quy tắc | fill 0,90 | fill 0,92 | fill 0,94 |
|---|---|---|---|---|
| w52 → w26 | Luôn TSB-NB | −8,8 [−12,8; −5,8] | −11,3 [−15,6; −7,6] | −8,8 [−13,0; −5,7] |
| w52 → w26 | **Quy tắc tăng trưởng** | **−5,2 [−8,6; −2,4]** | **−7,0 [−12,7; −3,5]** | **−6,6 [−10,3; −3,5]** |
| w52 → w26 | Tiên tri (biết trước chuỗi nào tăng) | −6,4 | −8,7 | −9,8 |
| w26 → main | Luôn TSB-NB | +6,4 [+1,2; +13,4] | +5,0 [−0,5; +13,5] | +10,5 [+2,6; +20,6] |
| w26 → main | Quy tắc tăng trưởng (học 1 cửa sổ) | +4,1 [−1,4; +11,3] | +3,0 [−3,0; +11,8] | +9,2 [+0,6; +20,1] |
| w52 + w26 → main | **Quy tắc tăng trưởng (học 2 cửa sổ)** | −2,6 [−6,9; +1,2] | **−5,2 [−10,4; −0,5]** | +0,5 [−6,3; +6,0] |
| → main | Tiên tri | −5,6 [−9,4; −1,9] | −9,7 [−13,6; −4,0] | −6,6 [−11,6; −2,2] |

Trên M5, mọi quy tắc trùng với LGB-Q (chênh lệch từ 0 đến −0,3%).

### Độ nhạy theo tỷ lệ giao cho A

Cửa sổ kế tiếp, fill rate 0,92, chênh lệch so với LGB-Q (%):

| Lần chuyển (VN1) | 5% | 10% | 20% | 30% | 40% | 50% | 60% | 70% |
|---|---|---|---|---|---|---|---|---|
| w52 → w26 | −10,7 | −10,3 | −9,8 | −9,7 | −9,4 | −8,3 | −7,2 | −7,0 |
| w26 → main | +3,0 | +1,6 | +0,7 | −0,3 | −1,0 | −1,1 | −0,4 | −0,4 |
| w52 + w26 → main | −2,7 | −3,7 | −4,4 | −4,5 | −5,5 | −5,5 | −5,0 | −5,2 |

### Kết luận

1. **"Tăng trưởng" là đúng chiều để chọn phương pháp.** Nếu biết trước chuỗi nào tăng mạnh (dòng "tiên tri"), chính sách ghép cần ít tồn kho hơn LGB-Q 5,6–9,8% ở **cả hai** cửa sổ đánh giá của VN1, kể cả cửa sổ chính, nơi không phương pháp đơn lẻ nào và không quy tắc theo nhóm nào hơn được LGB-Q. Điều này trái với "tiên tri theo phân khúc" ở `phase2_b3_b4.md`, vốn không có lợi ích nào.
2. **Quy tắc dùng dự đoán thu được một phần lợi ích đó:**
   - ở cửa sổ thứ hai: −5 đến −7%, có ý nghĩa thống kê;
   - ở cửa sổ chính, khi học trên hai cửa sổ: −5,2% tại fill rate 0,92 (có ý nghĩa), không khác LGB-Q tại 0,90 và 0,94.
3. **Quy tắc thu hẹp rủi ro chọn sai phương pháp.** Luôn dùng TSB-NB thì được −11% ở một cửa sổ nhưng mất +5 đến +10% ở cửa sổ kia; luôn dùng LGB-Q thì ngược lại. Quy tắc học trên hai cửa sổ không kém LGB-Q một cách có ý nghĩa ở cửa sổ nào.
4. **Hạn chế** (phải nêu trong bài):
   - Quy tắc học trên **một** cửa sổ thì không ổn định: tỷ lệ giao cho A được chọn là 5%, quy tắc gần như trở thành TSB-NB, và kém LGB-Q +9,2% [+0,6; +20,1] tại fill rate 0,94 ở cửa sổ chính.
   - Chỉ có hai cửa sổ đánh giá và ba lần chuyển, trong đó hai lần dùng chung cửa sổ đánh giá.
   - Bộ dự đoán tăng trưởng mới đạt AUC 0,66–0,70 ngoài mẫu trên VN1.
   - Trên M5 quy tắc không mang lại gì.
   - [Nhận định nhóm] Đây là bằng chứng ban đầu cho một hướng đi, chưa phải một phương pháp đã được kiểm chứng.

## 3. Độ nhạy theo siêu tham số

Nguồn: `code/outputs/hparam_sensitivity.md` (script `code/hparam_sensitivity.py`); bản sao ở `06_experiment_results/tables/hparam_sensitivity.md`.

### Thiết kế

LGB-Q và LGB-T được huấn luyện lại ở h = 3, cửa sổ chính, với 4 cấu hình LightGBM. TSB-NB không có siêu tham số loại này.

| Cấu hình | Số lá | Mẫu tối thiểu mỗi lá | Khác |
|---|---|---|---|
| default (dùng trong bài) | 63 | 200 | λ₂ = 1; feature fraction 0,8 |
| small_trees | 15 | 100 | |
| large_trees | 255 | 50 | |
| regularised | 31 | 500 | λ₂ = 10; feature fraction 0,6 |

### Kết quả VN1

Chênh lệch tồn kho (%, khoảng tin cậy 95%). Hai cột Δ LGB-T và Δ TSB-NB: số dương = LGB-Q (cùng cấu hình) cần ít tồn kho hơn.

| Cấu hình | wQL LGB-Q | wQL LGB-T | Fill 0,94: Δ LGB-T | Fill 0,94: Δ TSB-NB | LGB-Q so với cấu hình default, fill 0,94 |
|---|---|---|---|---|---|
| default | 0,1026 | 0,1215 | +0,9 [−6,1; +9,2] | +10,5 [+0,9; +19,4] | — |
| small_trees | 0,1071 | 0,1216 | −5,9 [−10,8; +1,5] | +4,0 [−4,8; +11,4] | +6,3 [+3,8; +8,6] |
| large_trees | 0,1015 | 0,1195 | +0,3 [−6,9; +8,7] | +14,5 [+3,5; +23,7] | −3,5 [−5,1; −1,7] |
| regularised | 0,1036 | 0,1213 | −1,3 [−7,3; +6,2] | +9,9 [+0,3; +18,7] | +0,5 [−0,6; +1,8] |

Tại fill rate 0,92: Δ LGB-T là −2,5 / −8,9 / −4,5 / −3,6 (chỉ small_trees có khoảng tin cậy không chứa 0); LGB-Q so với default là +8,1 / −0,9 / +1,2.

**Kết luận cho VN1:**

- Cấu hình default nằm trong nhóm tốt: cây nhỏ kém hơn rõ (+6 đến +8% tồn kho); cây lớn nhỉnh hơn (−3,5% tại fill rate 0,94); regularised tương đương.
- Kết luận của bài về VN1 **không đổi** ở ba trên bốn cấu hình:
  - LGB-Q và LGB-T không khác nhau có ý nghĩa ở cửa sổ chính;
  - LGB-Q cần ít tồn kho hơn TSB-NB tại fill rate 0,94.
- Với cây nhỏ, LGB-Q mất lợi thế trước cả hai.
- wQL của LGB-Q thấp hơn LGB-T ở mọi cấu hình (0,10 so với 0,12).

### Kết quả M5

Cùng cách đọc như bảng VN1. Fill rate 0,94:

| Cấu hình | wQL LGB-Q | wQL LGB-T | Tồn kho LGB-Q | Δ LGB-T | Δ TSB-NB | LGB-Q so với cấu hình default |
|---|---|---|---|---|---|---|
| default | 0,0761 | 0,0833 | 1,376 | +5,1 [+4,4; +5,8] | +10,0 [+9,0; +10,9] | — |
| small_trees | 0,0771 | 0,0843 | 1,384 | +5,8 [+5,0; +6,6] | +9,3 [+8,4; +10,1] | +0,6 [+0,4; +0,8] |
| large_trees | 0,0759 | 0,0834 | 1,366 | +5,7 [+4,8; +6,3] | +10,8 [+10,0; +11,6] | −0,7 [−1,0; −0,5] |
| regularised | 0,0761 | 0,0836 | 1,376 | +5,5 [+4,7; +6,2] | +10,0 [+9,1; +10,8] | +0,0 [−0,1; +0,2] |

Tại fill rate 0,92: Δ LGB-T là +5,5 / +4,7 / +6,8 / +5,3 và Δ TSB-NB là +10,4 / +7,5 / +11,0 / +9,8 (default / small / large / regularised), mọi khoảng tin cậy đều nằm hẳn trên 0; LGB-Q so với default là +2,6 / −0,5 / +0,6.

**Kết luận cho M5:**

- Kết luận của bài **không đổi ở cả bốn cấu hình**: LGB-Q cần ít tồn kho hơn LGB-T 4,7–6,8% và ít hơn TSB-NB 7,5–11,0%, mọi khoảng tin cậy đều không chứa 0.
- Tồn kho của chính LGB-Q thay đổi rất ít theo cấu hình: trong khoảng −0,7% đến +2,6% so với default.
- wQL của LGB-Q thấp hơn LGB-T ở mọi cấu hình (0,076 so với 0,083).
- [Nhận định nhóm] Lợi thế của LGB-Q trên M5 đến từ cách dự báo (phân vị trực tiếp so với dự báo điểm + tồn kho an toàn), không phải từ một cấu hình siêu tham số may mắn.

### Kết luận chung về siêu tham số

- M5: kết luận vững với cả bốn cấu hình.
- VN1: kết luận vững với ba trên bốn cấu hình; với cây nhỏ (15 lá), LGB-Q mất lợi thế. Điều này phù hợp với nhận định trong bài rằng kết luận trên VN1 kém vững hơn M5.
- Câu "Hyper-parameters are not tuned" trong Limitations có thể kèm theo: "M5 conclusions hold under four LightGBM settings" (chi tiết trong repo).
