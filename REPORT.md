# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Nguyen Van Han

**MSSV:** 2A202602339

**Hình thức:** Cá nhân

**Mã cặp:** `SOLO`

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- Bốn mã ảnh: `drive_022`, `drive_033`, `drive_038`, `drive_008`
- Số vật thể thực tế: **57** (`car`: 43, `truck`: 2, `bus`: 5, `van`: 7)
- Mã SHA-256 của gói YOLO của tôi (`YOLO.zip`, có kèm ảnh): `40c79dbbaaaccc308f8c9107e9f6e1ec5e88debe5adcb140ced82cbe7257d914`
- Mã SHA-256 của gói CVAT gốc của tôi: `e416529007722fca03916fd3dc5250d9a35f139350535182ddcce43d81db49d3`
- Nguồn đối chiếu: bộ nhãn đối chiếu do Lab Coach cấp.
- Mã SHA-256 của gói đối chiếu: `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
- Mã lần phát:



## 2. Quyết đnh phân lớp


| Ảnh/vật thể         | Lớp     | Dấu hiệu nhìn thấy                                  | Quy tắc áp dụng                                      |
| ------------------- | ------- | --------------------------------------------------- | ---------------------------------------------------- |
| `drive_022`, hàng 1 | `bus`   | Thân xe khách dài, nhiều cửa sổ, kích thước lớn     | Xe khách dài và có nhiều cửa sổ được gán `bus`       |
| `drive_008`, hàng 1 | `truck` | Có thùng hàng lớn tách biệt phía sau cabin          | Xe có thùng/ben hoặc khoang hàng rõ được gán `truck` |
| `drive_038`, hàng 4 | `car`   | Thân xe con, không có thùng hàng hoặc thân xe khách | Sedan/SUV/hatchback được gán `car`                   |


Lớp và thuộc tính là hai loại thông tin khác nhau. Ví dụ, vật thể `drive_008`, hàng 15 có lớp `bus`, nhưng đồng thời có `visibility=unclear` và `boundary=truncated`: `bus` trả lời “đây là loại xe gì”, còn hai thuộc tính mô tả mức nhìn thấy và quan hệ với mép ảnh.

## 3. Tự kiểm tra và sửa nhãn


| Trước khi sửa                                                                                | Loại lỗi           | Cách phát hiện                                                                        | Sau khi sửa và quy tắc                                                                                                                  |
| -------------------------------------------------------------------------------------------- | ------------------ | ------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `drive_038`, hàng 4: `car`, `boundary=inside`, `review_state=needs_review`, hộp có `ybr=640` | Thuộc tính mép ảnh | Ảnh cao 640 px và hộp chạm đúng mép dưới; ảnh phủ cũng cho thấy xe bị cắt bởi mép ảnh | Gói hiện tại chưa được sửa; cần đổi `boundary` thành `truncated`, kiểm tra lại hình học rồi mới chuyển `review_state` thành `confident` |


- Số hộp `needs_review` trước khi tự kiểm: **1/57**.
- Quyết định chưa đủ bằng chứng: `drive_008`, hàng 15 chỉ lộ một phần nhỏ ở mép trên và có `visibility=unclear`.



## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: `2 0.390273 0.723195 0.433516 0.353297`
- Tên lớp và tọa độ điểm ảnh `xyxy`: lớp `bus`; `[111.0, 349.8, 388.5, 575.9]` trên ảnh 640 × 640.
- Dòng đúng định dạng chỉ chứng minh có đủ năm trường và tọa độ hợp lệ. Nó vẫn có thể sai nếu chọn nhầm lớp, gán một vật ngoài phạm vi, bỏ sót/ghép nhiều xe, hoặc vẽ hộp chứa quá nhiều nền hay ước lượng cả phần bị che.



## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`
- Mã ảnh thẩm định: `drive_008`
- Cấu hình: Ultralytics `8.4.145`, `yolo11n.pt`, tối đa 8 epoch, `seed=42`, `imgsz=640`, `batch=4`, `freeze=10`, `patience=3`, ngưỡng dự đoán `conf=0.25`.
- Do phần output Colab của ô huấn luyện bị thu gọn khi lập báo cáo, tôi tái tạo cùng dữ liệu, trọng số, seed và siêu tham số trên CPU. Lần tái tạo dừng sớm ở epoch 4 và không có hộp dự đoán nào đạt `conf >= 0.25` trên `drive_008`; ảnh kết quả vì vậy không có hộp phủ.
- Kết quả này gợi ý cần kiểm lại độ nhất quán lớp và mất cân bằng dữ liệu: 43/57 nhãn là `car`, chỉ có 2 `truck`, đồng thời một số vật thể `bus/van/truck` khác lớp so với bộ đối chiếu.
- Minh chứng có thể bác bỏ nhận định trên là output gốc của đúng lần chạy Colab/GPU có hộp dự đoán ổn định, hoặc kết quả lặp lại trên một tập kiểm thử độc lập lớn hơn với nhãn đã được rà soát.
- Bốn ảnh, trong đó chỉ một ảnh thẩm định và ba ảnh huấn luyện, không đại diện cho phân phối thực tế. Không có tập kiểm thử độc lập, độ đa dạng điều kiện, đánh giá sai số theo lớp hay kiểm tra độ ổn định; vì vậy đây chỉ là tín hiệu chẩn đoán dữ liệu, không phải đánh giá mô hình dùng trong thực tế.



## 6. Đối chiếu nhãn

- Cách ghép: tối ưu IoU theo hình học trên từng ảnh, không dùng lớp để ghép; sàn kỹ thuật `0.01` không phải ngưỡng đạt.
- Số hộp ghép được: **42**
- IoU trung bình và trung vị: **0.847714** và **0.849503**
- Mức đồng thuận lớp: **0.690476** (29/42 hộp ghép, khoảng 69.05%)
- Số hộp phía tôi không ghép được: **15**
- Số hộp phía đối chiếu không ghép được: **8**
- Một khác biệt cụ thể: tại `drive_022`, hộp hàng 1 của tôi là `bus`, còn hộp ghép của bộ đối chiếu là `van`; IoU **0.928400**. Hình học gần trùng nhưng lớp khác nhau.
- Hành động phát sinh: mở lại ảnh ở 100%, áp dụng quy tắc “thân xe khách dài, nhiều cửa sổ → `bus`”. Tôi không đổi nhãn chỉ để khớp bộ tham chiếu ; ảnh và quy tắc đang ủng hộ `bus`.
- Đồng thuận cao không chứng minh mọi nhãn đúng vì hai nguồn có thể cùng dùng sai một quy tắc, cùng bỏ sót vật thể, hoặc vẽ hộp giống nhau nhưng gán sai lớp. IoU cao chủ yếu cho thấy hình học gần nhau.



## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất trong báo cáo này.
- [x] Có thông tin lần huấn luyện và kết quả dự đoán chẩn đoán.
- [x] Có tóm tắt và số liệu của bước đối chiếu, nhưng chưa xác minh ảnh phủ đã được đưa vào kho GitHub.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất là hai định dạng xuất độc lập đều có đúng 57 hộp và cùng phân bố lớp `43/2/5/7`, kèm hash của từng gói. Câu hỏi còn lại cho Lab Coach: vì sao một số hộp có hình học gần trùng nhưng bộ tham chiếu đổi `bus` thành `van` hoặc `truck` thành `bus`?
