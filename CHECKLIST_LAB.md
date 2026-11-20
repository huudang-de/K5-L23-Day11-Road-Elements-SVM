# Checklist công việc · Ngày 11: Road Elements & SVM

> Tổng hợp theo thứ tự thời gian từ [README.md](README.md), [CHECKPOINTS.md](CHECKPOINTS.md), [SUBMISSION.md](SUBMISSION.md), [docs/guideline_road_v1.0.md](docs/guideline_road_v1.0.md) và [docs/qc_severity_road_v1.0.md](docs/qc_severity_road_v1.0.md).

---

## 1. M0 (13:00) — Kiểm tra môi trường & chuẩn bị

- [x] Cập nhật repo: `git pull`
- [x] Kiểm tra thư mục `data/` có đủ 28 ảnh (`A01–A16`, `B01–B12`)
- [x] Chạy kiểm tra hệ thống: `PYTHONPATH=src python3 -m road_lab.road_check selftest` (kết quả phải in `SELFTEST PASS`)
- [x] Đăng nhập CVAT của lớp (tuyệt đối **không** đưa ảnh lên `app.cvat.ai`) và mở được project `d11-road-<tên>`
- [x] Đọc nắm bảng label và quy tắc làn trong [docs/guideline_road_v1.0.md](docs/guideline_road_v1.0.md)

---

## 2. Task 0 (13:15) — Kiểm tra Project & Taxonomy trên CVAT

- [x] Chạy `PYTHONPATH=src python3 -m road_lab.taxonomy` (hoặc đối chiếu [configs/road_v1_labels.json](configs/road_v1_labels.json)) để kiểm tra khớp với project trên CVAT:
  - [x] Đủ **6 label hình học**: `lane`, `road_curb`, `crosswalk`, `drivable`, `traffic_light`, `traffic_sign` (đúng kiểu hình & thuộc tính)
  - [x] Đủ **4 label Tag** (mỗi tag có 1 thuộc tính select `value`): `camera`, `timeofday`, `weather`, `source`
- [x] Kiểm tra đủ **4 task**: `front`, `front-extra`, `fisheye`, `fisheye-extra`

---

## 3. Task 1 (13:50) — Gán nhãn Set A (Camera trước BDD100K)

- [ ] Gán nhãn bắt buộc **12 ảnh `A01–A12`** (nếu xong sớm, làm thêm **`A13–A16`** để lấy điểm thưởng +0,5 điểm/ảnh)
- [ ] **Thứ tự thao tác trên mỗi ảnh**:
  1. [ ] Gắn đủ **4 tag ảnh** (`camera = front` trước tiên, sau đó `timeofday`, `weather`, `source`) và kiểm lại giá trị
  2. [ ] Vẽ `lane` (`R1`, `L1`, rồi `L2`, `R2` nếu thấy) và `road_curb` (nếu mép đường không sơn):
      - Đặt điểm trên **tâm vệt sơn** (lệch ≤ 5 px), vẽ từ **gần xe ra xa**
      - Vạch đứt vẽ thành **1 polyline** liên tục
      - Đoạn bị che ngắn: nối liền qua chỗ che và bật cờ `occluded`
      - Dừng ở điểm xa nhất còn thấy vạch (không kéo tới chân trời)
  3. [ ] Mở Details sửa lại **thuộc tính** từng vạch (`position`, `style`, `color`, `kind`), tránh để nguyên mặc định `L1 / solid / white / single` của CVAT
  4. [ ] Vẽ `drivable` (`direct` trước — không gồm dải đỗ xe sát lề, thân xe, vỉa hè, phần sau cọc; rồi `alternative` nếu có) và `crosswalk` (polygon)
  5. [ ] Vẽ `traffic_light` và `traffic_sign` (chiều cao ≥ 8 px, box sát vỏ/mép, không gán hình in trên xe/phản chiếu; biển phụ tách hộp riêng `group = supplementary`)
  6. [ ] Nhấn `Ctrl+S` lưu lại
- [ ] **Mốc kiểm tra Task 1**:
  - [ ] **Ảnh đầu tiên (`A01`)**: Gọi coach kiểm tra ngay để tránh lặp lỗi cho 11 ảnh sau
  - [ ] **Giữa giờ (~14:20, xong ~6 ảnh)**: Lưu ý `A03` có `occluded`, `A04` có vạch vàng đôi (`kind = double`) và `drivable` chia `direct`/`alternative`, `A07–A08` box đèn sát vỏ và đúng màu đang sáng

---

## 4. Task 2 (15:05) — Gán nhãn Set B (Fisheye WoodScape) & Self-QC

