# Giai đoạn 2 — B1 (đồng bộ) và B2 (khác biệt giữa M5 và VN1)

Ngày 13/10/2026. Quy trình: `07_paper_draft/phase2_workflow.md`.

Nguồn số liệu:

- `code/outputs/sync_check.md` (script `code/sync_check.py`);
- `code/outputs/dataset_shift.md` (script `code/dataset_shift.py`);
- bản sao trong `tables/`, hình trong `figures/fig_dataset_shift.png`.

Mọi đặc trưng theo chuỗi chỉ tính từ lịch sử **trước** giai đoạn kiểm thử, trên các chuỗi được đánh giá.

## B1 — Hai bộ dữ liệu đã đồng bộ chưa?

**Kết luận: đã đồng bộ ở mọi điểm ảnh hưởng tới so sánh.** Các khác biệt còn lại thuộc về bản chất dữ liệu.

| Tiêu chí | M5 | VN1 | Đánh giá |
|---|---|---|---|
| Độ dài tuần | 7 ngày | 7 ngày | Giống |
| Ngày đầu tuần | Thứ Bảy | Thứ Hai | Khác, không ảnh hưởng (hai bảng không ghép theo thời gian) |
| Thiếu doanh số sau điểm bắt đầu | 0 | 0 | Giống |
| Doanh số âm | 0 | 0 | Giống |
| Điều kiện chọn chuỗi; chia tập; mốc huấn luyện lại | Cùng một đoạn code | Cùng | Giống |
| Định nghĩa điểm bắt đầu chuỗi | Tuần đầu có giá (lên kệ) | Tuần bán đầu tiên | Khác về định nghĩa, **không khác về kết quả** (xem dưới) |
| Giá quan sát được sau điểm bắt đầu | 100% số ô | 44,6% số ô | Khác; giá VN1 được điền tiếp từ tuần trước. Kiểm tra bằng thử nghiệm bỏ đặc trưng giá (B0) |
| Đặc trưng của mô hình ML | 27 | 21 | 21 đặc trưng của VN1 đều có ở M5; M5 thêm 6 (sự kiện, SNAP, 4 thuộc tính) |
| Đơn vị chuỗi | sản phẩm × cửa hàng | nhà bán × kho × sản phẩm | Khác về bản chất; đã nêu trong bài |

**Về định nghĩa điểm bắt đầu chuỗi:**

- 99,5% chuỗi M5 có tuần lên kệ trùng với tuần bán đầu tiên.
- Tính lại M5 theo định nghĩa của VN1: tỷ lệ tuần bằng 0 giữ nguyên (24,4%), tỷ lệ các nhóm ADI–CV² đổi tối đa 0,1 điểm %, số chuỗi được đánh giá đổi 2–5 chuỗi.
- Điều này đúng ở cả ba cửa sổ.
- Vì vậy **không cần chạy thêm phiên bản M5 nào**.

**Lỗi phát hiện trong bài:** `final_paper_revA.docx` mục 3.4 ghi "M5 27, VN1 and VNF 21, 19 shared". Con số đúng là **21 đặc trưng chung**.

## B2 — Hai bộ dữ liệu khác nhau ở đâu?

### Mức khác biệt tổng thể

Một bộ phân loại "chuỗi này thuộc M5 hay VN1?" (5-fold, mẫu cân bằng):

| Đặc trưng dùng | AUC cửa sổ chính | Cửa sổ thứ hai | Cửa sổ thứ ba |
|---|---|---|---|
| Chỉ ADI và CV² | 0,865 | 0,866 | 0,855 |
| 9 đặc trưng chung (không phụ thuộc độ dài lịch sử hay giá) | 0,913 | 0,908 | 0,893 |
| Tất cả (thêm độ dài lịch sử, giá, mùa vụ) | 0,994 | 0,993 | 0,992 |

- AUC = 0,5 nghĩa là hai bộ giống nhau; AUC = 1 nghĩa là tách biệt hoàn toàn.
- Hai bộ dữ liệu **khác nhau rõ rệt** ngay cả khi chỉ nhìn hành vi nhu cầu, và mức khác biệt ổn định qua ba cửa sổ.
- ADI và CV² đã bắt được phần lớn khác biệt; các đặc trưng còn lại thêm khoảng 0,04–0,05 AUC.

### Khác ở đặc trưng nào (cửa sổ chính)

