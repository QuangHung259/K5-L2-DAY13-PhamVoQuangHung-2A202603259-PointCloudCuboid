# Báo cáo Day 13 — Phạm Võ Quang Hưng, 2A202603259

## 1. Kết luận trước
- Mã số: 2A202603259 · Task: `D13-2A202603259`
- 10 frame: s01f07, s01f25, s02f02, s02f19, s03f09, s03f52, s04f02, s04f37, s05f00, s05f31
- Hệ điều hành: Windows 11 · Trình duyệt: Chrome · CVAT: 2.81.0 (máy cá nhân, cổng 8081)
- MSSV bạn ngồi cạnh: 2A202603247

Tổng cộng tôi đã vẽ **67 cuboid** trên 10 frame (`s01f07`: 10, `s01f25`: 6, `s02f02`: 5, `s02f19`: 7, `s03f09`: 7, `s03f52`: 5, `s04f02`: 10, `s04f37`: 6, `s05f00`: 5, `s05f31`: 6); trong đó các frame `s02f02`, `s03f52` và `s05f00` có ít cuboid nhất (5 cuboid mỗi frame) vì mật độ phương tiện quanh xe trong phạm vi ưu tiên 30 m thưa hơn và một số cụm điểm ở xa bị khuất sau xe lớn không đủ 15–20% thể tích để gán nhãn. Phiếu tự kiểm `selfcheck` chỉ gắn cờ **1/67 cuboid** (`s03f09`, idx 7, class `Cyclist`). Tôi tin tưởng nhất vào việc phân loại `class`, xác định hướng `yaw` và căn đáy `z_bottom` chạm mặt đường (quanh −1,7 m) cho các phương tiện ở cự ly gần và trung bình (0–25 m); phần còn yếu nhất là ước lượng chiều dài thực và tâm của các vật ở xa (30–40 m) hoặc bị bóng che khi point cloud chỉ còn điểm ở một mặt phản xạ.

