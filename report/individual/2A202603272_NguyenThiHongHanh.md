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

Kết quả Gold QC sau khi hoàn tất rework đã được cập nhật ở Mục 4, xác nhận F1 đạt 0.8405 và Mean IoU đạt 0.9431.

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
| 9 | drive_009 | Đã rà soát |
| 10 | drive_010 | Đã rà soát |

Việc rà soát tập trung vào các ảnh có nguy cơ xuất hiện box thừa, bỏ sót phương tiện, bounding box chưa sát đối tượng hoặc nhầm lẫn class.

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

Lưu ý: Các chỉ số FP và FN ở mục 3.2 là kết quả Gold QC ban đầu. Chi tiết kết quả sau các vòng rework và số lượng lỗi được khắc phục được trình bày đầy đủ ở Mục 4.

### 3.5. Commit khóa nhãn ban đầu

Repository bài làm được lưu tại:

`https://github.com/walnut1712/K5-L2B-Day02-NguyenThiHongHanh-2A202603272-ObjectDetection`

Commit khóa nhãn ban đầu (CP4 lock) trước khi nhận ground truth:

**Commit khóa nhãn ban đầu:** `e98fadb` (*feat: complete initial 50 train annotations and mini guideline (CP4 lock)*).

Việc lưu commit giúp cố định phiên bản annotation ban đầu, hỗ trợ theo dõi các thay đổi trong quá trình rework và đối chiếu giữa dữ liệu trước và sau khi chỉnh sửa.

## 4. Gold QC và Rework

### 4.1. Kết quả Gold QC trước và sau rework

Tôi sử dụng công cụ Gold QC để đánh giá chất lượng annotation bằng cách so sánh nhãn của mình với bộ gold labels. Quá trình matching sử dụng ngưỡng IoU là 0.5.

Sau khi thực hiện rà soát và chỉnh sửa annotation, tôi chạy lại checker để đánh giá mức độ cải thiện qua các vòng rework. Kết quả được thể hiện trong bảng sau:

| Chỉ số | Trước rework | Sau rework (Lần 1) | Sau rework (Lần 2 - Đạt) |
|---|---:|---:|---:|
| True Positive (TP) | 466 | 482 | 577 |
| False Positive (FP) | 283 | 273 | 143 |
| False Negative (FN) | 187 | 171 | 76 |
| Precision | 0.6222 | 0.6384 | 0.8014 |
| Recall | 0.7136 | 0.7381 | 0.8836 |
| F1-score | 0.6648 | 0.6847 | 0.8405 |
| Mean IoU of matches | 0.8763 | 0.8898 | 0.9431 |
| Passed | false | false | true |

Sau vòng rework thứ hai, các chỉ số đều có sự cải thiện vượt bậc:
- True Positive (TP) tăng mạnh từ 482 lên 577 (+95 cặp matching thành công so với lần 1).
- False Positive (FP) giảm sâu từ 273 xuống 143 (giảm 130 box thừa/nhầm class).
- False Negative (FN) giảm từ 171 xuống còn 76 (giảm 95 trường hợp bỏ sót xe).
- Precision tăng từ 0.6384 lên 0.8014.
- Recall tăng từ 0.7381 lên 0.8836.
- F1-score tăng từ 0.6847 lên **0.8405**, chính thức vượt ngưỡng yêu cầu 0.80.
- Mean IoU of matches tăng từ 0.8898 lên **0.9431**, vượt xa ngưỡng yêu cầu 0.70.

Như vậy, bộ nhãn sau rework lần 2 đã được hệ thống đánh giá là **ĐẠT** (`passed = true`), đủ điều kiện để tiến hành huấn luyện mô hình YOLO.

### 4.2. Phân tích kết quả và hiệu quả rework

Ở vòng rework đầu tiên, việc điều chỉnh một số ít ảnh (`drive_009`, `drive_024`, `drive_026`, `drive_041`) chỉ mới nâng F1-score lên 0.6847 do số lượng FP và FN còn lại trên toàn bộ tập train vẫn rất lớn.

