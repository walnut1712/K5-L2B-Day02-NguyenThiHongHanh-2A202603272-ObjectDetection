# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe ô tô con, chủ yếu dùng để chở người, kích thước nhỏ hoặc trung bình. | Sedan, hatchback, SUV, coupe. |
| `truck` | Xe tải được thiết kế chủ yếu để vận chuyển hàng hóa. | Xe tải thùng, xe tải ben, xe tải đầu kéo. |
| `bus` | Xe cơ giới được thiết kế để vận chuyển nhiều hành khách. | Xe buýt công cộng, xe khách, xe bus trường học.|
| `van` | Xe có khoang thân kín, thường có thiết kế hình hộp, dùng để chở người hoặc hàng hóa. | Minivan, passenger van, cargo van. |

## Quy tắc phân biệt:
- car và van: Phân biệt dựa trên thiết kế thân xe và khoang chứa hàng/người.
- truck và van: Truck thường có thiết kế xe tải chuyên chở hàng hóa; van có thân xe kín liền khối.
- bus và van: Bus có thiết kế chuyên chở nhiều hành khách hơn, thường có thân xe dài và nhiều hàng ghế.

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Bounding box phải bao quanh toàn bộ phần vật thể nhìn thấy được, không bao gồm quá nhiều background.
- Vật thể bị che khuất: Vẫn gán nhãn nếu nhận diện được đối tượng. Box bao quanh phần nhìn thấy được và bật thuộc tính Occluded nếu bị vật thể khác che khuất.
- Vật thể bị cắt bởi rìa ảnh: Gán nhãn nếu phần còn lại đủ để xác định lớp đối tượng. Box không vượt ra ngoài ranh giới ảnh.
- Vật thể rất nhỏ/mơ hồ: Chỉ gán nhãn khi có đủ bằng chứng để xác định lớp. Không đoán nhãn nếu không thể phân biệt đáng tin cậy.   

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_...` | | | |
| `drive_...` | | | |
| `drive_...` | | | |
