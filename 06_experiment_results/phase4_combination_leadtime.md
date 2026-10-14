# Giai đoạn 4 — Kết hợp dự báo (việc A) và lead time ở cùng fill rate (việc 3)

Ngày 14/10/2026. Phần diễn giải gắn nhãn [Nhận định nhóm].

Nguồn:

- `code/combination.py` → `code/outputs/combination.md`, `combination.csv`, `combination_regret.csv`;
- `code/leadtime_equal_fill.py` → `code/outputs/leadtime_equal_fill.md`, `.csv`;
- bản sao ở `06_experiment_results/tables/`.

Cả hai chỉ dùng dự báo đã có trong cache; không huấn luyện lại mô hình nào.

**Kiểm chứng:**

- Bảng regret của 7 phương pháp đơn tái lập đúng số trong bài: LGB-Q 9,4%, HGB-Q 11,4%, LGB-T 9,5%, TSB-NB 20,7%.
- Ở L = 2, kết quả lead time khớp Bảng 3: LGB-Q cần 1,38 tuần tồn kho tại fill rate 0,94 trên M5; TSB-NB cần thêm 10,0%.

## A. Kết hợp dự báo phân vị

### Thiết kế

Mỗi phân vị của tổ hợp là trung bình cộng các phân vị của các phương pháp thành viên. Hai tổ hợp được cố định trước khi xem kết quả:

- **C2** = trung bình (LGB-Q, TSB-NB): tổ hợp chính;
- **C3** = trung bình (LGB-Q, HGB-Q, TSB-NB): tổ hợp phụ.

Chính sách mặc định: h = 3, L = 2, không thanh lý. Chạy trên ba cửa sổ của cả hai dataset.

**Lưu ý về chọn lựa:** hai thành viên của C2 được chọn vì phân tích chính cho thấy chúng thắng xen kẽ trên VN1, trên **cùng** các cửa sổ này. Đây là lựa chọn hiển nhiên (mô hình ML tốt nhất + mô hình thống kê tốt nhất), nhưng vẫn là lựa chọn sau khi đã thấy dữ liệu, nên phải nói rõ trong bài.

### Kết quả

Tồn kho so với LGB-Q (%, khoảng tin cậy 95%; số âm = cần ít tồn kho hơn):

| Dataset | Cửa sổ | C2, fill 0,90 | C2, fill 0,92 | C2, fill 0,94 | TSB-NB, fill 0,94 |
|---|---|---|---|---|---|
| M5 | chính | +0,8 [+0,5; +1,1] | +0,4 [0,0; +0,7] | +0,9 [+0,5; +1,3] | +10,0 |
| M5 | thứ hai | +0,2 [−0,1; +0,5] | +0,1 [−0,2; +0,4] | −0,3 [−0,6; 0,0] | +7,5 |
| M5 | thứ ba | −1,0 [−1,5; −0,7] | −1,5 [−2,1; −1,1] | −1,1 [−1,6; −0,7] | +4,9 |
| VN1 | chính | **−5,8 [−9,7; −0,9]** | **−5,9 [−8,7; −2,2]** | **−8,1 [−10,8; −4,8]** | +10,5 |
| VN1 | thứ hai | **−9,0 [−10,9; −7,2]** | **−10,5 [−15,2; −8,0]** | **−11,2 [−13,0; −9,3]** | −8,8 |
| VN1 | thứ ba | **−5,5 [−8,8; −1,5]** | **−7,1 [−10,4; −3,0]** | −5,3 [−10,9; +4,9] | không đạt |

Regret: phần tồn kho dư so với phương pháp **đơn** tốt nhất trong mỗi lần chạy, lấy trung bình trên các mức fill 0,90–0,98 mà phương pháp đạt được. Số âm = tốt hơn mọi phương pháp đơn.

| Phương pháp | M5 chính | M5 thứ hai | M5 thứ ba | VN1 chính | VN1 thứ hai | VN1 thứ ba | **Lớn nhất** |
|---|---|---|---|---|---|---|---|
| C3 | −0,3 | −1,3 | −2,2 | −6,7 | +0,2 | −6,5 | **0,2** |
| C2 | +0,9 | −0,6 | −1,4 | −7,7 | −1,4 | −5,3 | **0,9** |
| LGB-Q | 0 | 0 | 0 | +0,7 | +9,4 | +0,2 | 9,4 |
| LGB-T | +6,0 | +7,1 | +3,8 | +0,3 | +5,8 | +9,5 | 9,5 |
| HGB-Q | +2,3 | +2,0 | +0,9 | +5,9 | +11,4 | +2,4 | 11,4 |
| TSB-NB | +11,5 | +8,4 | +6,5 | +10,4 | +3,7 | +20,7 | 20,7 |

