# Giai đoạn 4 — Baseline deep learning (N-HiTS)

Ngày 14/10/2026. Phần diễn giải gắn nhãn [Nhận định nhóm].

Nguồn:

- `code/run_neural.py` và `code/member_run_neural.py`: chạy mô hình;
- `code/neural_eval.py` → `code/outputs/neural_eval.md` (bản sao ở `tables/`): chấm điểm.

Dự báo nằm ở `code/outputs/neural/`.

| Phần | Máy | Trạng thái |
|---|---|---|
| VN1, ba cửa sổ | Anh Duy (i5-12450H, 16 GB) | ✅ xong; mỗi cửa sổ 5–6 phút |
| M5, ba cửa sổ | Thành viên B (`07_paper_draft/handoff_neural_baseline.md`) | ⏳ chờ B push |

## Thiết kế

- **Mô hình:** N-HiTS (`neuralforecast` 3.3.0), loss đa phân vị.
- **Mục tiêu:** dự báo trực tiếp phân vị của tổng h tuần (h = 3).
  - Mô hình được huấn luyện trên chuỗi tổng h tuần dịch tới, Z[t] = Y[t] + … + Y[t+h−1].
  - Giá trị cần dự báo là bước thứ h của dự báo trên chuỗi Z.
- **Cấu hình huấn luyện:**
  - huấn luyện lại ở đầu mỗi khối 13 tuần;
  - lịch sử đầu vào 52 tuần;
  - 1.500 bước huấn luyện;
  - mô hình đơn biến (không dùng giá hay lịch);
  - siêu tham số cố định, một seed.

## Kết quả VN1

| | Cửa sổ chính | Cửa sổ thứ hai | Cửa sổ thứ ba |
|---|---|---|---|
| wQL N-HiTS / LGB-Q / TSB-NB | 0,107 / **0,103** / 0,129 | 0,079 / **0,078** / 0,088 | 0,104 / **0,102** / 0,138 |
| Tồn kho N-HiTS so với LGB-Q, fill 0,92 | +8,3% [+3,6; +12,7] | −2,1% [−8,3; +0,4] | +2,6% [−1,8; +10,8] |
| Hạng đường đánh đổi của N-HiTS (8 phương pháp) | 4,4 | 2,8 | 3,8 |
| Regret của N-HiTS | 9,5% | 5,9% | 7,8% |

Regret lớn nhất qua ba cửa sổ VN1, so với phương pháp đơn tốt nhất trong 8:

| C2 | N-HiTS | LGB-T | LGB-Q | HGB-Q | TSB-NB |
|---|---|---|---|---|---|
| **−0,7%** | 9,5% | 9,5% | 10,2% | 12,3% | 20,7% |

Lưu ý: regret của LGB-Q ở cửa sổ thứ hai tăng từ 9,4% lên 10,2%, vì N-HiTS thắng LGB-Q ở một số mức fill rate trong cửa sổ đó, nên mốc "phương pháp đơn tốt nhất" thay đổi.

## Nhận xét (VN1)

1. **Độ chính xác:** N-HiTS đứng thứ hai hoặc thứ ba về wQL, sau LGB-Q, ngang HGB-Q, và tốt hơn TSB-NB rõ rệt. Độ phủ ở các phân vị cao gần mức danh nghĩa (ví dụ 0,99 → 0,986–0,992).
2. **Tồn kho:** N-HiTS không dẫn đầu ở cửa sổ nào; hạng đường đánh đổi 2,8–4,4.
   - Ở cửa sổ thứ hai (nơi TSB-NB thắng), N-HiTS nhỉnh hơn LGB-Q một chút.
   - Ở cửa sổ chính, N-HiTS kém LGB-Q 3,5–8,8%.
3. **Kết luận chính không đổi khi thêm N-HiTS:**
   - LGB-Q vẫn là phương pháp đơn tốt nhất ở cửa sổ chính và cửa sổ thứ ba;
   - C2 vẫn có regret thấp nhất, và vẫn tốt hơn mọi phương pháp đơn kể cả khi tính N-HiTS.
4. [Nhận định nhóm] N-HiTS ở đây là mô hình đơn biến, không tinh chỉnh. Một mô hình neural có dùng biến ngoại sinh và được tinh chỉnh có thể làm tốt hơn. Khi viết bài cần nói rõ đây là một baseline deep learning tiêu chuẩn, không phải giới hạn trên của deep learning.

## Việc tiếp theo

- Khi B push phần M5: `git pull`, rồi chạy `python code/neural_eval.py`. Script tự nhận các file mới và cập nhật bảng.