Cách dùng CVAT:
- **Đổi góc nhìn**: Trong khung 3D (Perspective), kéo chuột trái để xoay góc nhìn (xoay thẳng từ trên xuống để quan sát toàn cảnh BEV 360°), kéo chuột phải để dời (pan), lăn con lăn chuột để phóng to/thu nhỏ và nháy đúp vào khoảng trống để về góc nhìn mặc định.
- **Vẽ và chỉnh một cuboid**: Chọn công cụ cuboid ở thanh bên trái → chọn class → bấm **Shape** (không dùng Track) rồi đưa con trỏ tới cụm điểm của vật và nháy đúp để đặt hộp; sau đó dùng khung **Top** để dời tâm (`x, y`), kéo cạnh chỉnh dài/rộng (`l, w`) và xoay góc `yaw`, dùng khung **Side** và **Front** để hạ đáy chạm mặt đường và chỉnh chiều cao (`h`) (không xoay ở Side/Front).
- **Bật mũi tên hướng và xem trên ảnh camera**: Mở **Appearance → Cuboid orientation** để hiện trục hướng của hộp (căn vạch đỏ ngắn `+X` dọc thân xe chỉ về đầu xe hoặc hướng người di chuyển), đối chiếu với 7 góc ảnh camera bên phải và chạy `python scripts/lab.py preview` để kiểm tra hình chiếu 3D của cuboid lên ảnh camera trong `work/xem/`.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f07-3d.png`, `t1-s01f07-top.png`, `t1-s01f25-3d.png`, `t1-s01f25-top.png` (ảnh nhìn từ trên xuống, có đánh dấu phía trước `−y`, `+x`, `+y`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f07` | Phía trước xe (`−y`, mũi tên xanh dương) chỉ về **cạnh phải** của ảnh; trục `+x` (vạch cam, bên trái xe) chỉ **lên cạnh trên**; trục `+y` (vạch xanh lá, phía sau xe) chỉ **sang cạnh trái** | Tương đối phẳng theo chiều dọc (làn ngược chiều bên trái thấp hơn nhẹ ~0,2–0,3 m); thấp hơn LiDAR khoảng **1,7 m** (`z_bottom` quanh −1,56 m đến −2,18 m) | Khoảng **38,1 m** (xe con `idx 10` ở `dist_m = 38,09 m`; các cấu trúc tòa nhà/hàng cây nhận ra tới ~45–50 m) | Xe con (`Car`) cùng chiều và ngược chiều, xe tải/bán tải (`Truck`), dải phân cách cây bụi giữa đường, hàng cây và mặt tiền tòa nhà hai bên | Ngay sát bên trái có xe con màu xanh (`idx 1`, `dist_m = 3,60 m`); phía sau có 2 xe con cùng chiều (`idx 3` ở `18,83 m`, `idx 4` ở `32,59 m`); bên trái và sau-trái ở làn ngược chiều có xe tải (`idx 8`, `15,96 m`) và các xe con (`idx 5` ở `18,24 m`, `idx 9` ở `29,31 m`, `idx 10` ở `38,09 m`) | Các cụm điểm thưa thớt nằm khuất sau hàng cây vỉa hè hai bên và các vệt điểm rời rạc ở cự ly > 45 m phía trước/phía sau không rõ hình dáng xe hay vật kiến trúc |
| `s01f25` | Phía trước xe (`−y`, mũi tên xanh dương) chỉ về **cạnh phải (hơi chếch lên trên)**; trục `+x` (vạch cam) chỉ **lên cạnh trên (hơi lệch trái)**; trục `+y` (vạch xanh lá) chỉ **về cạnh trái (hơi chếch xuống dưới)** | Phẳng tại khu vực nút giao; thấp hơn LiDAR khoảng **1,7 m** (`z_bottom` của các xe từ −1,60 m đến −1,90 m) | Khoảng **18,5 m** đối với xe gán nhãn chắc chắn (`idx 6` ở `dist_m = 18,52 m`), và đọc được hình khối xe/vật cản ở xa khoảng **30–35 m** quanh giao lộ | Nhiều xe con (`Car`) đang đi thẳng và chuyển hướng/rẽ qua giao lộ (`yaw` = 270,0°, 91,3°, 225,2°, 348,9°), dải phân cách, cột đèn và cây xanh | Phía sau bên phải có 2 xe con (`idx 1` ở `dist_m = 6,80 m` và `idx 3` đang rẽ ở `7,65 m`); phía trước bên phải có xe con đang nhập làn (`idx 4`, `12,79 m`); bên trái có xe ngược chiều (`idx 5`, `17,21 m`); phía sau có xe cùng chiều (`idx 6`, `18,52 m`) | Cụm điểm lộn xộn gần góc giao lộ bên phải phía sau rào/bụi cây ở khoảng cách > 25–30 m bị che khuất, quá ít điểm để kết luận là xe đỗ hay vật thể cố định |

Ba điều **chỉ camera** cho biết và ba điều **chỉ LiDAR** cho biết (lấy từ hai frame `s01f07` và `s01f25`):
- **Chỉ camera cho biết**:
  1. Màu sắc, chất liệu sơn và chi tiết ngoại thất giúp nhận diện chính xác từng xe (như chiếc sedan màu xanh dương sát bên trái ở `s01f07` trên `cam05`/`cam06`, hay các xe màu trắng, đen, đỏ trên `cam01`).
  2. Phân biệt chắc chắn đầu xe (đèn pha, lưới tản nhiệt, kính chắn gió trước) với đuôi xe (đèn hậu) của các xe ở xa hoặc đang xoay chéo góc tại giao lộ `s01f25` khi cụm điểm LiDAR chỉ hiện một cạnh chữ L.
  3. Hệ thống vạch kẻ phân làn, mũi tên chỉ hướng sơn phẳng trên mặt đường, đèn tín hiệu giao thông và bối cảnh thị giác ở cự ly rất xa ngoài vùng điểm LiDAR đậm.
