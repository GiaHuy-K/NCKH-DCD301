# Bàn giao: chạy baseline deep learning (N-HiTS)

Ngày 14/10/2026. Người giao: Anh Duy. Người nhận: một thành viên khác của nhóm SE1926_G01.

## 1. Bối cảnh

Bài báo so sánh 7 phương pháp dự báo xác suất bằng hiệu quả tồn kho trên hai bộ dữ liệu bán lẻ (M5, VN1). Mỗi bộ có ba cửa sổ kiểm thử:

- `main`: offset 0;
- `second`: offset 26;
- `third`: offset 52.

Reviewer và bài Theodorou et al. (2025) đều yêu cầu có một mô hình deep learning làm baseline. Script `code/run_neural.py` chạy **N-HiTS** (thư viện `neuralforecast`) theo **đúng giao thức** của các phương pháp khác:

- dự báo phân vị (0,5 / 0,8 / 0,9 / 0,95 / 0,99) của **tổng nhu cầu h tuần**;
- dự báo tại mỗi tuần trong 26 tuần kiểm thử;
- huấn luyện lại ở đầu mỗi khối 13 tuần.

Kết quả là một file `.npz` theo định dạng cache sẵn có. Máy của Anh Duy sẽ dùng file đó để chấm điểm cùng 7 phương pháp kia.

**Không sửa script, siêu tham số hay dữ liệu.** Hai máy phải chạy giống hệt nhau thì kết quả mới gộp được.

## 2. Phân công

| Người | Dataset | Lệnh | Số lần chạy |
|---|---|---|---|
| **Thành viên B** | **M5** (30.490 chuỗi, nặng hơn) | `--dataset M5 --offset 0`, `26`, `52`, với `--h 3` | 3 |
| Anh Duy | VN1 (15.053 chuỗi) | `--dataset VN1 --offset 0`, `26`, `52`, với `--h 3` | 3 |

Thành viên B **chỉ cần dữ liệu M5**, không cần VN1.

Nếu máy B yếu (ít hơn 16 GB RAM), có thể đổi lại: B chạy VN1, Anh Duy chạy M5. Báo nhau trước khi đổi.

## 3. Chuẩn bị (khoảng 30–60 phút)

### 3.1. Lấy code

```bash
git clone https://github.com/NCKH-LongT/NCKH-DCD301.git
cd NCKH-DCD301
git checkout FA26_BDT301_SE1926_G01
```

Nếu đã có repo thì chỉ cần `git checkout FA26_BDT301_SE1926_G01` rồi `git pull`. Kiểm tra file `code/run_neural.py` có tồn tại; nếu không có, Anh Duy chưa push, hãy báo Anh Duy.

### 3.2. Lấy dữ liệu M5

1. Tải từ Kaggle, cuộc thi *M5 Forecasting – Accuracy*. Cần đăng nhập Kaggle và bấm chấp nhận luật cuộc thi.
2. Giải nén vào `data/m5-forecasting-accuracy/`. Cần đúng ba file sau (kiểm tra kích thước):

| File | Kích thước (byte) |
|---|---|
| `calendar.csv` | 103.469 |
| `sales_train_evaluation.csv` | 121.736.518 |
| `sell_prices.csv` | 203.395.785 |

**Lưu ý:** `sales_train_evaluation.csv` khác `sales_train_validation.csv`. Phải dùng bản *evaluation*.

### 3.3. Môi trường Python (3.12)

Đặt môi trường ảo ở ổ còn trống: cần khoảng 3 GB cho torch, ray và các thư viện kèm theo.

```bash
python -m venv D:/venvs/nf
D:/venvs/nf/Scripts/python.exe -m pip install numpy pandas neuralforecast==3.3.0
```

Máy của Anh Duy dùng neuralforecast 3.3.0 với torch 2.12.0 (bản CPU).

## 4. Chạy

Mọi lệnh chạy từ thư mục gốc của repo.

### Bước 1 — Chạy thử (vài phút)

```bash
D:/venvs/nf/Scripts/python.exe code/run_neural.py --dataset M5 --offset 0 --h 3 --quick
```

Dòng `fingerprint` phải **khớp đúng** với bảng dưới. Nếu không khớp, dữ liệu sai: dừng lại và kiểm tra mục 3.2.

| Lần chạy | Dòng fingerprint đúng |
|---|---|
| M5, offset 0 | `fingerprint M5 w0: 30490 series x 277 weeks, evaluated 30381, total sales of evaluated series 66793704` |
| M5, offset 26 | `fingerprint M5 w26: 30490 series x 251 weeks, evaluated 29917, total sales of evaluated series 59390858` |
| M5, offset 52 | `fingerprint M5 w52: 30490 series x 225 weeks, evaluated 28824, total sales of evaluated series 52145075` |
| VN1, offset 0 | `fingerprint VN1 w0: 15053 series x 196 weeks, evaluated 13844, total sales of evaluated series 39234029` |
| VN1, offset 26 | `fingerprint VN1 w26: 15053 series x 170 weeks, evaluated 11442, total sales of evaluated series 31620787` |
| VN1, offset 52 | `fingerprint VN1 w52: 15053 series x 144 weeks, evaluated 9383, total sales of evaluated series 24659310` |

Kết quả chạy thử đạt yêu cầu khi có dòng `done ...; series with missing forecasts: 0.00%`. Bản chạy thử chỉ huấn luyện 50 bước nên độ phủ phân vị còn thấp; điều này bình thường.

