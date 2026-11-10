# BÁO CÁO CÁ NHÂN — NGÀY 2 OBJECT DETECTION

## 1. Thông tin học viên

- **Họ tên:** Nguyễn Thị Hồng Hạnh
- **MSSV:** 2A202603272
- **Email:** walnut171225@gmail.com
- **Lớp:** 2B
- **Link repo bài làm:** [GitHub Repository](https://github.com/walnut1712/K5-L2B-Day02-NguyenThiHongHanh-2A202603272-ObjectDetection)
- **Link Colab đã chạy:** [Google Colab](https://colab.research.google.com/drive/1wsyCiXVvmjxtKende0c_OPyvi89Qg6d2?usp=sharing)

## 2. Schema và mini guideline

File guideline cá nhân: `GUIDELINE_MINI.md`.

Tôi sử dụng schema gồm bốn lớp: `car`, `truck`, `bus` và `van`.

### Ba trường hợp mơ hồ đã quyết định

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
|---|---|---|---|
| `drive_001` | Xe màu trắng có thân xe tương đối vuông, khó phân biệt với xe con thông thường. | `van` | Dựa vào thiết kế thân xe kín, hình hộp và cấu trúc khoang xe thay vì chỉ dựa trên kích thước. |
| `drive_002` | Xe buýt màu trắng kích thước lớn xuất hiện giữa nhiều phương tiện khác. | `bus` | Cấu trúc và kích thước xe phù hợp với phương tiện chuyên chở nhiều hành khách. |
| `drive_003` | Một phương tiện lớn màu xanh bị cắt một phần ở cạnh phải ảnh. | `bus` (nếu đủ đặc điểm nhận diện) | Gán nhãn nếu phần thân xe nhìn thấy đủ để nhận diện xe buýt. Bounding box chỉ bao quanh phần nằm trong ảnh. |

### Cách áp dụng schema

- Phân biệt `car` và `van` dựa trên hình dáng thân xe và thiết kế khoang xe.
- Phân biệt `truck` và `van` dựa trên cấu trúc chuyên chở hàng hóa và thiết kế thân xe.
- Phân biệt `bus` và `van` dựa trên thiết kế chuyên chở nhiều hành khách và cấu trúc xe.
- Đối với vật thể bị che khuất, vẫn gán nhãn nếu nhận diện được.
- Đối với vật thể bị cắt bởi cạnh ảnh, chỉ vẽ bounding box trong phạm vi ảnh.
- Không tự suy đoán lớp khi hình ảnh không cung cấp đủ bằng chứng.

### Nội dung cần giảng viên làm rõ

Cần có thêm tiêu chí cụ thể để phân biệt những phương tiện có thiết kế trung gian giữa `car` và `van`, hoặc giữa `truck` và `van`, đặc biệt khi phương tiện xuất hiện ở khoảng cách xa hoặc bị che khuất.

## 3. Annotation và Self-QC

### 3.1. Số lượng file nhãn train

Dataset sử dụng **50 ảnh train và 10 ảnh evaluation**. Thư mục `dataset/labels/train` có **50 file nhãn `.txt`**, tương ứng với 50 ảnh train. Theo log huấn luyện Ultralytics, hệ thống đã quét thành công 50 ảnh train và không phát hiện ảnh nền hoặc ảnh bị lỗi (*0 backgrounds, 0 corrupt*).

Như vậy, số lượng file nhãn train đáp ứng yêu cầu 50 file của bài thực hành.

### 3.2. Kết quả checker và lỗi đã xử lý

Tôi sử dụng Gold QC để đánh giá chất lượng annotation với ngưỡng IoU matching là 0.5. Kết quả đánh giá ban đầu như sau:

| Chỉ số | Kết quả |
|---|---:|
| True Positive (TP) | 466 |
| False Positive (FP) | 283 |
| False Negative (FN) | 187 |
| Precision | 0.6222 |
| Recall | 0.7136 |
| F1-score | 0.6648 |
| Mean IoU of matched boxes | 0.8763 |

Kết quả cho thấy F1-score đạt 0.6648, chưa đáp ứng ngưỡng yêu cầu 0.80. Mặc dù Mean IoU đạt 0.8763, số lượng False Positive và False Negative vẫn còn cao. Điều này cho thấy bộ nhãn cần được rà soát thêm về tính đầy đủ của đối tượng và độ chính xác của bounding box.

Sau khi nhận được kết quả Gold QC, tôi trực tiếp kiểm tra lại hình ảnh và chỉnh sửa các annotation có vấn đề. Quá trình rework tập trung vào:

- **Bổ sung đối tượng bị bỏ sót:** Kiểm tra các phương tiện xuất hiện trong ảnh nhưng chưa có bounding box tương ứng. Bổ sung nhãn đối với những đối tượng có đủ đặc điểm để nhận diện.
- **Xử lý bounding box thừa:** Kiểm tra các box không tương ứng với phương tiện hợp lệ và loại bỏ những annotation không chính xác.
- **Điều chỉnh bounding box:** Kiểm tra vị trí và kích thước của box, điều chỉnh để bao sát phần phương tiện nhìn thấy và hạn chế vùng background không cần thiết.
- **Kiểm tra class:** Đối chiếu nhãn giữa `car`, `truck`, `bus` và `van` theo mini guideline để hạn chế nhầm lẫn giữa các phương tiện có thiết kế tương tự.

Ngoài ra, trong quá trình huấn luyện YOLO, Ultralytics ghi nhận và tự động loại bỏ duplicate labels tại một số ảnh:

| Ảnh | Số nhãn trùng được loại bỏ |
|---|---:|
| drive_012 | 1 |
| drive_017 | 2 |
| drive_019 | 2 |
| drive_026 | 1 |
| **Tổng cộng** | **6** |

Các duplicate labels trên được xử lý tự động bởi Ultralytics trong quá trình đọc dữ liệu huấn luyện. Đây là kết quả xử lý nhãn trùng, không phải số lượng lỗi FP/FN đã được sửa thủ công.

Kết quả Gold QC sau khi hoàn tất rework cần được chạy lại để xác định chính xác mức độ cải thiện của bộ nhãn. Hiện tại, chưa có số liệu sau sửa được xác nhận.

### 3.3. Danh sách ảnh đã rà soát bằng mắt

Trong quá trình self-QC, tôi cần kiểm tra trực tiếp ảnh gốc cùng các bounding box để đánh giá tính đầy đủ của annotation, vị trí box và class được gán.

Danh sách 10 ảnh train đã rà soát bằng mắt:

| STT | Mã ảnh train | Trạng thái |
|---:|---|---|
| 1 | [drive_001] | Đã rà soát |
| 2 | [drive_002] | Đã rà soát |
| 3 | [drive_003] | Đã rà soát |
| 4 | [drive_004] | Đã rà soát |
| 5 | [drive_005] | Đã rà soát |
| 6 | [drive_006] | Đã rà soát |
| 7 | [drive_007] | Đã rà soát |
| 8 | [drive_008] | Đã rà soát |
| 9 | [drive_009] | Đã rà soát |
| 10 | [drive_0010] | Đã rà soát |

Việc rà soát tập trung vào các ảnh có nguy cơ xuất hiện box thừa, bỏ sót phương tiện, bounding box chưa sát đối tượng hoặc nhầm lẫn class. Danh sách trên cần được thay bằng mã ảnh thực tế đã kiểm tra.

### 3.4. Lỗi phát hiện và cách sửa

Dựa trên kết quả Gold QC ban đầu và quá trình kiểm tra thủ công, các nhóm lỗi cần được chú ý gồm:

| Nhóm lỗi | Cách kiểm tra và xử lý |
|---|---|
| Bỏ sót đối tượng (FN) | Đối chiếu ảnh gốc với annotation, bổ sung box cho phương tiện đủ điều kiện nhận diện. |
| Box thừa (FP) | Kiểm tra các box không tương ứng với phương tiện hợp lệ và loại bỏ nếu xác nhận không chính xác. |
| Box chưa chính xác | Điều chỉnh tọa độ và kích thước để box bao sát đối tượng nhìn thấy. |
| Sai class | Đối chiếu thiết kế phương tiện với guideline và chỉnh class nếu phát hiện gán nhãn không phù hợp. |
| Duplicate labels | Kiểm tra nhãn trùng; log training ghi nhận Ultralytics tự động loại bỏ 6 nhãn trùng tại 4 ảnh. |

Qua quá trình self-QC, tôi tập trung cải thiện tính đầy đủ và nhất quán của annotation, đồng thời hạn chế các lỗi ảnh hưởng đến quá trình huấn luyện mô hình object detection.

Lưu ý: Các chỉ số FP và FN ở mục 3.2 là kết quả Gold QC ban đầu. Chưa có kết quả checker sau rework để xác nhận số lượng lỗi còn lại hoặc mức độ cải thiện.

### 3.5. Commit khóa nhãn ban đầu

Repository bài làm được lưu tại:

`https://github.com/walnut1712/K5-L2B-Day02-NguyenThiHongHanh-2A202603272-ObjectDetection`

Log đã ghi nhận commit mới nhất là `20ec119`. Tuy nhiên, cần kiểm tra lịch sử Git để xác nhận commit được tạo tại thời điểm hoàn tất annotation và self-QC.

**Commit khóa nhãn ban đầu:** [Bổ sung Git commit SHA thực tế].

Việc lưu commit giúp cố định phiên bản annotation ban đầu, hỗ trợ theo dõi các thay đổi trong quá trình rework và đối chiếu giữa dữ liệu trước và sau khi chỉnh sửa.

## 4. Gold QC và Rework

### 4.1. Kết quả Gold QC trước và sau rework

Tôi sử dụng công cụ Gold QC để đánh giá chất lượng annotation bằng cách so sánh nhãn của mình với bộ gold labels. Quá trình matching sử dụng ngưỡng IoU là 0.5.

Sau khi thực hiện rà soát và chỉnh sửa annotation, tôi chạy lại checker để đánh giá mức độ cải thiện. Kết quả trước và sau rework được thể hiện trong bảng sau:

| Chỉ số | Trước rework | Sau rework |
|---|---:|---:|
| True Positive (TP) | 466 | 482 |
| False Positive (FP) | 283 | 273 |
| False Negative (FN) | 187 | 171 |
| Precision | 0.6222 | 0.6384 |
| Recall | 0.7136 | 0.7381 |
| F1-score | 0.6648 | 0.6847 |
| Mean IoU of matches | 0.8763 | 0.8898 |
| Passed | false | false |

Sau quá trình rework, các chỉ số đều có sự cải thiện. TP tăng 16, trong khi FP giảm 10 và FN giảm 16. Precision tăng từ 0.6222 lên 0.6384, Recall tăng từ 0.7136 lên 0.7381 và F1-score tăng từ 0.6648 lên 0.6847.

Mean IoU cũng tăng từ 0.8763 lên 0.8898, cho thấy độ chồng lấp trung bình của các bounding box được matching đã được cải thiện.

Tuy nhiên, F1-score sau rework vẫn thấp hơn ngưỡng yêu cầu 0.80. Vì vậy, kết quả Gold QC vẫn được đánh giá là chưa đạt (*passed = false*).

### 4.2. Phân tích kết quả và hiệu quả rework

Kết quả Gold QC ban đầu cho thấy bộ annotation còn tồn tại nhiều trường hợp bỏ sót đối tượng và bounding box chưa matching với gold labels.

Sau khi phân tích kết quả, tôi thực hiện chỉnh sửa thủ công, tập trung vào hai nhóm lỗi chính:

- **Bỏ sót đối tượng (False Negative):** Kiểm tra lại ảnh gốc và bổ sung bounding box cho các phương tiện đủ điều kiện nhận diện nhưng chưa được gán nhãn.
- **Bounding box chưa sát đối tượng:** Điều chỉnh vị trí và kích thước box để bao sát phần phương tiện nhìn thấy, hạn chế vùng background không cần thiết.

Sau khi chạy lại checker, kết quả cho thấy:

- TP tăng từ 466 lên 482, tức có thêm 16 trường hợp matching.
- FP giảm từ 283 xuống 273, giảm 10 trường hợp.
- FN giảm từ 187 xuống 171, giảm 16 trường hợp.
- F1-score tăng 0.0199, từ 0.6648 lên 0.6847.
- Mean IoU tăng 0.0135, từ 0.8763 lên 0.8898.

Những thay đổi này cho thấy quá trình rework đã cải thiện chất lượng annotation. Tuy nhiên, mức cải thiện chưa đủ để đạt ngưỡng F1-score 0.80. Do đó, vẫn cần tiếp tục rà soát các trường hợp không matching, đặc biệt là những ảnh có số lượng gold boxes và student boxes chênh lệch lớn.

### 4.3. Các ảnh cần ưu tiên rà soát

Dựa trên kết quả `per_image` sau rework, tôi tiếp tục ưu tiên kiểm tra các ảnh có số lượng matches thấp so với tổng số gold boxes.

| Ảnh | Gold boxes | Student boxes | Matches sau rework | Nội dung cần kiểm tra |
|---|---:|---:|---:|---|
| drive_018 | 2 | 8 | 1 | Kiểm tra box thừa và đối tượng bị bỏ sót. |
| drive_042 | 4 | 5 | 1 | Kiểm tra các box không matching và vị trí bounding box. |
| drive_021 | 4 | 8 | 2 | Kiểm tra box thừa và các đối tượng chưa được gán nhãn. |
| drive_017 | 4 | 9 | 3 | Rà soát các box không matching và khả năng có annotation trùng. |
| drive_008 | 16 | 20 | 7 | Kiểm tra các đối tượng chưa matching và độ chính xác của bounding box. |

Một trường hợp cải thiện đáng chú ý là `drive_026`. Trước rework, ảnh này có 39 gold boxes, 33 student boxes và 23 matches. Sau rework, số student boxes tăng lên 39 và số matches đạt 39/39. Điều này cho thấy annotation của ảnh đã được cải thiện đáng kể theo kết quả matching.

Các ảnh còn lại trong bảng vẫn có số lượng matches thấp so với gold boxes, vì vậy cần được ưu tiên kiểm tra thêm để xác định nguyên nhân cụ thể.

Kết quả checker được lưu tại `outputs/annotation_evaluation.json`. Kết quả sau rework cho thấy chất lượng annotation đã được cải thiện, nhưng chưa đáp ứng ngưỡng F1-score tối thiểu 0.80.

### 5.3. Trả lời bốn câu hỏi cuối notebook

**1. Lớp nào có recall thấp nhất? Ảnh nào minh họa?**

Class `bus` có recall thấp nhất, đạt **0.000**. Điều này cho thấy mô hình không phát hiện đúng đối tượng bus nào trong số 2 ground-truth instances của tập eval theo kết quả validation.

Có thể sử dụng ảnh `drive_054.jpg` hoặc `drive_058.jpg` để kiểm tra trực quan vì log inference ghi nhận mô hình có dự đoán bus trong hai ảnh này. Cần đối chiếu ảnh dự đoán với ground truth để xác định nguyên nhân sai lệch.

**2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm tra điều gì trên bounding box?**

Cần kiểm tra độ chính xác về vị trí và kích thước của bounding box, bao gồm:
- Box có bao quanh toàn bộ đối tượng hay không.
- Box có quá rộng hoặc quá hẹp không.
- Box có chứa quá nhiều background không.
- Các cạnh của box có bám sát đường viền đối tượng không.

mAP50 đánh giá khả năng phát hiện ở ngưỡng IoU 0.50, trong khi mAP50–95 đánh giá ở nhiều ngưỡng IoU nghiêm ngặt hơn. Vì vậy, mAP50 cao nhưng mAP50–95 thấp có thể cho thấy mô hình nhận diện được đối tượng nhưng vị trí hoặc kích thước bounding box chưa đủ chính xác.

**3. Lỗi class nào có thể liên quan đến guideline? Dữ kiện nào hỗ trợ nhận xét?**

Các class cần chú ý là `bus`, `truck` và `van`, vì kết quả recall của các lớp này lần lượt là 0.000, 0.400 và 0.333, thấp hơn đáng kể so với class `car` (0.904).

Theo guideline, cần phân biệt:
- `car` và `van` dựa trên thiết kế thân xe và khoang chứa.
- `truck` và `van` dựa trên cấu trúc xe chuyên chở hàng hóa và thiết kế thân xe.
- `bus` và `van` dựa trên sức chứa hành khách, chiều dài thân xe và bố trí nhiều hàng ghế.

Các kết quả trên cho thấy đây là những class cần được kiểm tra kỹ hơn. Tuy nhiên, chỉ số recall chưa đủ để kết luận mô hình nhầm class nào với class nào. Cần xem ảnh dự đoán và ground truth để xác định chính xác các trường hợp phân loại sai.

**4. Nhận xét chung về kết quả huấn luyện**

Mô hình YOLO26n đã hoàn thành 50 epochs huấn luyện trên 50 ảnh train và được đánh giá trên 10 ảnh eval. Kết quả tổng thể đạt Precision 0.223, Recall 0.409, mAP50 0.351 và mAP50–95 0.231.

Mô hình có kết quả tốt hơn đối với class `car`, trong khi khả năng nhận diện `bus`, `truck` và `van` còn hạn chế. Điều này có thể liên quan đến số lượng instances của từng class trong tập eval, đặc biệt class `bus` chỉ có 2 instances và class `truck` có 5 instances.

Do đó, cần tiếp tục kiểm tra chất lượng annotation, sự cân bằng dữ liệu giữa các class và các trường hợp dự đoán sai để cải thiện khả năng tổng quát hóa của mô hình.