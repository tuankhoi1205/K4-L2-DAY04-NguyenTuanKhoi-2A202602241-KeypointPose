# Self-QC checklist

Người gán: Nguyễn Tuấn Khôi

Người kiểm: Nguyễn Tuấn Khôi (self-QC, không có peer reviewer)

Ngày: 16/09/2026

Các lệnh đã dùng:

```powershell
python tools/check_pose_labels.py --images dataset/images/train --labels dataset/labels/train
python tools/visualize_pose.py --images dataset/images/train --labels dataset/labels/train --out outputs/vis_train
python tools/visibility_report.py --labels dataset/labels/train --out outputs/visibility_report.json --markdown reports/visibility_report.md
```

| # | Mục kiểm | Trạng thái | Ghi chú / ảnh nào |
| ---: | --- | --- | --- |
| 1 | Mọi người trong ảnh đều có đủ 17 điểm, không ai bị thiếu | ☑ Đạt định dạng | Công cụ đọc 20/20 ảnh, 29 skeleton; mỗi skeleton có đủ cấu trúc 17 điểm. |
| 2 | Bật đường nối: không có xương nào cắt chéo ở vai hoặc hông | ☑ Đã self-QC | Đã xem ảnh overlay trong `outputs/vis_train/`; trái/phải tính theo cơ thể người. |
| 3 | Không có xương nào kéo dài sang một cơ thể khác | ☑ Đã self-QC | Đặc biệt kiểm tra ảnh nhiều người như `train_13`. |
| 4 | Khớp bị che dùng `v=1` và có chấm, không phải `v=0` | ☐ Cần đồng bộ lại export | TXT hiện tại vẫn có các điểm vùng mặt/chi bị che nhưng còn trong ảnh đang là `v=0`, ví dụ `train_06`, `train_09`, `train_13`. |
| 5 | `v=0` chỉ xuất hiện ở khớp thật sự ra ngoài mép ảnh | ☐ Cần đồng bộ lại export | Visibility report hiện tại còn `v=0: 102`; phải bảo đảm các sửa đổi `O -> Q` trong CVAT đã được export và chuyển đổi lại. |
| 6 | Không có dấu hiệu dùng `Hidden` | ☑ Đạt | Quy trình chỉ dùng Visible, Occluded (`Q`) và Outside (`O`). |
| 7 | Export đúng COCO Keypoints 1.0: mảng `keypoints` có 51 số mỗi người | ☑ Đạt định dạng | JSON đã chuyển đổi thành công sang YOLO Pose. |
| 8 | Bản YOLO Pose: mỗi dòng 56 số, `kpt_shape: [17, 3]` | ☑ Đạt định dạng | `check_pose_labels.py` báo 0 lỗi định dạng. |
| 9 | Visibility report đã nộp và hai bảng đã được đặt cạnh nhau | N/A | Đã tạo report của bài cá nhân; không có dữ liệu peer-review để so sánh. |
| 10 | Mọi ca không rõ đều được ghi trong `GUIDELINE_MINI.md` | ☑ Đạt | Đã ghi `train_06`, `train_09`, `train_13`. |
| 11 | `check_pose_labels.py` chạy 0 lỗi | ☑ Đạt | Kết quả gần nhất: đạt định dạng, 0 lỗi; cảnh báo visibility cần đối chiếu bằng mắt. |

## Lỗi/điểm cần xác nhận trước khi khóa nhãn

| Ảnh | Người thứ | Khớp | Hiện trạng trong TXT | Cách xử lý |
| --- | ---: | --- | --- | --- |
| `train_06` | 1 | nose, eyes, ears; right elbow/wrist/knee/ankle | Đang là `v=0` dù nhiều tâm khớp còn trong ảnh | Mở lại CVAT; khớp còn trong ảnh thì đặt điểm ước lượng và `Q`; chỉ giữ `O` nếu tâm thật sự ngoài biên. |
| `train_09` | 1 | nose/right eye/right ear; right elbow; wrists; right knee/ankle | Đang là `v=0`; cần phân biệt bị xe/thân người che với ra ngoài ảnh | Áp dụng `Q` cho điểm còn trong ảnh và `O` cho điểm thật sự vượt biên. |
| `train_13` | 2 | nose, eyes, ears, left elbow/wrist | Đang là `v=0` dù người nhỏ còn nằm trong vùng ảnh | Đặt điểm giải phẫu ước lượng và `Q` nếu tâm khớp còn trong ảnh. |

## Hai câu kết luận

- Lỗi lặp lại nhiều nhất trong bản TXT hiện tại: dùng `Outside (v=0)` cho khớp không nhìn thấy nhưng tâm khớp vẫn còn trong ảnh.
- Đây chủ yếu là lỗi thao tác/phân biệt `Occluded` với `Outside`; guideline đã được làm rõ, nhưng cần export và chuyển đổi lại để TXT phản ánh bản sửa trong CVAT.