- **Chỉ LiDAR cho biết**:
  1. Tọa độ không gian 3 chiều (`x, y, z`) và khoảng cách thực tính bằng mét (`dist_m`) tới mọi vật trong 360° quanh xe (ví dụ đo chính xác xe `idx 1` cách 3,60 m và xe `idx 10` cách 38,09 m ở `s01f07`).
  2. Kích thước hình học thật (`length, width, height`) và cao độ mặt đường (`z_bottom ≈ -1,7 m`) của từng phương tiện mà không bị biến dạng phối cảnh gần to – xa nhỏ như ảnh 2D.
  3. Sự hiện diện và vị trí chính xác của các vật nằm ở vùng rìa hoặc góc mù giữa các khung hình quạt của 7 camera nhờ chùm tia quét 360° liên tục.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f02` | Cạnh phải (khi đặt trục `+x` hướng lên cạnh trên; tương ứng hướng `−y`) | Có xe buýt rất lớn (`Bus`, dài 12,058 m) áp sát ngay bên trái (`dist_m = 3,37 m`, 16317 điểm) chắn tia LiDAR tạo vùng bóng che lớn phía sau bên trái; có 1 `Cyclist` ở xa `34,84 m` rất thưa (33 điểm) |
| `s02f19` | Cạnh phải (khi đặt trục `+x` hướng lên cạnh trên; tương ứng hướng `−y`) | Xuất hiện cả người đi bộ (`Pedestrian`, `dist_m = 8,68 m`) và người đi xe hai bánh (`Cyclist`, `dist_m = 15,59 m`) ở phía trước bên phải cùng dòng xe con hai chiều |
| `s03f09` | Cạnh phải (khi đặt trục `+x` hướng lên cạnh trên; tương ứng hướng `−y`) | Hai bên đường có tường/rào chắn công trình; các xe con chạy bám đuôi gần nhau (4,00 m – 11,53 m) gây bóng che lẫn nhau; có 1 `Cyclist` bên phải (`dist_m = 13,62 m`) có đáy cao hơn mặt đường trung bình 0,54 m |
| `s03f52` | Cạnh phải (khi đặt trục `+x` hướng lên cạnh trên; tương ứng hướng `−y`) | Có xe buýt dài 12,345 m ở làn ngược chiều phía trước bên trái (`dist_m = 23,13 m`) và 1 `Cyclist` ở phía trước bên phải (`dist_m = 9,69 m`) |
| `s04f02` | Cạnh phải (khi đặt trục `+x` hướng lên cạnh trên; tương ứng hướng `−y`) | Có cụm 5 người đi bộ (`Pedestrian`, `idx 5` đến `idx 9`) đi gần nhau ở phía sau bên phải (`dist_m` từ 17,14 m đến 23,49 m) với số điểm thưa (59–101 điểm) và biển báo bên trái có cường độ phản xạ (`intensity`) rất cao |
| `s04f37` | Cạnh phải (khi đặt trục `+x` hướng lên cạnh trên; tương ứng hướng `−y`) | Có xe tải lớn (`Truck`, dài 6,621 m, cao 2,736 m) ở làn ngược chiều khá gần (`dist_m = 7,72 m`, 6310 điểm) và 2 `Cyclist` ở hai phía đường (`13,19 m` và `14,23 m`) |
| `s05f00` | Cạnh phải (khi đặt trục `+x` hướng lên cạnh trên; tương ứng hướng `−y`) | Có xe tải cao (`Truck`, `idx 4`, cao 3,124 m) ngay phía sau (`dist_m = 9,72 m`, 3696 điểm); xe con `idx 5` bên trái (`11,80 m`) nằm ở vùng mặt đường bị dải phân cách che nên `ground_gap_m` để trống |
| `s05f31` | Cạnh phải (khi đặt trục `+x` hướng lên cạnh trên; tương ứng hướng `−y`) | Dải phân cách giữa đường có hàng cây bụi dày cản tia quét tầm thấp sang làn bên cạnh; có các cụm điểm lẻ tẻ/điểm lạ trên dải phân cách và mặt đường |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f07`, idx 1 | `Car` | Đúng (`yaw_deg = 270,0°`, mũi tên đỏ `+X` hướng thẳng về đầu xe theo chiều `−y`) | Chạm đường (`z_bottom = -1,704 m`, `ground_gap_m = 0,08 m`) | Đã kiểm đủ Top/Side/Front và đối chiếu `cam05`, `cam06` | Trong bảng 4.3 (`l = 4,599 m`, `w = 1,808 m`, `h = 1,405 m`; chuẩn Sedan: dài 3,8–5,0 m, rộng 1,70–1,90 m, cao 1,40–1,60 m) |
| (b) xe buýt hoặc xe tải | `s02f02`, idx 1 (và `s01f07`, idx 8) | `Bus` (và `Truck`) | Đúng (`Bus`: `yaw_deg = 270,8°` cùng chiều; `Truck` ở `s01f07` idx 8: `yaw_deg = 91,0°` ngược chiều) | Chạm đường (`Bus`: `z_bottom = -1,770 m`, `ground_gap_m = 0,07 m`; `Truck`: `z_bottom = -1,958 m`, `ground_gap_m = 0,22 m`) | Đã kiểm đủ Top/Side/Front và đối chiếu `cam05`, `cam06` | `Bus` có `l = 12,058 m` (trong 10,0–12,5 m), `h = 2,921 m` (sát mức 3,00–3,50 m), `w = 2,974 m` (rộng hơn 2,50–2,60 m do bao cả gương chiếu hậu lớn); `Truck` có `l = 5,300 m, w = 1,992 m, h = 2,400 m` khớp bảng 4.3 |
| (c) người đi bộ hoặc xe hai bánh | `s02f19`, idx 6 (và `s02f02`, idx 5) | `Pedestrian` (và `Cyclist`) | Đúng (`Pedestrian`: `yaw_deg = 84,0°` theo hướng bước đi; `Cyclist` ở `s02f02` idx 5: `yaw_deg = 270,1°`) | Chạm đường (`Pedestrian`: `z_bottom = -1,511 m`, `ground_gap_m = 0,25 m`; `Cyclist`: `z_bottom = -1,764 m`, `ground_gap_m = 0,07 m`) | Đã kiểm đủ Top/Side/Front và đối chiếu `cam01`, `cam07` | `Pedestrian` có `l = 0,518 m` (trong 0,4–0,9 m), `w = 0,590 m` (trong 0,5–0,9 m), `h = 1,460 m` (thấp hơn nhẹ ngưỡng 1,50 m khoảng 0,04 m do dáng bước đi và thiếu điểm đỉnh đầu) |

