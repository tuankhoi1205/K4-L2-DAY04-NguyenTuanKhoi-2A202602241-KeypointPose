# Visibility report

- Thư mục nhãn: `dataset\labels\train`
- 20 ảnh, 29 skeleton, trung bình 13.48 khớp có v > 0 mỗi người
- Tổng: v=2 346 | v=1 45 | v=0 102

| # | Khớp | v=2 | v=1 | v=0 | %v=1 |
| ---: | --- | ---: | ---: | ---: | ---: |
| 0 | nose | 22 | 0 | 7 | 0% |
| 1 | left_eye | 21 | 1 | 7 | 3% |
| 2 | right_eye | 22 | 0 | 7 | 0% |
| 3 | left_ear | 14 | 7 | 8 | 24% |
| 4 | right_ear | 16 | 5 | 8 | 17% |
| 5 | left_shoulder | 25 | 4 | 0 | 14% |
| 6 | right_shoulder | 28 | 1 | 0 | 3% |
| 7 | left_elbow | 23 | 1 | 5 | 3% |
| 8 | right_elbow | 25 | 1 | 3 | 3% |
| 9 | left_wrist | 20 | 2 | 7 | 7% |
| 10 | right_wrist | 21 | 2 | 6 | 7% |
| 11 | left_hip | 22 | 6 | 1 | 21% |
| 12 | right_hip | 22 | 5 | 2 | 17% |
| 13 | left_knee | 17 | 2 | 10 | 7% |
| 14 | right_knee | 18 | 3 | 8 | 10% |
| 15 | left_ankle | 16 | 1 | 12 | 3% |
| 16 | right_ankle | 14 | 4 | 11 | 14% |

## Đọc bảng này thế nào

1. Khớp nào có **%v=1 cao**: khớp hay bị che. Cổ tay và hông thường là hai vị trí cần xem lại guideline trước khi kết luận.
2. Khớp nào có **v=0 cao bất thường**: mọi người đang dùng Outside ở chỗ đáng lẽ là Occluded. Đó là lỗi số 3 của slide 46, và nó xoá thẳng khớp đó khỏi bảng điểm OKS.
3. Khi so hai người: **lệch lớn = bất đồng về guideline**, không phải về bức ảnh. Sửa guideline trước, sửa nhãn sau.
