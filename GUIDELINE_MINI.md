# Mini guideline - nhóm: Cá nhân | người gán: Nguyễn Tuấn Khôi | ngày: 16/09/2026

> File này ghi lại các quyết định đã dùng khi gán nhãn và self-QC bộ 20 ảnh train.

## 1. Luật bắt buộc (đã thống nhất cả lớp - không sửa)

- Bộ 17 điểm COCO, đúng tên, đúng thứ tự. Lấy từ file `.SVG` chung.
- Mọi người trong ảnh đều có **đủ 17 điểm**. Điểm không dùng được thì gắn cờ, không xoá.
- Trái/phải tính theo **cơ thể người**, không theo bức ảnh.
- Bị che, còn trong khung -> `v = 1`, **vẫn đặt chấm** ở vị trí ước lượng.
- Ra ngoài mép ảnh -> `v = 0`, **không** đặt chấm.
- Không dùng `Hidden` (`h`) - nó không được lưu vào file.

## 2. Luật áp dụng khi gán nhãn

| Tình huống | Luật đã chọn | Vì sao | Ảnh QC |
| --- | --- | --- | --- |
| Hông của người mặc quần áo dài | Nếu tâm hông bị quần áo che nhưng vẫn nằm trong ảnh, ước lượng theo trục vai, thân và chân; đặt điểm rồi dùng `v=1`. Chỉ dùng `v=2` khi xác định được tâm khớp trực tiếp. | Quần áo chỉ gây che khuất, không làm khớp ra khỏi ảnh. | [train_13](outputs/vis_train/train_13.jpg) |
| Tai bị tóc hoặc mũ bảo hiểm che một phần | Nếu còn thấy rõ tâm tai thì `v=2`; nếu tâm tai bị che thì ước lượng và dùng `v=1`; không dùng `v=0` khi đầu vẫn trong ảnh. | Visibility mô tả khả năng quan sát tâm khớp. | [train_06](outputs/vis_train/train_06.jpg) |
| Người bị cắt ở mép ảnh, chỉ thấy từ hông trở lên | Khớp có tâm ngoài biên ảnh dùng `v=0`; khớp còn trong ảnh nhưng bị vật khác che dùng `v=1`. Không kéo điểm vào sát mép để thay cho Outside. | `Outside` chỉ phụ thuộc biên ảnh. | [train_13](outputs/vis_train/train_13.jpg) |
| Cổ tay nằm sau tay lái hoặc sau thân mình | Ước lượng từ khuỷu tay và hướng cẳng tay, đặt điểm rồi dùng `v=1`. | Cổ tay còn trong khung nhưng bị che. | [train_09](outputs/vis_train/train_09.jpg) |
| Hai người chồng lên nhau | Tạo một skeleton riêng cho mỗi người; nối khớp đúng cơ thể. Khớp của người phía sau bị che dùng `v=1`, không lấy khớp của người phía trước. | Tránh ghép xương sang người khác và giữ đúng danh tính skeleton. | [train_13](outputs/vis_train/train_13.jpg) |
| Người nhỏ đến mức nào thì không gán nữa | Không đặt ngưỡng pixel tùy ý. Mọi người còn nhận diện được trong phạm vi bài đều được gán đủ 17 điểm; chỉ bỏ đối tượng không thể xác nhận là người. | Giữ nhất quán với luật “mọi người trong ảnh đều có đủ 17 điểm”. | [train_13](outputs/vis_train/train_13.jpg) |

> Các liên kết trên là ảnh overlay dùng cho self-QC. Nếu giảng viên yêu cầu đúng screenshot giao diện CVAT, thay từng ảnh bằng screenshot CVAT tương ứng trước khi nộp.

## 3. Ba ca mơ hồ đã gặp

### Ca 1 - ảnh `train_06`, người thứ `1`, khớp vùng mặt dưới mũ bảo hiểm

- Mơ hồ ở chỗ nào: đầu còn trong ảnh nhưng mũ bảo hiểm và góc quay làm nose, eyes, ears khó quan sát trực tiếp.
- Quyết định: khớp còn trong ảnh phải được ước lượng theo cấu trúc đầu và gắn `v=1`; không dùng `v=0` chỉ vì không nhìn thấy.
- Vì sao: `v=0` chỉ dành cho tâm khớp nằm ngoài biên ảnh.
- Nếu quyết ngược lại: model sẽ học rằng khớp bị che là khớp không tồn tại và mất tín hiệu OKS ở vùng đầu.

### Ca 2 - ảnh `train_09`, người thứ `1`, khớp `right_wrist`

- Mơ hồ ở chỗ nào: cổ tay bị thân người hoặc xe máy che, khó xác định chính xác bằng bề mặt nhìn thấy.
- Quyết định: suy ra từ khuỷu tay và hướng cẳng tay, đặt điểm rồi dùng `v=1`.
- Vì sao: tâm cổ tay vẫn thuộc vùng ảnh, chỉ bị che khuất.
- Nếu quyết ngược lại: model dễ bỏ mất cổ tay khi người tương tác với xe hoặc đồ vật.

### Ca 3 - ảnh `train_13`, người thứ `2`, khớp vùng mặt

- Mơ hồ ở chỗ nào: người nhỏ ở mép trái, các chi tiết nose, eyes, ears khó nhìn rõ nhưng đầu vẫn nằm trong ảnh.
- Quyết định: đặt các điểm có thể suy ra theo cấu trúc khuôn mặt và dùng `v=1` khi tâm khớp không quan sát trực tiếp; chỉ dùng `v=0` nếu tâm khớp thật sự vượt biên.
- Vì sao: kích thước nhỏ hoặc thiếu chi tiết không đồng nghĩa với Outside.
- Nếu quyết ngược lại: model sẽ học thiếu keypoint khuôn mặt ở người nhỏ và người gần mép ảnh.

## 4. So sánh visibility report

- Không thực hiện so sánh với bạn cùng nhóm vì bài được làm và self-QC cá nhân.
- Khớp lệch `%v=1` nhiều nhất: `N/A`.
- Nguyên nhân: `N/A` — không có bộ nhãn đối chiếu của reviewer.
- Luật được củng cố sau self-QC: mọi khớp còn trong biên nhưng bị che phải có tọa độ ước lượng và `v=1`; `v=0` chỉ dùng khi tâm khớp nằm ngoài biên ảnh.
