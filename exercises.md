# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng mẫu trả lời bằng câu trả lời của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Trần Cao Thắng  Mã học viên: 2A202602520

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu tôi deploy lên Railway/Render mà quên set `AGENT_API_KEY`, app sẽ dừng ngay lúc khởi động và log sẽ báo thiếu biến môi trường. Nhờ vậy tôi biết lỗi ở bước deploy và sửa trong dashboard. Nếu để mặc định `"changeme"`, service vẫn chạy công khai, người khác có thể đoán hoặc dùng key mặc định để gọi `/ask`, làm tốn quota mà tôi chỉ phát hiện khi xem log hoặc chi phí.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Ví dụ một dòng log sau khi gọi `/ask`: `{"event":"ask_completed","level":"info","timestamp":"2026-09-28T12:00:00+00:00","user_id":"sv-test","tokens_in":4,"tokens_out":24,"cost_usd":0.000028}`. Với log JSON này tôi có thể lọc theo `event` hoặc `user_id` để biết ai gọi API, và có thể cộng `cost_usd` để theo dõi chi phí. Nếu chỉ `print("đã trả lời xong")` thì không biết request của user nào, lúc nào, tốn bao nhiêu token hay tiền.

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
| 1 stage (bản đầu) | Chưa đo riêng vì file đã được thay bằng bản multi-stage |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Sau khi bật Docker, image multi-stage của tôi đo được khoảng 271 MB. Tôi không còn giữ Dockerfile 1 stage ban đầu để đo lại riêng, nhưng bản đó dùng `python:3.11` đầy đủ nên sẽ lớn hơn vì mang base image nặng và toàn bộ build context. Bản multi-stage dùng `python:3.11-slim`, cài dependency ở stage `builder` rồi chỉ copy kết quả sang stage runtime, nên image cuối chỉ còn Python runtime, thư viện cần thiết, `app/` và `utils/`.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Dockerfile của tôi copy `requirements.txt` và chạy `pip install` trước, sau đó mới copy source code. Khi sửa một ký tự trong `app/main.py`, các layer base image, `WORKDIR`, `COPY requirements.txt`, `pip install` và copy dependency từ builder vẫn dùng lại cache; layer copy source và các layer phía sau nó phải chạy lại. Nếu đặt `COPY . .` trước `RUN pip install`, chỉ cần sửa một file code nhỏ cũng làm mất cache của layer cài thư viện, build sẽ chậm hơn nhiều.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu app Python có lỗi cho phép chạy lệnh hệ thống hoặc ghi file tùy ý, kẻ tấn công có thể chiếm quyền bên trong container. Nếu container chạy bằng root, quyền đó là root trong container và có thể trở nên nguy hiểm hơn khi có mount volume, socket Docker hoặc lỗi escape container. Lệnh `USER appuser` cắt chuỗi này bằng cách làm process chạy với user thường, nên dù app bị khai thác thì quyền ghi/đọc trong container bị giới hạn hơn.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Nếu đếm theo phút đồng hồ, user có thể gửi tối đa 20 request trong khoảng 2 giây: gửi 10 request lúc 10:00:59, sau đó bộ đếm reset ở 10:01:00, rồi gửi thêm 10 request lúc 10:01:01. Sliding window 60 giây tránh lỗ hổng này vì nó luôn nhìn lại 60 giây gần nhất, nên 20 request đó vẫn nằm trong cùng một cửa sổ và bị chặn.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn tốc độ gọi API, còn cost guard giới hạn tổng chi phí theo ngân sách tháng. Rate limit có thể cho qua nhưng cost guard chặn khi user gọi ít request nhưng mỗi request rất dài, làm tổng tiền vượt ngân sách. Ngược lại, cost guard có thể vẫn cho qua vì còn ngân sách, nhưng rate limit chặn khi user spam nhiều request nhỏ trong một phút.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp `/health` và `/ready` rồi endpoint đó kiểm tra Redis, khi Redis mất kết nối 30 giây thì cả 3 container sẽ trả health check lỗi. Orchestrator tưởng process chết, bắt đầu restart các container. Trong lúc restart, traffic mới vẫn bị gián đoạn, request đang xử lý có thể rớt, và sự cố nhỏ ở Redis bị biến thành sự cố lớn ở toàn bộ app. Tách riêng giúp `/health` chỉ báo process còn sống, còn `/ready` mới báo có nên nhận traffic hay không.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi lưu history trong Redis, các instance dùng chung một nguồn state, nên cùng `X-User-Id` sẽ thấy `history_length` tăng ổn định qua các lần gọi: 0, 2, 4... tùy số lượt trước đó. Nếu lưu bằng dict Python trong RAM, mỗi container có dict riêng; request vào container A rồi container B sẽ thấy lịch sử lúc có lúc không, `history_length` có thể nhảy không đều hoặc quay về 0.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi tôi gặp ở bước chuẩn bị deploy/local run là Docker Desktop báo `Virtualization support not detected`, sau đó PowerShell chưa chạy được Docker. Tôi kiểm tra bằng `wsl --status` và `docker info`, rồi bật WSL2/Virtual Machine Platform, đặt WSL default version là 2 và mở lại Docker Desktop. Sau khi sửa, `docker info` chạy được, tôi build image và chạy `docker compose up -d` thành công.
