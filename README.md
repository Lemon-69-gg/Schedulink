# Lịch rảnh của team — bản tự host, đồng bộ thời gian thực

Một file HTML duy nhất. Chạy được ngay khi mở bằng trình duyệt (chế độ lưu trên máy),
và tự bật đồng bộ thời gian thực khi bạn dán config Firebase vào.

## Có gì bên trong

| File | Vai trò |
|---|---|
| `tkb-team.html` | Toàn bộ app. Không cần build, không cần npm. |
| `database.rules.json` | Quy tắc bảo mật cho Realtime Database. Bắt buộc dán lên trước khi dùng thật. |
| `README.md` | File này. |

## Hai chế độ chạy

**Lưu trên máy** (mặc định, khi `FIREBASE_CONFIG = null`)
Dữ liệu nằm trong `localStorage` và trong URL. Chia sẻ bằng nút *Chia sẻ* → sao chép mã →
người khác bấm *Nhập mã* → dán. Không có máy chủ nào giữ dữ liệu.

**Đồng bộ trực tiếp** (khi đã dán config)
Mỗi team có một "phòng", nhận diện bằng `?room=` trên URL. Ai mở link phòng cũng thấy và
sửa được cùng một lịch; thay đổi hiện ra ở máy người khác trong khoảng một giây.
Không cần đăng nhập.

Góc phải trên hiện chồng avatar của những người đang mở phòng, kiểu Google Docs:
icon rõ nét là người đang xem ngay lúc này, icon mờ là người vừa rời đi trong 15 phút
gần đây hoặc đang mở tab khác. Ai chưa bấm vào chồng avatar để nhận tên mình thì hiện
ra bằng hình bóng xám như khách ẩn danh.

---

## Thiết lập Firebase

### Bước 1 — Tạo project

1. Vào https://console.firebase.google.com → **Add project**.
2. Đặt tên (ví dụ `tkb-team`). Tắt Google Analytics cho gọn.
3. Chờ tạo xong → **Continue**.

### Bước 2 — Bật Realtime Database

1. Menu trái → **Build → Realtime Database** → **Create Database**.
2. Chọn vùng **Singapore (asia-southeast1)** cho độ trễ thấp nhất từ Việt Nam.
3. Chọn **Start in locked mode**. Đừng chọn test mode — nó mở toang database cho cả
   internet trong 30 ngày.
4. Ghi lại `databaseURL` hiện ở đầu trang, dạng
   `https://tkb-team-default-rtdb.asia-southeast1.firebasedatabase.app`

### Bước 3 — Dán quy tắc bảo mật

1. Trong Realtime Database → tab **Rules**.
2. Xoá hết, dán toàn bộ nội dung `database.rules.json` vào.
3. **Publish**.

Đây là bước quan trọng nhất. Quy tắc này:

- chỉ cho đọc/ghi dưới `/rooms/{roomId}`, và chỉ khi mã phòng dài từ 10 ký tự trở lên
  (mã 12 ký tự app sinh ra có khoảng 32¹² tổ hợp — không đoán mò được);
- dùng **danh sách trắng**: bất kỳ khoá nào không nằm trong sơ đồ đều bị từ chối,
  kèm chặn thẳng tên `email`, `phone`, `studentId`, `mssv`;
- giới hạn độ dài từng trường (tên ≤ 28 ký tự, tên môn ≤ 60, phòng ≤ 24…) nên không ai
  nhồi được dữ liệu rác vào quota của bạn.

> **Đã dán quy tắc từ trước?** Dán lại. Bản mới thêm nhánh `presence` cho phần
> "ai đang mở trang". Vì quy tắc dùng danh sách trắng, quy tắc cũ sẽ lặng lẽ từ chối
> mọi lượt ghi presence và chồng avatar ở góc phải sẽ không bao giờ hiện ai.

Realtime Database không có luật giới hạn tổng dung lượng một nhánh, nên phần chặn
"phình to" nằm ở giới hạn độ dài từng trường ở trên. Nếu muốn chắc hơn nữa, vào
**Usage → Set budget alert** để Firebase báo khi vượt ngưỡng.

### Bước 4 — Lấy config

1. ⚙️ **Project settings** → kéo xuống **Your apps** → biểu tượng `</>` (Web).
2. Đặt nickname bất kỳ, **không** tick Firebase Hosting → **Register app**.
3. Copy object `firebaseConfig`.

### Bước 5 — Dán config vào app

Mở `tkb-team.html`, tìm dòng gần đầu file:

```js
const FIREBASE_CONFIG = null;
```