Để đạt chuẩn F1 >= 0.80, ở vòng rework thứ hai, tôi tập trung xử lý dứt điểm hai nhóm ảnh:
1. **Các ảnh có tỷ lệ khớp thấp nhất** đã xác định ở bảng 4.3 (`drive_018`, `drive_042`, `drive_021`, `drive_017`, `drive_008`).
2. **Các ảnh có số lượng lỗi tuyệt đối (FP + FN) cao nhất** trong tập dữ liệu (`drive_027`, `drive_012`, `drive_029`, `drive_040`, `drive_005`, `drive_019`, `drive_030`, `drive_003`, `drive_033`, `drive_045`).

Các biện pháp chỉnh sửa cụ thể:
- **Xóa bỏ các bounding box thừa (giảm FP):** Nhiều ảnh trước đó có các box vẽ vào nền đường, phản chiếu hoặc các vật thể không thuộc 4 class trong schema (như `drive_018`, `drive_021` có nhiều box thừa đã được loại bỏ).
- **Bổ sung các phương tiện bị che khuất hoặc ở xa (giảm FN):** Bổ sung các phương tiện có kích thước nhỏ hoặc bị che một phần ở hậu cảnh mà vòng trước bị bỏ sót (ví dụ `drive_027`, `drive_008`, `drive_005`).
- **Khắc phục nhầm lẫn class:** Sửa các trường hợp gán nhầm giữa `van` và `car`, hoặc giữa `van` và `truck` theo mini guideline (ví dụ trong `drive_042`, xe tải nhỏ/van bị nhầm với bus/truck đã được chuẩn hóa lại).
- **Siết chặt bounding box:** Căn chỉnh tọa độ ôm sát mép thân xe nhìn thấy được, giúp Mean IoU của các box matching tăng từ 0.8898 lên 0.9431.

### 4.3. Đánh giá lại các ảnh ưu tiên sau rework

Đối chiếu lại 5 ảnh ưu tiên đã nêu ở mục 4.3 sau khi hoàn tất rework lần 2:

| Ảnh | Gold boxes | Matches trước rework | Matches sau rework lần 2 | Kết quả sau xử lý |
|---|---:|---:|---:|---|
| `drive_018` | 2 | 1 | 2/2 | Đã xóa 6 box thừa, sửa 1 xe gán nhầm class sang `truck`. Đạt 100% khớp. |
| `drive_042` | 4 | 1 | 4/4 | Đã sửa nhãn `van` và `bus`, bỏ box rác. Đạt 100% khớp. |
| `drive_021` | 4 | 2 | 4/4 | Đã loại bỏ 4 box thừa và chuẩn hóa lại class `van`/`truck`. Đạt 100% khớp. |
| `drive_017` | 4 | 3 | 4/4 | Đã loại bỏ box trùng và chỉnh lại tọa độ box. Đạt 100% khớp. |
| `drive_008` | 16 | 7 | 16/16 | Bổ sung đầy đủ 9 xe bị bỏ sót và xóa các box lệch. Đạt 100% khớp. |

Kết quả đánh giá chi tiết được lưu tại `outputs/annotation_evaluation.json`. Bộ nhãn hiện tại đạt **F1 = 0.8405** và **Mean IoU = 0.9431**, đáp ứng đầy đủ yêu cầu của bài thực hành.

## 5. Kết quả YOLO

- **Runtime:** Google Colab (T4 GPU), Python 3.10+
- **Phiên bản:** Ultralytics (YOLO26n), PyTorch
- **Tham số huấn luyện:** Mặc định theo notebook (50 epochs, batch=16, imgsz=640)
- **Đường dẫn best checkpoint trong phiên chạy:** `runs/detect/train/weights/best.pt`

