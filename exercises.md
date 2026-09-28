# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay từng dòng trả lời mẫu bằng câu trả lời của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Trịnh Hoàng Tùng  Mã học viên: L3A202602937

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Một tình huống cụ thể là khi tôi deploy service lên Railway nhưng quên khai
> báo `AGENT_API_KEY`. Nếu khóa có mặc định là `"changeme"`, container vẫn báo
> healthy và API công khai có thể bị gọi bằng khóa rất dễ đoán, làm phát sinh
> chi phí mà tôi không phát hiện ngay. Khi trường này không có mặc định,
> Pydantic báo lỗi ngay lúc khởi động, deployment không qua health check và tôi
> biết phải bổ sung secret trước khi service nhận traffic.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log tôi thu được khi gọi `/ask` là:
>
> ```json
> {"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T09:27:57.129512+00:00", "user_id": "exercise-user", "tokens_in": 6, "tokens_out": 40, "cost_usd": 2.49e-05}
> ```
>
> Thứ nhất, tôi có thể lọc và đếm các sự kiện `ask_completed` theo `user_id`
> hoặc theo khoảng thời gian mà không phải tách chuỗi thủ công. Thứ hai, tôi có
> thể cộng các trường số như `cost_usd`, `tokens_in`, `tokens_out` để làm
> dashboard hoặc cảnh báo khi chi phí tăng bất thường. Câu `print("đã trả lời
> xong")` không có trường dữ liệu ổn định để thực hiện hai việc này.

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
| 1 stage (bản đầu) | 1.70 GB |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi đo bằng `docker images` trên Docker Desktop. Phần chênh lệch khoảng 1,43
> GB chủ yếu đến từ image `python:3.11` đầy đủ: Debian packages, compiler và
> công cụ build như GCC/G++, Git cùng các header/thư viện phát triển. Bản
> production dùng `python:3.11-slim`; stage runtime chỉ nhận dependency đã cài
> từ builder và mã nguồn cần chạy, nên không mang toàn bộ công cụ build sang
> image cuối. Mã nguồn Python của ứng dụng rất nhỏ và không phải nguyên nhân
> chính của phần chênh lệch.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi tôi chỉ sửa `app/main.py`, các layer ở builder gồm base image, `WORKDIR`,
> `COPY requirements.txt` và `RUN pip install` vẫn được lấy từ cache vì
> `requirements.txt` không đổi. Ở runtime, base image và layer copy dependency
> từ builder cũng được dùng lại; `COPY app ./app` bị thay đổi nên layer này và
> các layer nằm sau nó phải được tạo lại. Nếu đặt `COPY . .` trước `RUN pip
> install`, chỉ một thay đổi trong mã nguồn cũng làm layer `COPY` đổi và buộc
> Docker cài lại toàn bộ dependency, khiến mỗi lần build chậm hơn nhiều.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu code Python có lỗ hổng thực thi lệnh từ xa, kẻ tấn công trước hết chạy
> được lệnh bên trong container. Khi process là root, các lệnh đó cũng có UID 0
> trong container; nếu container còn có capability nguy hiểm, mount Docker
> socket/volume nhạy cảm, hoặc kernel có lỗ hổng container escape, kẻ tấn công
> có thể sửa tài nguyên hệ thống rồi leo sang quyền cao trên host. Lệnh `USER
> appuser` cắt chuỗi tại bước đầu: code bị chiếm chỉ chạy với UID 10001, không
> thể tùy ý sửa thư mục hệ thống hay dùng tài nguyên đặc quyền. Cách này không
> thay thế việc vá lỗ hổng kernel nhưng làm giảm đáng kể phạm vi thiệt hại.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng có thể gửi tối đa **20 request trong 2 giây**. Họ gửi 10 request
> ở cuối phút, ví dụ từ `10:00:59`, rồi ngay sau khi bộ đếm reset ở `10:01:00`
> gửi thêm 10 request. Mỗi phút đồng hồ riêng vẫn chỉ ghi nhận 10 request nhưng
> thực tế 20 request đã dồn vào khoảng hai giây. Sliding window 60 giây nhìn
> lại đúng 60 giây gần nhất nên loạt thứ hai sẽ bị chặn.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn **tần suất/số request** trong 60 giây, còn cost guard giới
> hạn **tổng tiền** mỗi người dùng đã tiêu trong tháng. Rate limit có thể cho
> qua một request gửi chậm nhưng cost guard phải chặn nếu người dùng đã dùng hết
> ngân sách tháng, hoặc request dự kiến làm vượt ngân sách. Ngược lại, khi ngân
> sách vẫn còn nhiều nhưng người dùng gửi 11 request rất rẻ liên tiếp, cost
> guard vẫn cho phép về mặt chi phí còn rate limiter chặn request thứ 11 bằng
> HTTP 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu endpoint dùng cho liveness cũng kiểm tra Redis, khi Redis mất kết nối thì
> cả ba container đồng thời trả lỗi health check. Sau số lần probe thất bại,
> orchestrator coi cả ba process đã chết và restart chúng. Redis vẫn chưa phục
> hồi nên các container vừa khởi động lại tiếp tục fail, tạo thành vòng lặp
> restart và làm mất toàn bộ capacity của service; khi Redis trở lại, cả ba còn
> có thể cùng reconnect một lúc. Với hai endpoint tách biệt, `/health` vẫn trả
> 200 nên process không bị restart, còn `/ready` trả 503 để load balancer tạm
> ngừng chuyển request vào chúng. Khi Redis hoạt động lại, `/ready` tự trở về
> 200 và traffic được nhận lại mà không cần restart container.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với Redis dùng chung, mỗi request đọc được lịch sử do bất kỳ instance nào đã
> ghi. Vì một lần `/ask` thêm hai message (`user` và `assistant`), tôi kỳ vọng
> `history_length` tăng theo 0, 2, 4, 6... cho dù request được chuyển tới
> container nào (tối đa theo giới hạn lưu lịch sử). Nếu dùng một dict Python
> riêng trong ba process và load balancer phân phối gần round-robin, tôi sẽ thấy
> dạng 0, 0, 0, 2, 2, 2, 4, 4, 4... hoặc nhảy không đều. Mỗi container chỉ nhớ
> những request nó từng nhận và lịch sử của container sẽ về 0 sau khi restart.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi dùng Railway CLI, tôi gặp lỗi `UNABLE_TO_VERIFY_LEAF_SIGNATURE` lúc `npx`
> tải/chạy CLI. Tôi kiểm tra thấy lệnh HTTPS của Windows vẫn gọi được Railway,
> nên service và URL không hỏng; vấn đề là Node.js không sử dụng đúng CA trong
> kho chứng thư hệ thống của máy. Tôi đặt
> `$env:NODE_OPTIONS='--use-system-ca'` rồi chạy lại `npx -y @railway/cli`.
> Sau đó CLI đăng nhập, `railway up` hoàn tất với trạng thái `SUCCESS`, và cả
> `/health` lẫn `/ready` đều trả HTTP 200.
