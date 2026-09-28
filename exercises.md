# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng placeholder trong từng câu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Võ Phú Hãn
> Mã học viên: 2A202602628

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Nếu thiếu `AGENT_API_KEY` khi deploy, app dừng ngay lúc khởi động nên tôi phát hiện cấu hình thiếu trước khi nhận request. Nếu dùng mặc định `changeme`, service vẫn chạy với thông tin xác thực dễ đoán.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

Log thu được từ container local:

```json
{"event":"ask_completed","level":"info","timestamp":"2026-09-28T07:53:28.941376+00:00","user_id":"local-proof","tokens_in":2,"tokens_out":36,"cost_usd":2.19e-05}
```

Từ các trường có cấu trúc, tôi có thể lọc request theo `user_id` và cộng `cost_usd` để theo dõi chi phí. Một dòng `print()` không có dữ liệu riêng để truy vấn và tổng hợp như vậy.

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
| 1 stage (bản đầu) | 1.73 GB (`agent:single`, Docker build thật) |
| Multi-stage | 297 MB (`day12-agent:cp2-test`, Docker build thật) |

Giải thích: phần dung lượng chênh lệch đó là những gì?

Image multi-stage nhỏ hơn vì image cuối chỉ chứa runtime và dependency cần thiết; công cụ build và thành phần không cần khi chạy được giữ ở build stage. Số đo của tôi giảm từ 1.73 GB xuống 297 MB.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Khi sửa `app/main.py`, Docker dùng lại các layer trước bước `COPY` source; các layer từ bước đó trở đi phải chạy lại. Nếu `COPY . .` đặt trước `RUN pip install`, thay đổi source cũng làm layer cài dependency mất cache.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

Lỗ hổng có thể cho phép kẻ tấn công chạy lệnh trong container. Nếu tiến trình là root, lệnh đó có quyền cao trong container; khi kết hợp với lỗ hổng cô lập hoặc quyền truy cập host được cấp, thiệt hại có thể lan sang host. `USER` chạy app bằng tài khoản ít quyền hơn, giới hạn quyền sau khi bị khai thác.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

Có thể gửi tối đa 20 request: gửi 10 request ngay trước thời điểm sang phút mới, rồi gửi thêm 10 ngay sau khi bộ đếm reset. Bộ đếm theo phút lịch chỉ nhìn thấy 10 request ở mỗi phút, dù chúng xảy ra trong khoảng 2 giây.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

Rate limit giới hạn số request trong một khoảng thời gian; cost guard giới hạn chi phí tích lũy của user. User còn ngân sách tháng nhưng gửi quá nhiều request trong một phút sẽ bị rate limit chặn. Ngược lại, user gửi ít request nhưng request tiếp theo làm vượt ngân sách tháng sẽ bị cost guard chặn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

Khi Redis mất kết nối, nếu `/health` cũng kiểm tra Redis thì cả ba container có thể bị đánh dấu unhealthy và restart. Redis vẫn đang lỗi nên các container mới cũng không sẵn sàng, khiến gián đoạn kéo dài. `/health` chỉ nên kiểm tra tiến trình; `/ready` kiểm tra Redis để ngừng nhận traffic khi dependency chưa sẵn sàng.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Quan sát với 3 replica cùng dùng Redis: `history_length` lần lượt là `0`, `2`, `4`.

Nếu lịch sử nằm trong dict riêng của từng process, mỗi replica chỉ thấy request đã xử lý trên chính nó. Request chuyển sang replica khác có thể trả `history_length` thấp hơn hoặc bằng 0, thay vì cùng tăng từ 0 lên 2 rồi 4 như khi dùng Redis chung.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

Lần đồng bộ Blueprint đầu tiên dùng nhầm nhánh `main`, nên cấu hình deploy chưa lấy commit của bài lab. Tôi kiểm tra lại branch trong cài đặt Blueprint, đổi sang `day12/cloud-deployment` rồi xác nhận Render build commit `c9db5e9` và service chuyển sang Live.