**Vòng hiệu chuẩn và các vật bỏ qua vì không đủ cơ sở:**
- **Vòng hiệu chuẩn**: Khi so kết quả trên task chung với các bạn trong lớp, cuboid của tôi ban đầu lệch nhẹ ở kích thước chiều dài/rộng (khoảng 0,1–0,2 m do cách xử lý phần gương xe và đuôi xe bị khuất điểm) cùng góc `yaw` lệch vài độ. Cả lớp đã cùng coach chốt lại cách hiểu guideline: cuboid phải bao trọn toàn bộ điểm thuộc vật (kể cả gương chiếu hậu và phần thân bị khuất suy luận được từ ảnh camera + bảng 4.3, nhưng không bao dư điểm nhiễu mặt đường), đáy phải kiểm tra ở cả Side và Front để chạm sát mặt đường (`z_bottom ≈ -1,7 m`), và vạch đỏ `+X` phải chỉ đúng về đầu xe/hướng di chuyển.
- **Các vật bỏ qua vì không đủ cơ sở**:
  - `s01f07` (phía trước và phía sau ở khoảng cách > 40–48 m, gần ranh giới 50 m): một số cụm điểm xe ở rất xa chỉ còn vài điểm rời rạc ở đuôi/đầu xe, dưới 15–20% thể tích và trên ảnh camera quá nhỏ không suy luận được kích thước chuẩn → bỏ qua (`LOW_VISIBILITY` / `UNCERTAIN_SCOPE`).
  - `s02f02` (phía sau bên trái xe buýt `idx 1`, khoảng cách ~10–25 m): vùng bị thân xe buýt dài 12,058 m che khuất gần như hoàn toàn tia LiDAR, chỉ có vài điểm lẻ tẻ không rõ ranh giới vật → bỏ qua (`LOW_VISIBILITY`).
  - `s05f31` (làn bên trái phía sau dải phân cách cây bụi dày, khoảng cách ~25–35 m): cụm điểm bị tán cây che khuất nặng, không phân biệt được rõ ràng với điểm cây xanh → bỏ qua (`LOW_VISIBILITY`).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Cả 4 hiện tượng đều lấy trực tiếp từ các frame thuộc task cá nhân `D13-2A202603259`.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s03f09` / `t4-1-bong-che.png` / Task cá nhân `D13-2A202603259` | **Giả thuyết**: Tia laser từ LiDAR truyền theo đường thẳng nên bị thân các xe con ở gần (`idx 1, 2, 3` ở khoảng cách 4,00–6,46 m) chắn đường truyền, không xuyên qua được vật cản đặc để tới phần thân dưới của các xe phía sau (`idx 4, 5, 6`). | Phía sau mỗi xe gần xuất hiện một dải quạt tối hoàn toàn không có điểm mặt đường; các xe bị che phía sau bị khuyết mất phần đầu hoặc nửa thân dưới, chỉ còn cụm điểm ở phần nóc và sườn trên. | Nếu vật bị che vẫn nhìn thấy ≥ 15–20% thể tích hoặc đủ cơ sở suy luận từ ảnh camera (`cam01`, `cam09`) thì vẽ cuboid bao trọn cả phần bị che suy luận được (dựa theo kích thước chuẩn bảng 4.3) và bật thuộc tính `occluded`; không co hộp theo riêng phần điểm còn sót lại. |
| 2. Thưa ở xa | `s04f02` / `t4-2-thua-o-xa.png` / Task cá nhân `D13-2A202603259` | **Giả thuyết**: Các chùm tia quét LiDAR phân kỳ theo góc đứng và góc ngang khi truyền ra xa, làm khoảng cách giữa hai tia liền kề mở rộng theo khoảng cách khiến số tia chạm vào vật có tiết diện nhỏ giảm nhanh. | Nhóm người đi bộ phía sau bên phải ở cự ly 17,14–23,49 m (`idx 5` đến `idx 9`) chỉ còn từ 101 điểm giảm xuống 59 điểm (các đường quét cắt ngang người cách thưa nhau), còn các xe ở cự ly > 30–35 m phía xa chỉ còn vài vệt điểm mỏng ở một mặt. | Áp dụng quy tắc 15–20%: chỉ vẽ cuboid khi còn thấy khoảng 15–20% thể tích hoặc đối chiếu ảnh camera (`cam09`) nhận diện rõ hình dạng và dùng bảng 4.3 để giữ kích thước hợp lý; nếu quá thưa không đủ cơ sở suy luận thì bỏ qua và ghi `LOW_VISIBILITY`. |
| 3. Phản xạ thấp / ít điểm bất thường | `s04f02` / `t4-3-phan-xa-thap.png` / Task cá nhân `D13-2A202603259` | **Giả thuyết**: Bề mặt sơn xe tối màu hấp thụ bước sóng hồng ngoại của LiDAR nhiều hơn, đồng thời mặt kính chắn gió/cửa sổ đặt nghiêng làm tia laser phản xạ gương lệch đi hướng khác thay vì quay về đầu thu (ngược hẳn với biển báo phản quang bên trái có `intensity` rất cao). | Trên hai xe con phía trước (`idx 2` ở 15,50 m và `idx 3` ở 16,46 m), vùng kính hậu và phần thân trên tối màu bị rỗng điểm bất thường dù ở cự ly trung bình, điểm chỉ tập trung ở cản sau, biển số và phần đuôi thẳng đứng. | Chỉ ghi nhận nguyên nhân dưới dạng **giả thuyết** (không khẳng định tuyệt đối); khi gán nhãn không được ép chiều cao hay chiều dài hộp nhỏ lại theo vùng có điểm ở cản dưới mà phải đối chiếu `cam01`/`cam03` và bảng 4.3 để dựng hộp bao đủ toàn bộ thân và mui xe thực tế. |
| 4. Điểm lạ | `s05f31` / `t4-4-diem-la.png` / Task cá nhân `D13-2A202603259` | **Giả thuyết**: Nhiễu đa đường truyền (multipath reflection) khi tia laser phản xạ qua mặt kính/kim loại hoặc tán xạ qua bụi cây ở dải phân cách, bụi lơ lửng trong không khí và nhiễu điện tử của cảm biến. | Xuất hiện các cụm điểm rời rạc lơ lửng hoặc nằm lẻ loi quanh dải phân cách và mặt đường, không tạo thành bề mặt hình học của bất kỳ phương tiện hay người đi bộ nào khi đối chiếu với `cam03`, `cam05`. | Tuyệt đối **không gán thành vật** (không vẽ cuboid cho cụm điểm lạ); nếu điểm nhiễu hay quầng sáng nằm sát cạnh một xe thật thì cũng không nới rộng cuboid của xe theo các điểm lạ đó. |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603259-bangA.csv`; số điểm lấy từ cột `points_in` của `submission/D13-2A202603259-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f07`, idx 1, `Car` | −1,002 m | 1,405 m | −1,704 m | 3,60 m | 3,74 m | 270,0° | **Chạm đường** (z đáy = −1,704 m, chỉ lệch 0,004 m so với mốc −1,7 m; `ground_gap_m = 0,08 m`) |
| `s02f02`, idx 1, `Bus` | −0,310 m | 2,921 m | −1,770 m | 3,37 m | 3,38 m | 270,8° | **Chạm đường** (z đáy = −1,770 m, lệch 0,070 m so với mốc −1,7 m, nhỏ hơn nhiều ngưỡng 0,3 m; `ground_gap_m = 0,07 m`) |
| `s04f37`, idx 1, `Truck` | −0,377 m | 2,736 m | −1,745 m | 7,72 m | 7,73 m | 91,2° | **Chạm đường** (z đáy = −1,745 m, lệch 0,045 m so với mốc −1,7 m; `ground_gap_m = 0,08 m`) |

