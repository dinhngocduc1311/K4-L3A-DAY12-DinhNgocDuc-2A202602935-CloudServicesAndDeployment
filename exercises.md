# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng trả lời mẫu bên dưới bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Đinh Ngọc Đức  Mã học viên: 2A202602935

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Ví dụ cụ thể là lúc deploy lên Railway nhưng quên khai báo `AGENT_API_KEY`.
> Cấu hình bắt buộc làm tiến trình dừng ngay và log báo thiếu biến, nên deployment
> không thể nhận traffic. Nếu dùng mặc định `"changeme"`, service vẫn báo khỏe và
> bất kỳ ai đoán được khóa mặc định đều có thể gọi `/ask`, tiêu tốn ngân sách.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T09:15:09.200166+00:00", "user_id": "sv01", "tokens_in": 4, "tokens_out": 42, "cost_usd": 2.58e-05}`
>
> Từ log này tôi có thể (1) lọc và đếm sự kiện theo `event`, `user_id` hoặc khoảng
> thời gian; (2) cộng `cost_usd` và số token để lập dashboard/cảnh báo chi phí.
> Chuỗi `print("đã trả lời xong")` không có trường dữ liệu ổn định để làm hai việc đó.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.73 GB (1,728,524,700 byte) |
| Multi-stage | 271 MB (271,076,437 byte) |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi build trực tiếp Dockerfile gốc trong Git thành `agent:single` và bản hiện tại
> thành `day12-agent:cp2-test`. Chênh lệch khoảng 1.46 GB chủ yếu đến từ image
> `python:3.11` đầy đủ (Debian và bộ công cụ hệ thống lớn hơn) so với
> `python:3.11-slim`; bản một stage còn giữ cache của `pip` và toàn bộ build context.
> Bản multi-stage dùng `--no-cache-dir`, rồi runtime chỉ nhận dependency đã cài cùng
> `app/` và `utils/`, không mang môi trường builder sang.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa `app/main.py`, các layer lấy base image, copy `requirements.txt`, cài
> dependency trong builder, tạo user và copy `/install` đều được dùng lại từ cache.
> Layer `COPY app ./app` và mọi layer đứng sau nó, gồm `COPY utils ./utils` và
> `USER`, phải được tạo lại dù `utils/` không đổi. Nếu đặt
> `COPY . .` trước `RUN pip install`, mọi thay đổi source sẽ làm mất cache của layer
> copy và buộc cài lại toàn bộ dependency dù `requirements.txt` không thay đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Chuỗi rủi ro là: lỗi Python cho phép thực thi lệnh trong container; tiến trình đang
> là root nên kẻ tấn công có toàn quyền với filesystem và capability được cấp cho
> container; nếu runtime/kernel có lỗ hổng escape hoặc container bị gắn socket hay
> thư mục nhạy cảm của host, quyền đó có thể bị dùng để chiếm host. `USER appuser`
> cắt chuỗi ngay sau bước thực thi lệnh: mã độc chỉ có UID 10001 và bị giới hạn trên
> các tài nguyên container. Nó giảm blast radius, dù không thay thế việc cấu hình
> capability, mount và runtime an toàn.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Tối đa là 20 request: gửi 10 request ở giây cuối của phút cũ, rồi ngay sau khi bộ
> đếm reset ở giây 00 gửi thêm 10 request của phút mới. Hai nhóm nằm trong khoảng
> hai giây nhưng thuộc hai bucket khác nhau. Sliding window 60 giây vẫn nhìn thấy
> nhóm cũ nên không cho burst 20 request như vậy.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit bảo vệ tốc độ theo cửa sổ 60 giây; cost guard bảo vệ tổng tiền đã dùng
> trong tháng. Người dùng gửi một request sau thời gian nghỉ sẽ qua rate limit nhưng
> vẫn bị cost guard chặn nếu ngân sách tháng đã hết. Ngược lại, người dùng còn toàn
> bộ ngân sách nhưng gửi request thứ 11 liên tiếp khi hạn mức là 10/phút sẽ bị rate
> limit chặn dù cost guard vẫn cho phép.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Khi Redis mất kết nối, endpoint gộp bắt đầu trả 503 ở cả ba container. Bộ điều phối
> hiểu nhầm process chết, loại các instance khỏi traffic rồi có thể restart chúng.
> Các process mới vẫn không kết nối được Redis nên tiếp tục fail và tạo vòng lặp
> restart, làm mất cả khả năng quan sát liveness trong lúc sự cố dependency chỉ kéo
> dài 30 giây. Tách endpoint thì `/health` vẫn 200 để container không bị restart,
> còn `/ready` trả 503 để tạm ngừng request mới; Redis phục hồi thì readiness tự trở
> lại 200.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với Redis, cả ba replica đọc và ghi cùng một key theo `X-User-Id`, nên
> `history_length` phản ánh một lịch sử chung và tăng nhất quán cho tới giới hạn
> `history_max_messages`. Nếu dùng dict trong từng process, request được cân bằng qua
> replica sẽ thấy ba lịch sử riêng: số lúc tăng, lúc quay về giá trị nhỏ hơn tùy
> replica nhận request; restart một replica còn làm lịch sử của replica đó về 0.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi tôi gặp là Railway API báo `healthcheckPath: null` dù endpoint `/health` và
> Docker `HEALTHCHECK` đều hoạt động. Tôi xác định nguyên nhân bằng truy vấn
> `serviceInstance` và thấy cấu hình trong `railway.toml` chỉ gắn với từng deployment,
> chưa cập nhật Healthcheck Path ở service settings. Tôi dùng mutation
> `serviceInstanceUpdate` đặt `healthcheckPath` thành `/health`, redeploy, rồi kiểm tra
> lại API trả `/health` và deployment mới chuyển sang `SUCCESS`.
