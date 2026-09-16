# Báo cáo Ngày 4 - Keypoint & Pose

Họ tên: Nguyễn Tuấn Khôi  
Nhóm: Cá nhân  
Ngày: 16/09/2026

## 1. Nhãn của tôi

| Chỉ số | Giá trị |
| --- | ---: |
| Số ảnh đã gán | 20 |
| Số skeleton | 29 |
| v=2 / v=1 / v=0 | 355 / 109 / 29 |
| Thời gian trung bình mỗi ảnh | Khoảng 3–4 phút/ảnh |

Ba khớp có `%v=1` cao nhất:

1. `left_ear` — 52%
2. `left_eye` — 31%
3. `right_ear` — 31%

Các khớp này đúng là nhóm thường gây phân vân vì tóc, mũ bảo hiểm, góc quay đầu và độ phân giải làm tâm mắt/tai khó quan sát trực tiếp. Khi tâm khớp vẫn nằm trong ảnh nhưng bị che, tôi đặt vị trí ước lượng và dùng `v=1`, không dùng Outside. Ngoài nhóm đầu, hông và cổ tay cũng khó vì thường bị quần áo, thân người hoặc vật đang cầm che khuất.

## 2. Chấm với gold

| Chỉ số | Trước rework | Sau rework |
| --- | ---: | ---: |
| OKS trung bình | 0.887 | 0.9417 |
| OKS@0.50 | 1.000 | 1.000 |
| OKS@0.75 | 0.966 | 1.000 |
| Lỗi `dao_trai_phai` | 0 | 0 |
| Lỗi `nham_nguoi` | 0 | 0 |
| Lỗi `xoa_khop_bi_che` | 13 | 0 |

**Tôi đã sửa gì giữa hai lần chạy:**

- `train_15`, người #1: bổ sung `right_ear` nhìn thấy; đặt lại `right_wrist` và `left_knee` ở vị trí giải phẫu ước lượng rồi chuyển từ Outside sang Occluded (`v=1`).
- `train_06`, người #1: đặt lại `left_ear` và `right_ear` vì đầu vẫn nằm trong ảnh; không coi việc bị mũ/góc quay che là Outside.
- `train_16`, người #2: đặt lại `nose` dưới dạng Occluded và bổ sung `right_eye`, `right_ear` theo phần đầu còn nhìn thấy.

**Lỗi đảo trái/phải của tôi xảy ra ở ảnh nào?** Không có lỗi đảo trái/phải trong toàn bộ 20 ảnh ở cả hai lần đánh giá. Tôi đã kiểm thêm bằng ảnh overlay trong `outputs/vis_train`, trong đó bên trái cơ thể được vẽ màu xanh và bên phải màu cam; các cảnh báo heuristic ở `train_02` và `train_16` không tạo lỗi đảo trái/phải khi so với gold.

## 3. Kiểm chéo

Bạn cùng nhóm: Không thực hiện — bài được self-QC cá nhân theo yêu cầu của người học.

| Khớp | Bạn | Họ | Lệch | Nguyên nhân |
| --- | ---: | ---: | ---: | --- |
| N/A | N/A | N/A | N/A | Không có bộ nhãn peer để đối chiếu. |
| N/A | N/A | N/A | N/A | Không giả lập số liệu kiểm chéo. |

Luật được củng cố sau self-QC và rework: khớp còn trong biên ảnh nhưng bị tóc, quần áo, người khác hoặc vật thể che phải có tọa độ ước lượng và `v=1`; chỉ dùng `v=0` khi tâm khớp thực sự nằm ngoài biên ảnh.

## 4. Model

| Chỉ số | yolo26n-pose gốc | Sau fine-tune | Chênh |
| --- | ---: | ---: | ---: |
| pose_mAP50 | 0.8450 | 0.8450 | +0.0000 |
| pose_mAP50-95 | 0.6853 | 0.6908 | +0.0055 |
| pose_precision | 0.9734 | 0.9792 | +0.0058 |
| pose_recall | 0.8462 | 0.8462 | +0.0000 |
| box_mAP50-95 | 0.8119 | 0.8041 | -0.0078 |

### Trả lời năm câu hỏi ở cuối notebook

1. `pose_mAP50-95` tăng `0.0055`, từ `0.6853` lên `0.6908`. Mức tăng nhỏ cho thấy 20 ảnh fine-tune giúp model thích nghi nhẹ với cách gán COCO-17 và các ca che khuất trong tập này, nhưng chưa đủ để kết luận model tổng quát tốt hơn. Kích thước tập train chỉ 20 ảnh nên kết quả có thể dao động mạnh theo thành phần của 10 ảnh test.

2. Sau fine-tune, `box_mAP50-95 = 0.8041` và `pose_mAP50-95 = 0.6908`, chênh `0.1133`. Model tìm người dễ hơn tìm chính xác 17 khớp: bbox chỉ cần bao đúng thân người, còn pose phải đặt đúng từng điểm, giữ đúng trái/phải và xử lý khớp bị che. Việc box giảm `0.0078` trong khi pose tăng `0.0055` cũng cho thấy fine-tune trên tập nhỏ tạo ra một đánh đổi nhẹ thay vì cải thiện đồng đều mọi chỉ số.

3. Ở `test_02`, ảnh gốc chỉ có một người đứng phía phải, còn các vật thể bên trái là chim, cột và mép mái. Kết quả notebook báo hai `person`, tức model tạo thêm một dự đoán trên vùng không phải người. Tôi xếp đây là lỗi **trượt hẳn**, không phải lệch nhẹ, vì toàn bộ dự đoán thừa rơi vào nền/vật thể thay vì chỉ lệch vài pixel khỏi một khớp thật.

4. `train_13` có OKS thấp nhất giữa model và nhãn của tôi, bằng `0.580`. Tôi tin nhãn của mình đúng hơn ở ca này vì lần chấm sau rework ghép đủ cả ba người với gold, không thiếu/thừa người, và ba OKS tương ứng là `0.9560`, `0.9013`, `0.9455`. Ảnh có ba người gần nhau và bị che một phần, nên model dễ ghép nhầm hoặc đặt khớp sang cơ thể lân cận dù nhãn đã qua cổng gold.

5. Không trùng nhau. Trước rework, skeleton tôi gán tệ nhất là `train_15`, người #1, OKS `0.688`; lỗi chính là thiếu `right_ear` và xoá các khớp bị che như `right_wrist`, `left_knee`, sau đó đã được sửa trong CVAT. Model bất đồng nhiều nhất với nhãn ở `train_13` (OKS `0.580`), một cảnh ba người gần nhau; vì vậy lỗi của tôi ban đầu chủ yếu là thao tác visibility, còn lỗi của model liên quan đến cảnh nhiều người và che khuất.

## 5. Một rule evidence tôi đã dùng

Ở `train_06`, người #1, `right_wrist` bị thân người và xe máy che nên không thấy trực tiếp bề mặt khớp. Tuy vậy khuỷu tay, hướng cẳng tay và toàn bộ vùng tay vẫn nằm trong biên ảnh, cho phép ước lượng vị trí giải phẫu của cổ tay. Tôi đặt lại chấm theo hướng cẳng tay và chọn Occluded (`v=1`), không dùng Outside. Nếu để `v=0`, khớp bị che nhưng còn trong ảnh sẽ bị loại khỏi OKS và model không nhận được tín hiệu học cho ca che khuất này.