Lần đầu chạy M5 sẽ tạo `data/cache/panel_m5.npz`, mất vài phút. Các lần sau dùng lại file này.

### Bước 2 — Chạy thật

Ba lệnh, chạy lần lượt. Mỗi lệnh ghi log riêng.

```bash
D:/venvs/nf/Scripts/python.exe -u code/run_neural.py --dataset M5 --offset 0  --h 3 > nhits_M5_w0.log 2>&1
D:/venvs/nf/Scripts/python.exe -u code/run_neural.py --dataset M5 --offset 26 --h 3 > nhits_M5_w26.log 2>&1
D:/venvs/nf/Scripts/python.exe -u code/run_neural.py --dataset M5 --offset 52 --h 3 > nhits_M5_w52.log 2>&1
```

- **Thời gian:** chưa đo trên M5. Chạy thử 300 chuỗi VN1 với 50 bước mất khoảng 10 giây; bản thật có 1.500 bước và khoảng 30.000 chuỗi. Hãy ghi lại thời gian của lệnh đầu tiên (log có in số giây của từng khối) rồi báo cho Anh Duy.
- **Không được để máy ngủ hoặc tắt** trong khi chạy.

Mỗi lệnh xong khi log có dòng `written ...nhits_h3.npz` và `series with missing forecasts: 0.00%`.

## 5. Bàn giao kết quả

Gửi cho Anh Duy (Google Drive hoặc Zalo):

1. Ba file dự báo:
   - `data/cache/forecasts/M5/nhits_h3.npz`
   - `data/cache/forecasts/M5_w26/nhits_h3.npz`
   - `data/cache/forecasts/M5_w52/nhits_h3.npz`

   Mỗi file khoảng 10–20 MB. **Đổi tên khi gửi** thành `M5_nhits_h3.npz`, `M5_w26_nhits_h3.npz`, `M5_w52_nhits_h3.npz` để khỏi lẫn.
2. Ba file log `nhits_M5_w*.log`.
3. Thông tin máy: CPU, RAM, và thời gian mỗi lần chạy.

**Không commit hoặc push** các file `.npz`, log hay thư mục `data/` lên git. Anh Duy sẽ chấm điểm và commit kết quả.

## 6. Nếu gặp lỗi

| Lỗi | Cách xử lý |
|---|---|
| Fingerprint không khớp | Sai file dữ liệu (thường là dùng nhầm `sales_train_validation.csv`) |
| `MemoryError` hoặc máy treo | Đóng các ứng dụng khác. Nếu vẫn lỗi, báo Anh Duy để đổi phân công. **Không** tự giảm `batch_size` hay số chuỗi |
| `cannot find the column of quantile` | Sai phiên bản neuralforecast; cài đúng 3.3.0 |
| Lỗi khác | Gửi nguyên log cho Anh Duy. Không tự sửa `run_neural.py` |

## 7. Prompt cho AI của thành viên B

Dán nguyên đoạn sau vào trợ lý AI (Claude Code, Cursor…) đang mở trong thư mục repo:

```text
Tôi là thành viên nhóm nghiên cứu SE1926_G01. Nhiệm vụ: chạy baseline deep learning N-HiTS trên bộ dữ liệu M5
theo đúng hướng dẫn trong file 07_paper_draft/handoff_neural_baseline.md của repo này. Hãy đọc toàn bộ file đó
trước, rồi làm lần lượt các mục 3, 4 và 5.

Quy tắc bắt buộc:
- KHÔNG sửa code/run_neural.py, các file trong code/f2d/, siêu tham số hay dữ liệu. Nếu script lỗi, dừng lại và
  báo lỗi cho tôi kèm log; không tự sửa script.
- Trước khi chạy thật, phải chạy lệnh --quick và so dòng "fingerprint" với bảng ở mục 4; nếu không khớp từng con
  số thì dừng lại.
- Đặt môi trường ảo và cache pip ở ổ còn trống (ít nhất 3 GB). Hỏi tôi trước khi tải hoặc cài bất cứ thứ gì.
- Chạy ba lệnh ở mục 4, bước 2 lần lượt, mỗi lệnh ghi log riêng. Lệnh chạy lâu thì chạy nền và theo dõi log; báo
  cho tôi thời gian của từng khối.
- KHÔNG commit, KHÔNG push. Không đưa file .npz, log hay thư mục data/ lên git.
- Khi xong, kiểm tra mỗi log có "series with missing forecasts: 0.00%" và "written", rồi liệt kê đường dẫn ba
  file .npz cần gửi cho Anh Duy (mục 5).
```

## 8. Việc của Anh Duy sau khi nhận file

1. Chép ba file vào `data/cache/forecasts/M5{,_w26,_w52}/nhits_h3.npz` trên máy này.
2. Chạy VN1 ba cửa sổ trên máy này.
3. Chấm điểm:

   ```bash
   python code/equal_fill_ci.py --models empirical,tsb_nb,ets,lgb_tweedie,lgb_conformal,hgb_quantile,lgb_quantile,nhits --out-suffix nn
   ```

   Chạy thêm với `--offset 26` và `--offset 52`. Sau đó tính độ chính xác (wQL) và regret có N-HiTS.
4. Ghi kết quả vào `06_experiment_results/` (bản tạp chí).