KS = khoảng cách Kolmogorov–Smirnov (0 = cùng phân phối, 1 = tách rời).

| Đặc trưng | Trung vị M5 [Q25–Q75] | Trung vị VN1 [Q25–Q75] | KS | Diễn giải |
|---|---|---|---|---|
| Độ dài lịch sử (tuần) | 230 [143–251] | 103 [50–160] | 0,68 | Chuỗi VN1 ngắn hơn một nửa |
| Biến động giá (CV) | 0,023 [0–0,049] | 0,082 [0,027–0,156] | 0,45 | Giá VN1 dao động mạnh hơn |
| ADI | 1,25 [1,08–1,59] | 2,18 [1,16–6,80] | 0,40 | VN1 rời rạc hơn nhiều; một phần tư số chuỗi bán chưa tới 1 tuần trong 7 |
| Tỷ lệ tuần bằng 0 | 19,8% [7,7–37,1] | 54,2% [14,1–85,3] | 0,40 | Như trên |
| Mức bán (log) | 1,65 | 0,89 | 0,34 | VN1 bán ít hơn |
| Số tuần từ lần bán gần nhất | 0 [0–0] | 1 [0–15] | 0,33 | Một phần tư chuỗi VN1 không bán ≥ 15 tuần trước kiểm thử |
| CV² cỡ đơn | 0,32 [0,24–0,41] | 0,34 [0,12–0,63] | 0,23 | Trung vị gần bằng; VN1 phân tán rộng hơn hẳn |
| Xu hướng 26 tuần (log) | 0,07 [−0,21–0,37] | −0,04 [−0,63–0,25] | 0,22 | VN1 biến động xu hướng mạnh hơn, lệch về giảm |
| Đỉnh bán so với cỡ đơn (52 tuần) | 0,84 [0,69–1,00] | 0,85 [0,47–1,16] | 0,21 | Trung vị bằng nhau; VN1 phân tán hơn |
| Mùa vụ (tự tương quan trễ 52) | 0,04 | 0,06 | 0,11 | Gần như giống; cả hai yếu |

Trong 9 đặc trưng chung, các đặc trưng đóng góp nhiều nhất vào khả năng phân biệt (độ quan trọng hoán vị): ADI (0,099), CV² (0,079), xu hướng 26 tuần (0,046), cỡ đơn (0,034), đỉnh bán (0,025).

**Ghi chú phương pháp:** đặc trưng "tỷ trọng tuần bán lớn nhất trên toàn lịch sử" (KS 0,62) bị loại khỏi bộ đặc trưng chung. Lý do: nó giảm một cách cơ học khi lịch sử dài hơn, nên không so sánh được giữa hai bộ dữ liệu. Nó được thay bằng "đỉnh bán so với cỡ đơn trong 52 tuần gần nhất".

### Hình

![PCA](figures/fig_dataset_shift.png)

Trên hai thành phần chính đầu tiên (71% phương sai):

- M5 tập trung trong một vùng (chuỗi bán đều, ADI thấp).
- VN1 phủ vùng đó **và** kéo dài sang vùng ADI cao / lâu không bán, nơi M5 gần như không có chuỗi nào.

### Ý nghĩa cho các bước sau

[Nhận định nhóm]

1. Vùng nhu cầu của M5 gần như là **tập con** của VN1. Kết luận rút từ M5 có thể chuyển sang phần "giống M5" của VN1, nhưng không chắc chuyển sang phần đuôi.
2. Ba đặc trưng mà ADI–CV² **không** chứa — xu hướng, độ gần của lần bán cuối, đỉnh bán — khác nhau rõ giữa hai bộ. Phân tích dịch chuyển nhu cầu (`code/outputs/demand_shift.md`) đã cho thấy chuỗi "tăng mạnh" và "ổn định" có phương pháp tốt nhất khác nhau. Đây là cơ sở để thử phân cụm trên bộ đặc trưng rộng hơn ở B3.
3. Bước B3 dùng 9 đặc trưng chung. Không dùng độ dài lịch sử và giá, vì hai đặc trưng này chủ yếu nhận diện "đây là dataset nào" chứ không mô tả hành vi nhu cầu.

## Trạng thái

- [x] B1 Đồng bộ
- [x] B2 Khác biệt hai bộ dữ liệu
- [ ] B0 Thử nghiệm bỏ đặc trưng giá trên VN1 — đang chạy lại (`code/outputs/logs/rerun_nop_driver.log`)
