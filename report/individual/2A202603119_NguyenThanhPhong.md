# Báo cáo Day 13 — Nguyễn Thanh Phong, 2A202603119

## 1. Kết luận trước

Mã số 2A202603119, task `D13-2A202603119`. Mười frame: `s01f27`, `s01f55`, `s02f04`, `s02f33`, `s03f03`, `s03f43`, `s04f03`, `s04f39`, `s05f00`, `s05f23`. Làm trên Windows, trình duyệt Edge, CVAT server 2.74.1 (tài khoản `phong`). Chưa ghi MSSV bạn ngồi cạnh.

Bảng A có 111 cuboid. Frame ít nhất là `s03f43` với 5 hộp: cảnh đó ít cụm điểm rõ trong 50 m. Phiếu tự kiểm còn cờ 2/111 hộp. Phần chắc nhất là xe gần: kích thước sedan và `yaw` quanh 270°. Phần yếu là vật xa, đáy hộp `s01f27 #11` và kích thước hộp `s02f04 #12`.

Cách dùng CVAT: kéo chuột trái để xoay khung 3D, kéo chuột phải để dời, lăn chuột để phóng; nhìn từ trên khi các vòng điểm LiDAR gần tròn. Vẽ bằng Shape, nháy đúp để đặt hộp, rồi chỉnh dài và rộng ở Top, đáy và chiều cao ở Side và Front. Bật Cuboid orientation để thấy vạch đỏ dọc thân xe; đối chiếu ảnh camera bằng `python scripts/lab.py preview`.

## 2. Đọc frame (Bước 2)

Ảnh: `t1-s01f27-top`, `t1-s01f55-3d`, `t1-s01f55-top`. Ảnh 3D của frame đầu (`t1-s01f27-3d`) chưa có trong thư mục ảnh.

| Frame    | Phía trước; +x, +y trên ảnh nhìn từ trên                            | Mặt đường                                | Vật xa nhất đọc được                      | Loại vật        | Phía sau / hai bên             | Cụm chưa đọc được                           |
| -------- | ------------------------------------------------------------------- | ---------------------------------------- | ----------------------------------------- | --------------- | ------------------------------ | ------------------------------------------- |
| `s01f27` | Phía trước (−y) sang trái; +y (sau) sang phải; +x (trái) xuống dưới | Khá phẳng, thấp hơn LiDAR khoảng 1,7 m   | Xe tải khoảng 49 m, chỉ còn vài chục điểm | Car, Truck      | Có xe ngay phía sau, `y` dương | Điểm lơ lửng bên phải, không nằm trong hộp  |
| `s01f55` | Phía trước xuống dưới; +y lên trên; +x sang trái                    | Cùng mặt phẳng, đáy xe gần khoảng −1,6 m | Người đi bộ khoảng 41 m, 14 điểm          | Car, Pedestrian | Hàng xe phía sau gốc LiDAR     | Hộp người đi bộ quá ít điểm để thấy rõ dáng |

Chỉ camera cho biết: màu xe, đầu xe hay đuôi xe, và loại xe (xe buýt khác xe con). Chỉ LiDAR cho biết: khoảng cách theo mét, kích thước hộp, và độ cao đáy so với mặt đường.

Tám frame lướt. Phía trước luôn là −y; cạnh nào của ảnh là phía trước thì đổi theo góc xoay camera.