- [ ] Gán nhãn bắt buộc **8 ảnh `B01–B08`** (nếu xong sớm, làm thêm **`B09–B12`** để lấy điểm thưởng)
- [ ] **Quy tắc riêng cho ảnh Fisheye**:
  - [ ] Gắn tag `camera` đúng theo hậu tố tên ảnh: `FV` → `fisheye_front`, `RV` → `fisheye_rear`, `MVL` → `fisheye_left`, `MVR` → `fisheye_right`
  - [ ] Vẽ polyline **theo cung cong** trên ảnh (lệch ≤ 5 px), đặt nhiều điểm ở vòng ngoài
  - [ ] **Không** gán thân xe mình và vùng tối viền ảnh
  - [ ] Ở camera hai bên (`B05–B08`: `fisheye_left`, `fisheye_right`), mọi `lane` đều đặt `position = other`
  - [ ] Đèn/biển có tâm hộp nằm ngoài 80% bán kính vòng ảnh: tick chọn `edge_zone`
  - [ ] Vạch ngựa vạn gán `crosswalk` (polygon), không gán `lane`
- [ ] **Mốc kiểm tra Task 2**: Coach kiểm tra ảnh fisheye đầu tiên (`B01`) và nhóm camera hai bên (`B05–B08`)
- [ ] **5 phút cuối (Self-QC)**: Rà soát lại toàn bộ ảnh đã gán (đủ 4 tag, đủ L1/R1, chiều vẽ gần→xa, `occluded`, thuộc tính, fisheye)

---

## 5. Task 3 (15:40) — Kiểm chéo lô của bạn khác (Phiếu QC 1)

- [ ] Đổi lô với bạn được ghép và tạo file `<tên>-qc1-for-<tên bạn được kiểm>.md` theo mẫu [templates/qc1_form.md](templates/qc1_form.md)
- [ ] Kiểm đủ **12 mã** (`QC1-01` đến `QC1-12`), ghi rõ số mục đã kiểm (`shape + tag`) cho từng mã
- [ ] Với mỗi lỗi phát hiện: ghi đủ **mã lỗi, mã ảnh, nhãn, mức** (`nặng` / `chính` / `nhẹ`)
- [ ] Tính **tỉ lệ lỗi chính** = `(số lỗi chính) / (tổng số nhãn đã kiểm)` (không chia cho số ảnh)
- [ ] Ra quyết định đúng luật:
  - [ ] Có ≥ 1 lỗi **nặng** → **Rework**
  - [ ] Không có lỗi nặng nhưng tỉ lệ lỗi **chính** > 5% → **Rework**
  - [ ] Còn ca mơ hồ ảnh hưởng kết luận mà lead chưa trả lời → **Hold**
  - [ ] Các trường hợp còn lại → **Release**

---

## 6. Export (16:15) — Xuất file & Tự kiểm định dạng

- [ ] Đưa mọi job trên CVAT về trạng thái **completed**
- [ ] Export định dạng **CVAT for images 1.1** (**không kèm ảnh**) thành 2 file:
  - [ ] `<tên>-front.zip` (gồm task `front` và `front-extra` nếu có)
  - [ ] `<tên>-fisheye.zip` (gồm task `fisheye` và `fisheye-extra` nếu có)
- [ ] Chạy lệnh lint và tạo manifest lần 1:
  ```bash
  PYTHONPATH=src python3 -m road_lab.road_check lint <tên>-front.zip <tên>-fisheye.zip --manifest <tên>-manifest.json --name <tên> --revision 1
  ```
- [ ] Đảm bảo `lint` **không còn lỗi mức nặng** (nếu ảnh nào thật sự không thấy vạch `L1/R1`, không tự vẽ thêm — gọi coach xác nhận và ghi mã ảnh + lý do vào phiếu QC 1)
- [ ] Kiểm tra trong `<tên>-manifest.json` trường `contains_images` là `false`

---

## 7. Buffer (16:35) — Sửa lỗi sau Rework (nếu có) & Hoạt động tùy chọn

- [ ] Nếu nhận phiếu QC 1 yêu cầu sửa:
  - [ ] Sao lưu 2 file zip cũ và manifest cũ vào thư mục `r1/` trên máy (đổi tên manifest cũ thành `<tên>-manifest-r1.json`)
  - [ ] Sửa lỗi trên CVAT, export lại 2 file zip **cùng tên**
  - [ ] Chạy lại `lint --manifest` với `--revision 2`
- [ ] *(Tùy chọn, không tính điểm)* Thử hoạt động row-anchor:
  ```bash
  PYTHONPATH=src python3 -m road_lab.rowanchor <tên>-front.zip --image A01
  ```

---

## 8. Hoàn thiện báo cáo & Nộp bài (Hạn 16:35)

- [ ] Viết báo cáo `<tên>-report.md` theo mẫu [templates/report.md](templates/report.md):
  - [ ] Ảnh chụp màn hình & bảng kiểm file (mục 1)
  - [ ] Bảng các ca khó kèm lý do lựa chọn
  - [ ] Mục 4: Phản hồi phiếu QC 1 nhận được và ghi rõ đã sửa những gì sau buffer
  - [ ] Tự đánh giá nối lỗi gán nhãn với mô hình
- [ ] Nộp đủ **5 file** lên kênh coach chỉ định (không gửi GT, không gửi ảnh, không đăng lên kênh chung của lớp):
  1. [ ] `<tên>-front.zip`
  2. [ ] `<tên>-fisheye.zip`
  3. [ ] `<tên>-manifest.json`
  4. [ ] `<tên>-qc1-for-<tên bạn được kiểm>.md`
  5. [ ] `<tên>-report.md`
