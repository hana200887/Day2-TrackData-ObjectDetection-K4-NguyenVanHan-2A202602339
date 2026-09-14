# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Nguyen Van Han

**MSSV:** 2A202602339

**Hình thức:** Cá nhân

**Mã cặp:** `SOLO`

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định


| Mã  | Lớp              | Gán khi nhìn thấy                                       | Không gán vào lớp này                             |
| --- | ---------------- | ------------------------------------------------------- | ------------------------------------------------- |
| 0   | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
| 1   | `truck` (xe tải) | thùng, ben, sàn hàng hóa hoặc thiết bị công vụ rõ ràng  | ô tô con; thân xe buýt; xe van kín một khối       |
| 2   | `bus` (xe buýt)  | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế           | xe van nhỏ; xe tải; ô tô con                      |
| 3   | `van` (xe van)   | thân hộp nhỏ, kín, dùng chở người hoặc hàng             | thân xe buýt; khoang hàng tách biệt như xe tải    |


Thứ tự lớp cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.



## 4. Ba thuộc tính


| Thuộc tính     | Giá trị                        | Ý nghĩa                                  |
| -------------- | ------------------------------ | ---------------------------------------- |
| `visibility`   | `clear`, `occluded`, `unclear` | Mức bằng chứng nhìn thấy                 |
| `boundary`     | `inside`, `truncated`          | Vật thể có bị mép ảnh cắt hay không      |
| `review_state` | `confident`, `needs_review`    | Quyết định đã chắc chắn hay cần quay lại |


YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ



### Tình huống A — xe buýt hay xe van?

- Ảnh và mã vật thể: `drive_022`, hàng 1 trong gói YOLO của tôi.
- Dấu hiệu nhìn thấy: xe lớn, thân dài, nhiều cửa sổ hành khách; không phải thân hộp nhỏ của xe van.
- Quy tắc áp dụng: thân xe khách dài, nhiều cửa sổ hoặc hàng ghế → `bus`; thân hộp nhỏ, kín → `van`.
- Quyết định: `bus` (mã lớp 2).
- Nếu vẫn thiếu bằng chứng: phóng ảnh 100%, đánh dấu `needs_review`, ghi rõ ảnh/hàng và hỏi Lab Coach; không đổi lớp chỉ để khớp nguồn đối chiếu.



### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_008`, hàng 1 trong gói YOLO của tôi.
- Dấu hiệu nhìn thấy: cabin phía trước và thùng hàng lớn, cao, tách biệt rõ ở phía sau.
- Quy tắc áp dụng: có thùng/ben/sàn hàng hóa rõ → `truck`; thân hộp nhỏ, kín một khối → `van`; thân xe con → `car`.
- Quyết định: `truck` (mã lớp 1).
- Nếu vẫn thiếu bằng chứng: kiểm tra đường tách cabin–thùng và toàn bộ phần nhìn thấy ở độ phóng 100%; nếu chưa đủ thì đặt `needs_review` và xin Lab Coach xác nhận.



### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_038`, hàng 4 trong gói CVAT; `xyxy ≈ [74.73, 528.10, 269.77, 640.00]`.
- Dấu hiệu nhìn thấy khi phóng 100%: xe con rõ nhưng phần dưới chạm đúng mép ảnh; tọa độ `ybr=640` bằng chiều cao ảnh.
- Giá trị `visibility`: `clear`.
- Giá trị `boundary`: cần sửa từ `inside` thành `truncated`.
- Trạng thái `review_state`: giữ `needs_review` cho đến khi kiểm lại hộp và lưu bản xuất mới; sau đó mới chuyển `confident`.
- Lý do: “bị mép ảnh cắt” và “bị vật thể khác che” là hai hiện tượng khác nhau. Trường hợp này do mép dưới ảnh cắt, không phải do xe khác che.



## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng bằng số lượng theo từng ảnh và ảnh phủ đối chiếu.
- [ ] Đã kiểm các khác biệt nổi bật, nhưng chưa có bằng chứng đã rà thủ công lớp và hình học của toàn bộ 57 hộp.
- [x] Mỗi hộp trong gói CVAT có đủ ba thuộc tính; tổng số giá trị của mỗi thuộc tính đều là 57.
- [ ] Chưa xử lý xong hộp `needs_review`: gói hiện tại còn 1 hộp tại `drive_038`, hàng 4.
- [ ] Không có bằng chứng rằng ba tình huống trên đã được ghi vào phiếu trước khi xem nguồn đối chiếu; chỉ có bằng chứng các gói nhãn riêng đã được xuất trước khi nhận bộ tham chiếu.
- [x] Không áp dụng làm theo cặp; bài cá nhân đã được xuất và khóa trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: **57** — nằm trong mục tiêu khối lượng 40–60, không được dùng như điểm cắt chất lượng.