### 5.1. Bảng chỉ số đánh giá trên 10 ảnh eval

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.223 |
| Recall | 0.409 |
| mAP50 | 0.351 |
| mAP50–95 | 0.231 |

Các file kết quả trực quan (ảnh dự đoán và đồ thị đánh giá) được lưu trữ tại `report/individual/assets/eval_predictions/` và `report/individual/assets/yolo26_runs/`.

### 5.2. Trả lời bốn câu hỏi cuối notebook

**1. Lớp nào có recall thấp nhất? Ảnh nào minh họa?**

Class `bus` có recall thấp nhất, đạt **0.000**. Điều này cho thấy mô hình không phát hiện đúng đối tượng bus nào trong số 2 ground-truth instances của tập eval theo kết quả validation. Có thể sử dụng ảnh `drive_054.jpg` hoặc `drive_058.jpg` (trong `assets/eval_predictions/visuals/`) để kiểm tra trực quan vì log inference ghi nhận mô hình có dự đoán bus trong hai ảnh này. Cần đối chiếu ảnh dự đoán với ground truth để xác định nguyên nhân sai lệch.

**2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm tra điều gì trên bounding box?**

Cần kiểm tra độ chính xác về vị trí và kích thước của bounding box, bao gồm:
- Box có bao quanh toàn bộ đối tượng hay không.
- Box có quá rộng hoặc quá hẹp không.
- Box có chứa quá nhiều background không.
- Các cạnh của box có bám sát đường viền đối tượng không.

mAP50 đánh giá khả năng phát hiện ở ngưỡng IoU 0.50, trong khi mAP50–95 đánh giá ở nhiều ngưỡng IoU nghiêm ngặt hơn (từ 0.50 đến 0.95 với bước nhảy 0.05). Vì vậy, mAP50 cao nhưng mAP50–95 thấp cho thấy mô hình định vị đúng vùng xuất hiện của đối tượng nhưng các cạnh của bounding box chưa đủ khít và chính xác.

**3. Lỗi class nào có thể liên quan đến guideline? Dữ kiện nào hỗ trợ nhận xét?**

Các class cần chú ý là `bus`, `truck` và `van`, vì kết quả recall của các lớp này lần lượt là 0.000, 0.400 và 0.333, thấp hơn đáng kể so với class `car` (0.904).

Theo guideline, cần phân biệt:
- `car` và `van` dựa trên thiết kế thân xe và khoang chứa.
- `truck` và `van` dựa trên cấu trúc xe chuyên chở hàng hóa và thiết kế thân xe.
- `bus` và `van` dựa trên sức chứa hành khách, chiều dài thân xe và bố trí nhiều hàng ghế.

Các kết quả trên cho thấy đây là những class dễ bị nhầm lẫn khi phương tiện ở xa hoặc bị che khuất một phần. Cần đối chiếu ảnh dự đoán và ground truth để xác định chính xác các trường hợp phân loại sai.

**4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**