| Frame    | Phía trước | Điều ghi nhận                                 |
| -------- | ---------- | --------------------------------------------- |
| `s02f04` | −y         | Xe buýt gần, phía sau hộp là vùng rất ít điểm |
| `s02f33` | −y         | 7 hộp                                         |
| `s03f03` | −y         | 13 hộp, có xe tải                             |
| `s03f43` | −y         | Chỉ 5 hộp; xe gần nhất cả bài nằm ở frame này |
| `s04f03` | −y         | 13 hộp                                        |
| `s04f39` | −y         | Có xe buýt ở khoảng 40–45 m                   |
| `s05f00` | −y         | 7 hộp, có xe tải gần                          |
| `s05f23` | −y         | 14 hộp, có người đi bộ và xe hai bánh         |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật             | Frame, idx   | Class      | Mũi tên yaw        | Đáy                            | Top/Side/Front và ảnh                              | So với bảng 4.3                                                                      |
| --------------- | ------------ | ---------- | ------------------ | ------------------------------ | -------------------------------------------------- | ------------------------------------------------------------------------------------ |
| (a) xe con gần  | `s01f27 #2`  | Car        | 269,7°, cùng chiều | `z` đáy −1,65 m, chạm đường    | Đã xem đủ ba khung và cam                          | 4,54 × 1,90 × 1,52 m, trong khoảng sedan                                             |
| (b) xe buýt     | `s02f04 #9`  | Bus        | 269,3°, cùng chiều | `z` đáy −1,58 m, gần mặt đường | Đã xem hộp vàng dài và ảnh cam                     | Dài 12,40 m và cao 3,43 m trong khoảng xe buýt; rộng 3,06 m, vượt khoảng 2,50–2,60 m |
| (c) người đi bộ | `s01f55 #13` | Pedestrian | 134,2°             | `z` đáy −1,47 m                | Ba khung có hộp, bên trong gần như không thấy điểm | 0,58 × 0,60 × 1,77 m, trong khoảng người lớn; chỉ 14 điểm nên kích thước là suy luận |

Vòng hiệu chuẩn nằm ở task chung, không vào bảng A này. Không lưu được chỗ lệch so với cả lớp. Vật bỏ qua: cụm xa hơn 50 m và cụm điểm lạ không có hình xe hay người (LOW_VISIBILITY / không gán thành vật).

## 4. Bốn hiện tượng chất lượng (Bước 4)

Nguyên nhân dưới đây là giả thuyết. Cả bốn ảnh lấy từ task của tôi.

| Hiện tượng      | Frame / ảnh                   | Nguyên nhân (giả thuyết)                               | Thấy gì trên point cloud                                                          | Hệ quả khi gán                                                                              |
| --------------- | ----------------------------- | ------------------------------------------------------ | --------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| 1. Bóng che     | `s02f04`, `t4-1-bong-che`     | Thân xe buýt chặn tia LiDAR                            | Ngay sau hộp xe buýt là vùng tối, trong khi mặt đường hai bên vẫn có điểm         | Bật occluded nếu có vật bị che; hộp bao phần suy ra được; vùng trống không phải đường trống |
| 2. Thưa ở xa    | `s01f27`, `t4-2-thua-o-xa`    | Càng xa càng ít tia trả về                             | Hộp xe chỉ còn vài điểm bên trong. Trên bảng, `s01f27 #12` cách 47,8 m có 34 điểm | Vẽ khi còn nhận ra xe; không thì bỏ và ghi LOW_VISIBILITY                                   |
| 3. Phản xạ thấp | `s01f27`, `t4-3-phan-xa-thap` | Bề mặt tối hoặc nhiều kính trả ít điểm hơn xe cùng tầm | `s01f27 #11` cách 16,6 m chỉ có 247 điểm, trong khi `#3` cách 12,0 m có 1415 điểm | Giữ giả thuyết, không nới hộp theo chỗ không có điểm                                        |
| 4. Điểm lạ      | `t4-4-diem-la`                | Nhiễu hoặc vật mỏng không đủ để gọi là xe              | Cụm điểm nằm trên mặt đường, không nằm trong hộp nào                              | Không gán thành vật                                                                         |

## 5. Số từ cuboid của tôi (Bước 5)

Mặt đường khoảng −1,7 m. `z` đáy = `z − h/2`. `r` là khoảng cách 3D tới tâm hộp.