- **(a) Cuboid gần nhất và xa nhất**:
  - **Gần nhất**: `s02f02`, idx 1, class `Bus` (`x = 3,365 m`, `y = -0,168 m`, `z = -0,310 m`) có khoảng cách ngang `dist_m = 3,37 m` và khoảng cách 3D `r = √(3,365² + (-0,168)² + (-0,310)²) = 3,38 m`.
  - **Xa nhất**: `s01f07`, idx 10, class `Car` (`x = 9,331 m`, `y = 36,928 m`, `z = -0,895 m`) có khoảng cách ngang `dist_m = 38,09 m` và khoảng cách 3D `r = √(9,331² + 36,928² + (-0,895)²) = 38,10 m`.
- **(b) Kích thước `l, w, h` của một người đi bộ ( và xe hai bánh) và một xe con so với bảng 4.3**:
  - **Người đi bộ** (`s02f19`, idx 6, `Pedestrian`): `l = 0,518 m`, `w = 0,590 m`, `h = 1,460 m`. So với bảng 4.3 (Người lớn: dài 0,4–0,9 m, rộng 0,5–0,9 m, cao 1,50–1,95 m), chiều dài và chiều rộng nằm gọn trong khoảng chuẩn, riêng chiều cao `h = 1,460 m` thấp hơn ngưỡng dưới 1,50 m là `0,040 m` vì người thật có vóc dáng hơi thấp/khom khi bước đi và tia quét LiDAR ở khoảng cách 8,68 m không cắt trúng điểm cao nhất trên đỉnh đầu. (Tương tự với xe hai bánh `s02f19`, idx 7, `Cyclist`: `l = 1,379 m`, `w = 1,075 m`, `h = 1,553 m`, chiều cao `1,553 m` nằm trong khoảng 1,40–1,80 m của xe máy có người, nhưng `l = 1,379 m` ngắn hơn khoảng 1,6–2,3 m do bánh trước/sau bị khuyết điểm ở cự ly 15,59 m và `w = 1,075 m` vượt nhẹ 0,95 m do bao cả phần tay lái/chân người ngồi).
  - **Xe con** (`s03f09`, idx 5, `Car`): `l = 4,240 m`, `w = 2,130 m`, `h = 1,454 m` (và `s01f07`, idx 1, `Car`: `l = 4,599 m`, `w = 1,808 m`, `h = 1,405 m` có cả 3 chiều nằm trọn trong khoảng Sedan 3,8–5,0 m × 1,70–1,90 m × 1,40–1,60 m). Ở xe `s03f09` idx 5, chiều dài `4,240 m` và chiều cao `1,454 m` nằm trong khoảng chuẩn của Car, còn chiều rộng `w = 2,130 m` vượt nhẹ ngưỡng trên 2,10 m của bảng 4.3 (`0,030 m`) vì hộp được kéo bao trọn cả hai gương chiếu hậu nhô ra hai bên thân xe theo đúng quy tắc hình dạng cuboid.