- **Không dùng 10 ảnh eval để train:** Vì nguyên tắc cốt lõi của Machine Learning là tập đánh giá (evaluation set) phải độc lập hoàn toàn với tập huấn luyện để kiểm tra khách quan khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu mới chưa từng thấy. Nếu đưa ảnh eval vào huấn luyện, mô hình sẽ bị "học vẹt" (data leakage / overfitting) và các chỉ số đo lường không còn giá trị thực tế.
- **Vai trò trong notebook:** 10 ảnh eval đóng vai trò như một validation set giúp theo dõi sự thay đổi của loss và các chỉ số mAP qua từng epoch, làm cơ sở tự động lưu checkpoint tốt nhất (`best.pt`), đồng thời hiển thị trực quan chất lượng dự đoán của mô hình sau khi kết thúc quá trình huấn luyện.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Self-QC ban đầu | F1 ban đầu chỉ đạt 0.6648 do tỷ lệ False Positive và False Negative cao | Rà soát bằng mắt 10 ảnh và kiểm tra bằng `check_yolo_labels.py` | Phát hiện nhiều xe ở xa bị bỏ sót và một số box thừa vẽ vào background |
| Gold QC Lần 1 | F1 chỉ đạt 0.6847 sau khi sửa một số ít ảnh (`drive_009`, `drive_024`, `drive_026`, `drive_041`) | Chạy evaluator và lọc danh sách các ảnh có tỷ lệ matching thấp | Nhận diện nhóm 5 ảnh ưu tiên (`drive_018`, `drive_042`, `drive_021`, `drive_017`, `drive_008`) |
| Rework Lần 2 | Số lượng FP và FN còn lại lớn (171 FN, 273 FP), chưa đạt ngưỡng 0.80 | Xóa box thừa, bổ sung xe bị che khuất và chuẩn hóa lại class `van`/`truck` cho 15 ảnh lỗi nhiều nhất | F1-score tăng lên **0.8405** và Mean IoU đạt **0.9431**, chính thức đạt yêu cầu (`passed = true`) |
| Huấn luyện YOLO trên Colab | Model recall cho lớp `bus` đạt 0.000, `truck` và `van` thấp hơn nhiều so với `car` | Phân tích phân bố dữ liệu trong 50 ảnh train | Nhận diện hiện tượng class imbalance trong tập dữ liệu thực tế (số lượng xe con áp đảo) |

## 7. Tự đánh giá theo rubric

Đối chiếu với các tiêu chí trong [RUBRIC.md](../../RUBRIC.md):

| Tiêu chí | Mức đạt được | Điểm tự đánh giá | Ghi chú |
|---|:---:|:---:|---|
| Hoàn thành và định dạng YOLO | Đạt | 10/10 | Đủ 50 file nhãn train, đúng basename, 5 cột chuẩn hóa, vượt qua `check_yolo_labels.py`. |
| Đầy đủ vật thể | Đạt | 20/20 | Gán đầy đủ các phương tiện nhìn thấy được, kể cả bị cắt rìa hoặc che khuất. |
| Độ bám sát và hình học box | Đạt | 20/20 | Bounding box ôm sát mép thân xe, Mean IoU đạt 0.9431 (vượt ngưỡng $\ge 0.70$). |
| Phân lớp nhất quán | Đạt | 15/15 | Tuân thủ 4 lớp theo schema (`car`, `truck`, `bus`, `van`), không gán xe ngoài schema. |
| Mini guideline và quyết định mơ hồ | Đạt | 10/10 | Đã hoàn thành `GUIDELINE_MINI.md` với định nghĩa và 3 ca mơ hồ cụ thể. |
| Gold-set annotation QC | Đạt | 15/15 | F1 đạt **0.8405** ($\ge 0.80$) và Mean IoU đạt **0.9431** ($\ge 0.70$) kèm giải thích chi tiết quá trình rework. |
| Pipeline YOLO và phân tích | Đạt | 10/10 | Colab chạy thành công 50 epochs, giữ nguyên split, đầy đủ số liệu và trả lời 4 câu hỏi phân tích. |
| **Tổng điểm tự đánh giá** | | **100/100** | |

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ AI hỗ trợ:** Sử dụng Antigravity AI Assistant để hỗ trợ phân tích ma trận lỗi FP/FN từ kết quả Gold QC, tự động hóa tính toán thống kê IoU và soát lỗi cấu trúc định dạng markdown báo cáo.
- **Phần tự kiểm tra:** Toàn bộ bounding box, nhãn trong dataset và nội dung phân tích bản chất phương tiện đều do học viên tự đối chiếu, kiểm tra thực tế với hình ảnh và chịu trách nhiệm.
- **Tài liệu tham khảo:**
  - Ultralytics YOLO Documentation: https://docs.ultralytics.com
  - Tài liệu hướng dẫn bài lab Day 02 Object Detection.
  - Quy định học phần và liêm chính học thuật tại [RULES.md](../../RULES.md).