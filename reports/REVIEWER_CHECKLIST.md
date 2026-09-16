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
| 4 | Khớp bị che dùng `v=1` và có chấm, không phải `v=0` | ☑ Đạt sau rework | Đã sửa trong CVAT, export lại và đánh giá với gold; lỗi `xoa_khop_bi_che` giảm từ 13 xuống 0. |
| 5 | `v=0` chỉ xuất hiện ở khớp thật sự ra ngoài mép ảnh | ☑ Đạt sau rework | Visibility report cuối: `v=2: 355`, `v=1: 109`, `v=0: 29`; các ca còn lại đã được kiểm tra bằng mắt. |
| 6 | Không có dấu hiệu dùng `Hidden` | ☑ Đạt | Quy trình chỉ dùng Visible, Occluded (`Q`) và Outside (`O`). |
| 7 | Export đúng COCO Keypoints 1.0: mảng `keypoints` có 51 số mỗi người | ☑ Đạt định dạng | JSON đã chuyển đổi thành công sang YOLO Pose. |
| 8 | Bản YOLO Pose: mỗi dòng 56 số, `kpt_shape: [17, 3]` | ☑ Đạt định dạng | `check_pose_labels.py` báo 0 lỗi định dạng. |
| 9 | Visibility report đã nộp và hai bảng đã được đặt cạnh nhau | N/A | Đã tạo report của bài cá nhân; không có dữ liệu peer-review để so sánh. |
| 10 | Mọi ca không rõ đều được ghi trong `GUIDELINE_MINI.md` | ☑ Đạt | Đã ghi `train_06`, `train_09`, `train_13`. |
| 11 | `check_pose_labels.py` chạy 0 lỗi | ☑ Đạt | Kết quả cuối: đạt định dạng, 0 lỗi; cảnh báo heuristic đã được đối chiếu bằng mắt. |

## Lỗi đã xử lý trong rework

| Ảnh | Người thứ | Khớp | Lỗi trước rework | Cách đã xử lý |
| --- | ---: | --- | --- | --- |
| `train_15` | 1 | `right_ear`, `right_wrist`, `left_knee` | Thiếu khớp nhìn thấy và xoá khớp bị che | Bổ sung tai; đặt lại cổ tay và gối ở vị trí ước lượng với `v=1`. |
| `train_06` | 1 | `left_ear`, `right_ear` | Thiếu hai tai dù đầu còn trong ảnh | Đặt lại hai tai trong CVAT theo phần đầu còn quan sát được. |
| `train_16` | 2 | `nose`, `right_eye`, `right_ear` | Nose bị xoá; mắt và tai phải bị thiếu | Đặt nose với `v=1`, bổ sung mắt và tai phải rồi export lại. |

## Hai câu kết luận

- Lỗi lặp lại nhiều nhất trước rework: dùng `Outside (v=0)` cho khớp bị che nhưng tâm khớp vẫn còn trong ảnh.
- Đây chủ yếu là lỗi thao tác/phân biệt `Occluded` với `Outside`; sau rework, OKS trung bình đạt `0.9417`, OKS@0.75 đạt `1.000` và không còn skeleton cần sửa.