| Cuboid           | z (m) | h (m) | z đáy (m) | dist_m | r (m) | yaw (°) | Nhận xét                                             |
| ---------------- | ----: | ----: | --------: | -----: | ----: | ------: | ---------------------------------------------------- |
| `s01f27 #2` Car  | −0,89 |  1,52 |     −1,65 |   6,76 |  6,82 |   269,7 | Chạm đường                                           |
| `s01f27 #11` Car | −1,17 |  1,38 |     −1,86 |  16,58 | 16,62 |    86,2 | Chìm: `ground_gap_m` −0,29 m                         |
| `s02f04 #7` Car  |  1,11 |  1,86 |      0,18 |  32,57 | 32,59 |    72,8 | Lơ lửng khoảng 1,9 m; phiếu không ước được mặt đường |

Gần nhất: `s03f43 #3` Car, `dist_m` 3,31 m, `r` 3,40 m, 4203 điểm, `yaw` 269,3°. Xa nhất: `s01f27 #16` Truck, `dist_m` 48,98 m, `r` 48,99 m, 38 điểm; `z` đáy −0,35 m nên hộp này đang lơ lửng.

Người đi bộ `s01f55 #13`: 0,58 × 0,60 × 1,77 m, nằm trong khoảng người lớn của bảng 4.3. Xe con `s01f27 #2`: 4,54 × 1,90 × 1,52 m, trong khoảng sedan. Xe cùng chiều phía trước `s01f55 #2` có `yaw` 270,3°, sát 270°. Xe tải `s01f27 #15` có `yaw` 90,1°, sát 90° của hướng ngược lại.

Mật độ trên `s01f27`:

| Cuboid   | dist_m | Số điểm |
| -------- | -----: | ------: |
| `#2` Car |   6,76 |    1976 |
| `#3` Car |  11,99 |    1415 |
| `#5` Car |  20,39 |     189 |
| `#4` Car |  40,58 |      80 |

Từ 6,8 m đến 40,6 m, xa hơn khoảng 6 lần thì số điểm còn khoảng 1/25. Cùng hướng với đường cong trên slide: xe con khoảng 1968 điểm ở 0–10 m và khoảng 72 điểm ở 30–40 m.

Đã sửa hai hộp sau phiếu tự kiểm. `s02f04 #11` Cyclist hết cờ `length < width`: nay dài 2,09 m, rộng 1,20 m, cao 2,04 m, `yaw` 0°. `s02f04 #7` Car thu rộng từ 2,35 m xuống 2,30 m nên hết cờ bề rộng; đáy vẫn ở +0,18 m. Hai cờ giữ lại: `s01f27 #11` vì đáy chỉ thấp hơn ngưỡng 0,25 m khoảng 4 cm; `s02f04 #12` vẫn sai kích thước (dài 2,80 m, rộng 5,43 m, `yaw` 0°) và cần sửa tiếp trên CVAT.

Đổi kiểm với MSSV 2A202603210, chỉ xem bảng A trên màn hình. Hai số của mình so với bảng A của mình: `s01f27 #2` có `dist_m` 6,76 m và `s03f43 #3` có `dist_m` 3,31 m, đều khớp. Hai số của họ so với bảng A của họ: `s01f30 #1` Car có `h` 1,603 m và `dist_m` 8,02 m, đều khớp.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Nhãn 3D dùng mét vì model phải học vị trí và kích thước thật của vật trong không gian, không phụ thuộc độ phân giải ảnh. Vật thưa điểm cần quy tắc riêng vì vài chục điểm không đủ để đo hết thân xe; đoán kích thước cho đủ hộp sẽ dạy model một hình sai.

Có dùng AI để hỗ trợ. Câu hỏi chính là đọc cờ của phiếu tự kiểm, đối chiếu số trong bảng A và viết báo cáo từ các số đó. Kết quả dùng để chỉ hộp cần xem lại và để điền mục 5; hộp nào giữ hay sửa vẫn quyết theo point cloud trên CVAT.

Còn chưa chắc: đáy `s02f04 #7` và `s01f27 #16` có đang lơ lửng thật hay mặt đường quanh đó dốc; hộp `s02f04 #12` chưa được xoay lại; rộng xe buýt `s02f04 #9` vượt bảng 4.3 khoảng 0,5 m.