Thay bằng:

```js
const FIREBASE_CONFIG = {
  apiKey: "AIza…",
  authDomain: "tkb-team.firebaseapp.com",
  databaseURL: "https://tkb-team-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId: "tkb-team"
};
```

`databaseURL` là trường bắt buộc — thiếu nó Realtime Database không kết nối được.
Ba trường còn lại (`storageBucket`, `messagingSenderId`, `appId`) không cần cho app này,
có để lại cũng không sao.

> **Về chuyện "lộ API key".** Mấy giá trị này công khai theo thiết kế — bất kỳ web app
> Firebase nào cũng gửi chúng xuống trình duyệt. Chúng chỉ định *database nào*, không
> cấp quyền gì. Thứ quyết định ai làm được gì là `database.rules.json` ở Bước 3.
> Đừng bao giờ tự trấn an rằng "giấu key là đủ".

---

## Đưa lên GitHub Pages

```bash
# trong thư mục chứa tkb-team.html
git init
git add tkb-team.html README.md database.rules.json
git commit -m "Lich ranh cua team"
git branch -M main
git remote add origin https://github.com/<tên-của-bạn>/tkb-team.git
git push -u origin main
```

Rồi trên GitHub:

1. Repo → **Settings → Pages**.
2. **Source**: `Deploy from a branch`. **Branch**: `main`, thư mục `/ (root)` → **Save**.
3. Chờ 1–2 phút. Link sẽ là `https://<tên-của-bạn>.github.io/tkb-team/tkb-team.html`

Muốn link gọn hơn thì đổi tên file thành `index.html`, khi đó link chỉ còn
`https://<tên-của-bạn>.github.io/tkb-team/`.

### Chặn tên miền lạ

Sau khi có link Pages, quay lại Firebase Console → **Authentication → Settings →
Authorized domains** và thêm `<tên-của-bạn>.github.io`. Việc này chặn người khác
copy file HTML của bạn lên site của họ mà vẫn ghi vào database của bạn.

---

## Dùng hằng ngày

1. Mở link Pages. App tự sinh mã phòng và gắn vào URL: `…/tkb-team.html?room=k7mq2xb4tz9a`
2. Bấm **Chia sẻ** → **Sao chép link phòng** → gửi vào nhóm chat của team.
3. Mỗi người mở link, vào tab tên mình, sửa lịch. Mọi người thấy thay đổi gần như ngay lập tức.

Chấm tròn ở góc trên phải cho biết tình trạng: xanh là đang đồng bộ, vàng là đang gửi,
đỏ là mất kết nối. Mất mạng vẫn sửa được bình thường, thay đổi tự gửi lên khi mạng quay lại.

**Đổi sang phòng khác:** sửa `?room=` trên URL thành mã khác (từ 10 ký tự trở lên).
**Sao lưu:** bấm *Chia sẻ* → *Sao chép mã*, cất vào đâu đó. Dán lại qua *Nhập mã* khi cần.

---

## Ghi chú kỹ thuật

- **Ghi theo nhánh nhỏ.** Sửa lịch một người chỉ ghi vào `rooms/{room}/members/{id}`,
  không đụng phần còn lại. Có debounce 400 ms nên gõ phím không spam database.
- **Xung đột.** Last-write-wins theo từng thành viên. Hai người sửa *cùng một* thành viên
  cùng lúc thì người bấm sau thắng. Hai người sửa hai thành viên khác nhau thì không đụng nhau.
- **Mảng lưu thành object.** Realtime Database xử lý mảng kém, nên `members` và `events`
  lưu dạng object theo id, còn danh sách tuần lưu dạng chuỗi `"38,40,44"`. App tự chuyển đổi.
- **Không có Firebase cũng chạy.** Nếu CDN bị chặn hoặc config sai, app rơi về chế độ lưu
  trên máy và hiện banner báo — không trắng trang.
- **Quota.** Gói Spark miễn phí cho 1 GB lưu trữ và 10 GB tải xuống mỗi tháng. Một phòng
  6 người nặng khoảng 10 KB, nên team bạn dùng gần như không đáng kể.

## Đừng nhập vào đây

Họ tên đầy đủ, MSSV, email, số điện thoại, địa chỉ, ảnh thẻ. Chỉ biệt danh và lịch học.
Bất kỳ ai có link phòng đều đọc được toàn bộ nội dung — đó là cái giá của việc không bắt
đăng nhập. Quy tắc bảo mật đã chặn sẵn những trường đó, nhưng đừng nhét chúng vào ô
"tên môn" để lách.