- **(c) `yaw_deg` của một xe cùng chiều phía trước và một xe ngược chiều**:
  - **Xe cùng chiều phía trước**: `s01f07`, idx 2, `Car` (`x = 2,319 m`, `y = -19,057 m`) có `yaw_deg = 270,0°` (hoặc `s02f02`, idx 2, `Car` tại `y = -14,837 m` có `yaw_deg = 267,6°`) — rất gần **270°** vì xe hướng thẳng theo chiều tiến `−y` của hệ tọa độ VF.
  - **Xe ngược chiều**: `s01f07`, idx 5, `Car` (`x = 18,104 m`, `y = -2,196 m`) có `yaw_deg = 91,0°` (hoặc `s03f52`, idx 4, `Bus` có `yaw_deg = 89,7°`) — rất gần **90°** vì xe di chuyển ngược lại theo chiều `+y`.

Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau trải đều 4 khoảng cách):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f07`, idx 1, `Car` | 3,60 m (`3,6 m`) | 3159 điểm |
| `s01f25`, idx 1, `Car` | 6,80 m (`6,8 m`) | 1896 điểm |
| `s04f02`, idx 2, `Car` | 15,50 m (`15,5 m`) | 728 điểm |
| `s01f07`, idx 9, `Car` | 29,31 m (`29,3 m`) | 219 điểm |
| `s01f07`, idx 10, `Car` | 38,09 m (`38,1 m`) | 89 điểm |

- **Quy luật và so sánh với slide bài giảng**:
  - **Quy luật**: Khi khoảng cách từ LiDAR tới xe tăng gấp khoảng **2 lần** thì số điểm LiDAR (`points_in`) giảm mạnh chỉ còn khoảng **1/4 đến 1/3** (ví dụ từ `6,80 m` lên `15,50 m` số điểm giảm từ 1896 xuống 728 điểm; từ `15,50 m` lên `29,31 m` giảm từ 728 xuống 219 điểm; từ `19,20 m` với 375 điểm lên `38,09 m` chỉ còn 89 điểm) do sự phân kỳ hình học hai chiều của chùm tia laser.
  - **So với đường cong ở slide bài giảng** (xe con trung vị: ~1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m, 19 ở 60–80 m): các cuboid của tôi bám rất sát đường cong chuẩn ở cả 4 phân đoạn (`6,80 m` có 1896 điểm ≈ 1968; `15,50 m` có 728 điểm ≈ 718; `29,31 m` có 219 điểm ≈ 214; `38,09 m` có 89 điểm ≈ 72).
- **Tự kiểm và chỉnh sửa sau `preview` / `selfcheck`**:
  - **Hai cuboid đã sửa**:
    1. `s03f09`, idx 6, `Car` (`dist_m = 11,53 m`): Ở lần chạy `selfcheck` đầu tiên bị cảnh báo *"787 điểm sát ngay ngoài mép (quá chật, hay vật bên cạnh?)"* vì hộp dài 3,693 m bị hụt phần thân xe (chỉ bao được 341 điểm). Sau khi kiểm tra lại trên CVAT và ảnh `preview`, tôi đã kéo dài `length` lên `4,020 m`, chỉnh `width = 1,829 m`, `height = 1,907 m` và góc `yaw_deg = 269,1°`, nâng `points_in` lên 1341 điểm và khắc phục hoàn toàn cảnh báo.
    2. `s03f52`, idx 4, `Bus` (`dist_m = 23,13 m`): Ở lần chạy `selfcheck` đầu tiên bị cảnh báo *"width=3.73 ngoài khoảng 2.0–3.2 của Bus"* do cạnh ngang hộp bị kéo dư sang khoảng trống bên hông xe. Tôi đã thu hẹp `width` từ `3,731 m` về `3,200 m` và dời tâm `x` từ `16,009 m` về `15,591 m` cho ôm sát mặt phản xạ của xe buýt, sau khi chạy lại `sync` đã hết cảnh báo. (Ngoài ra tôi cũng tinh chỉnh nhẹ kích thước của `s03f09` idx 3 và idx 5 cho khít điểm hơn).
  - **Cảnh báo giữ nguyên kèm lý do**:
    - Giữ nguyên 1 cảnh báo tại `s03f09`, idx 7, `Cyclist` (`dist_m = 13,62 m`, `z = -0,509 m`, `h = 1,401 m`, `z_bottom = -1,210 m`, `ground_gap_m = 0,54 m`) với cờ *"đáy cao hơn mặt đường 0.54 m (lơ lửng?)"*. Lý do: Khi kiểm tra lại trên khung 3D và ảnh camera, người đi xe hai bánh này di chuyển ở khu vực lề/nền đường bên phải (`x = -8,670 m`, `y = -10,509 m`) có cốt cao độ cao hơn mặt đường chỗ xe thu dữ liệu và phần bánh dưới bị khuất tia quét; đáy cuboid đã chạm sát điểm thấp nhất của cụm điểm vật thể nên tôi giữ nguyên theo thực tế point cloud thay vì kéo đáy chìm xuống chỉ để tắt cờ.
- **Đổi kiểm với bạn bên cạnh**:
  - MSSV người đã kiểm chéo: **2A202603247**.
  - Hai con số được bạn chọn đối chiếu trực tiếp với Bảng A trên màn hình máy tôi:
    1. Khoảng cách ngang `dist_m = 3,37 m` (cùng độ cao đáy `z_bottom = -1,770 m`) của cuboid gần nhất `s02f02`, idx 1, `Bus`.
    2. Khoảng cách ngang `dist_m = 38,09 m` (cùng góc hướng `yaw_deg = 91,0°`) của cuboid xa nhất `s01f07`, idx 10, `Car`.
  - Kết quả đối chiếu: **Khớp chính xác 100%** với dữ liệu trong file `submission/D13-2A202603259-bangA.csv`, không có sai lệch cần sửa.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa từ phía model học trên nhãn này**:
  - Model phát hiện vật thể 3D trên xe tự hành phải dự đoán trực tiếp vị trí, kích thước thực và khoảng cách của chướng ngại vật trong không gian vật lý để hệ thống lập kế hoạch chuyển động tính toán quỹ đạo tránh va chạm và quãng đường phanh an toàn, nên nhãn 3D bắt buộc phải đo bằng **mét (m)** (bất biến theo khoảng cách) thay vì **pixel** trên ảnh 2D vốn bị co giãn theo phép chiếu phối cảnh gần to – xa nhỏ.
  - Với các vật ở xa hoặc bị che khuất có mật độ điểm rất thưa, cần quy tắc riêng (ngưỡng 15–20% thể tích kết hợp kích thước tham chiếu bảng 4.3 hoặc bỏ qua khi `LOW_VISIBILITY`) để ngăn annotator đoán mò hoặc co nhỏ hộp theo vài điểm lẻ tẻ, tránh đưa nhãn nhiễu (noisy/inconsistent ground-truth) vào tập huấn luyện làm model học sai phân bố kích thước vật lý và tăng tỷ lệ báo động giả (false positives).
- **(b) Cách dùng AI**:
  - Công cụ: Trợ lý AI (Antigravity / LLM).
  - Câu hỏi chính: Hỏi cách kiểm tra định dạng báo cáo Markdown, cách chạy các lệnh `python scripts/lab.py sync`, `pack`, `check_submission.py` trên Windows PowerShell và giải thích ý nghĩa các cột chỉ số kiểm tra trong phiếu tự kiểm.
  - Cách dùng kết quả: Dùng để rà soát lỗi định dạng báo cáo, kiểm tra tính nhất quán của đơn vị đo (`m`, `°`) và quy trình đóng gói nộp bài theo đúng hướng dẫn `SUBMISSION.md`.
- **(c) Điều còn chưa chắc**:
  - Với các xe ở khoảng cách trên 25–35 m bị che khuất một phần hoặc chỉ nhận được tia LiDAR ở mặt đuôi/đầu xe (như `s01f07` idx 4 ở 32,59 m và idx 10 ở 38,09 m), dù đã tham chiếu ảnh camera và bảng 4.3 để đặt chiều dài hộp khoảng 3,885–4,233 m nhưng vẫn khó chắc chắn tuyệt đối tâm hộp dọc trục thân xe không bị dịch vài chục cm so với kích thước thật của xe. Ngoài ra với nhóm người đi bộ đi sát nhau ở xa tại `s04f02` (`idx 5` đến `idx 9`), mật độ điểm khá thưa nên góc `yaw` của từng người đi bộ có thể còn sai số nhỏ.