Độ chính xác (wQL) của C2 kém LGB-Q một chút ở mọi lần chạy: ví dụ VN1 cửa sổ chính 0,108 so với 0,103.

### Kết luận

1. **Trên VN1, C2 cần ít tồn kho hơn LGB-Q ở cả ba cửa sổ** (5–11%; khoảng tin cậy không chứa 0, trừ một ô). Ở cửa sổ thứ hai, nơi TSB-NB thắng, C2 ngang TSB-NB (chênh −2,6% đến +0,9%, không có ý nghĩa).
2. **Trên M5, C2 nằm trong khoảng ±1,5% so với LGB-Q.** Thêm TSB-NB không làm hỏng kết quả.
3. **Regret lớn nhất của C2 chỉ 0,9%, so với 9,4% của LGB-Q.** Tổ hợp chính là lựa chọn ít hối tiếc nhất, tốt hơn bất kỳ phương pháp đơn nào. Kết quả này **thay thế** kết luận cũ của bài "LGB-Q is the minimax-regret choice".
4. C2 kém LGB-Q về wQL nhưng tốt hơn về tồn kho, thêm một ví dụ cho luận điểm "độ chính xác ≠ hiệu quả tồn kho".
5. So với quy tắc tăng trưởng (`phase3_q3_extensions.md` mục 2), C2 đơn giản hơn, không cần dự đoán, và ổn định ở cả ba cửa sổ. Quy tắc tăng trưởng chỉ có ý nghĩa ở một cửa sổ.
6. Kết quả khớp với Petropoulos et al. [24]: kết hợp dự báo cho kết quả tồn kho tốt nhất.
7. [Nhận định nhóm] Cách giải thích hợp lý: hai thành viên sai theo hướng khác nhau. TSB-NB phủ thiếu với chuỗi tăng mạnh; LGB-Q dư tồn kho với chuỗi ổn định. Trung bình hai bên giảm cả hai lỗi. Chưa kiểm chứng trực tiếp bằng phân tích theo nhóm chuỗi ổn định / tăng mạnh.

## 3. Độ nhạy theo lead time ở cùng fill rate

### Thiết kế

- L = 1, 2, 4 tuần (R = 1).
- Dùng dự báo đã cache của cửa sổ chính cho h = L + 1, tức h = 2, 3, 5.
- HGB-Q không có trong phân tích vì không có dự báo ở h = 2 và h = 5.

### Kết quả

Δ so với LGB-Q tại fill rate 0,94 (%; số dương = LGB-Q cần ít tồn kho hơn):

| Dataset | L | LGB-T | LGB-C | TSB-NB | ETS | C2 |
|---|---|---|---|---|---|---|
| M5 | 1 | +6,5* | +6,5* | +14,8* | +9,8* | +1,7* |
| M5 | 2 | +5,1* | +4,5* | +10,0* | +10,6* | +0,9* |
| M5 | 4 | +6,3* | +8,6* | +9,6* | +13,4* | −0,6* |
| VN1 | 1 | +0,2 | +2,5 | +16,0* | +21,0* | −4,1* |
| VN1 | 2 | +0,9 | +1,7 | +10,5* | +12,0* | −8,1* |
| VN1 | 4 | +3,7 (không có CI) | −0,5 | +4,8 (không có CI) | +7,9 | −5,4* |

\* = khoảng tin cậy 95% không chứa 0.

### Kết luận

1. **Kết luận về M5 không đổi với L = 1, 2 và 4.** LGB-Q cần ít tồn kho hơn LGB-T, LGB-C, TSB-NB và ETS ở cả ba lead time, mọi khoảng tin cậy không chứa 0. Regret lớn nhất của LGB-Q qua sáu lần chạy (2 dataset × 3 lead time) là 1,0% (không tính HGB-Q).
2. **VN1, cửa sổ chính:**
   - LGB-Q, LGB-T và LGB-C không khác nhau có ý nghĩa ở mọi lead time, giống kết quả chính.
   - Lợi thế của LGB-Q so với TSB-NB giảm khi lead time dài: +16% ở L = 1, +10,5% ở L = 2, không còn ý nghĩa ở L = 4.
3. **C2 tốt nhất trên VN1 ở cả ba lead time** (−4 đến −8% so với LGB-Q).
4. Câu Limitations trong bài ICIIT ("only the fill-rate ranking at τ = 0.9 was checked") giờ có thể thay bằng kết quả ở cùng fill rate.
5. **Giới hạn:** chỉ cửa sổ chính. Hai cửa sổ còn lại cần dự báo h = 2 và h = 5, phải huấn luyện lại.
