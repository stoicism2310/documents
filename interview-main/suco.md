# Phát hiện sự cố

- Thông qua hệ thống giám sát
- Phản ánh từ người dùng


# Xác nhận và phân loại xự cố

Xác nhận: Kiểm tra logs, health check, monitoring dashboards.

Phân loại:
- Mức độ nghiêm trọng: Critical / High / Medium / Low.
- Loại sự cố: DB, service, network, traffic spike…

# Phân tích nguyên nhân & ưu tiên xử lý 

- Xem logs, trace error, kiểm tra dependency.
- Xử lý những phần ảnh hưởng lớn nhất tới người dùng trước.


# Giải pháp tạm thời / hạn chế thiệt hại (Mitigation / Containment)

- Fallback / degrade service: Ví dụ trả dữ liệu cache, giảm tính năng không quan trọng.
- Restart service / DB / server: Nếu lỗi phần mềm tạm thời.
- Chặn traffic / rate limiting: Nếu do spike quá tải.

# Khắc phục (Resolution / Recovery)

- Fix root cause: Patch bug, tăng resource, config lại DB, deploy lại service…
- Kiểm tra: Đảm bảo hệ thống trở lại trạng thái bình thường.
- Theo dõi sát: Monitoring để chắc chắn lỗi không tái phát.


# Hậu sự cố (Post-Incident / Review)

- Postmortem / RCA (Root Cause Analysis): Ghi lại nguyên nhân, cách xử lý, lesson learned.
- Cải thiện: Update playbook, monitoring, alert, tự động hóa retry, scaling, circuit breaker…
- Báo cáo: Cho team / management để tránh tái diễn.


## Postgres

-> trích xuất slow query
-> check thời gian query, lock time









# Com

1. Nhận diện vấn đề
2. Ưu tiên giảm impact
3. Debug bằng tín hiệu (log/metric/trace)
4. Mitigation
5. Bài học / cải tiến




# Câu hỏi

BỘ CÂU HỎI XỬ LÝ SỰ CỐ (INCIDENT HANDLING)
I. TƯ DUY & QUY TRÌNH (BẮT BUỘC)

1. Khi hệ thống production có lỗi nghiêm trọng, việc đầu tiên bạn làm là gì?

-> xem chức năng nào lỗi, đánh giá xem có ảnh hưởng lớn tới người dùng hay k. Nếu lỗi ảnh hưởng lớn đến hạ tầng thì scale out, tăng ratelimit để giảm tải đến hệ thống tránh lỗi ảnh hưởng đến database. còn nếu lỗi do mới upcode -> có thể rollback luôn sau đó trace log để tìm nguyên nhân cụ thể

2. API chậm nhưng k lỗi.
-> check metric xem nguyên nhân nghẽn cổ chai có thể ở đâu -> xem APM của api xem do service hay database chậm.

3. Metric nào kiểm tra đầu tiên?
Error rate, latency (p95/p99), tài nguyên (CPU, memory), và dependency latency. Vì metric cho em cái nhìn nhanh, log chỉ dùng khi đã biết nghi ngờ chỗ nào.

4. API timeout hàng loạt – làm gì ngay?

Em giảm tải ngay bằng rate limiting hoặc tắt tạm một số chức năng không critical, sau đó scale service nếu cần. Mục tiêu là ổn định hệ thống trước, không để domino fail.

5. 

# Kafka

6. Consumer lag Kafka tăng liên tục?

Em kiểm tra tốc độ consume so với produce. Nếu consume chậm do xử lý nặng, em xem lại concurrency, partition và logic xử lý. Scaling consumer không có tác dụng nếu partition không đủ hoặc processing bị block.

7. Scale consumer mà lag không giảm?

Vì offset được xử lý theo partition, nếu số partition không đủ thì scale consumer sẽ không giúp. Ngoài ra còn có khả năng consumer bị block bởi I/O hoặc retry không kiểm soát.





Bạn phân biệt thế nào giữa Incident – Bug – Performance issue?

Nếu vừa nhận alert nhưng chưa rõ nguyên nhân, bạn làm gì tiếp theo?

Bạn ưu tiên fix nhanh hay tìm root cause? Vì sao?

Khi nào cần rollback, khi nào nên hotfix?

Incident severity (P0/P1/P2) là gì? Tác động đến xử lý ra sao?

Làm sao để đảm bảo không làm sự cố tệ hơn khi đang xử lý?

Bạn giao tiếp với team / business khi hệ thống đang down thế nào?

II. API / BACKEND BỊ CHẬM – TIMEOUT

API bắt đầu response chậm nhưng không error → bạn debug ra sao?

Quy trình chuẩn:

1. Kiểm tra các metric hệ thống (CPU, Memory, GC, Thread pool, Connection pool).

2. Kiểm tra log ứng dụng:

- Slow log (từng request).

- Log thời gian gọi DB, external APIs.

3. Bật trace APM/Profiling (Jaeger, Zipkin, New Relic, Prometheus + Grafana, Spring Boot Actuator):

- Xác định request bị “tắc” ở service nào.

- Xác định phần code có time-consuming bất thường (ví dụ: serialization, regex, Stream API, batch processing).

4. Kiểm tra DB:

- Slow query log.

- Locking.

- Connection pool usage.

5. Kiểm tra external services:

- Latency tăng.

- Rate limit/timeout từ bên thứ ba.

- Tái hiện issue ở môi trường dev/staging nếu có thể để kiểm tra truy vết.


Metric nào bạn kiểm tra đầu tiên? Vì sao là metric đó?

Metric đầu tiên: Latency theo từng layer

- App latency

- DB latency

- External service latency

- Thread pool / connection pool usage

Lý do: latency breakdown cho thấy chính xác “tắc” ở đâu. Bạn sẽ biết ngay request chậm do app, DB, hay external service mà không cần phỏng đoán.

Nếu phải chọn 1 metric duy nhất, tôi chọn Thread pool usage (hoặc request queue length):

- Nếu thread pool bị full → toàn bộ request chậm đồng loạt.

- Đây là nguyên nhân phổ biến nhất khi API “không lỗi nhưng chậm”.

Làm sao phân biệt:

CPU bottleneck

DB bottleneck

### External service bottleneck

- CPU bottleneck

- CPU usage ~ 80–100%.

- GC time tăng cao (GC Overhead > 20%).

- Thread runnable tăng đột biến.

- App latency tăng ở các đoạn xử lý logic, không phải thời gian I/O.

- pprof/YourKit/Java Flight Recorder: thấy CPU hotspot trong code (ví dụ: map/reduce, encryption).

### DB bottleneck

- Slow query log nhiều.

- Latency DB tăng, connection pool chờ lâu.

- IOPS cao, lock, deadlock, index scan thay vì index seek.

- API chậm ở các step DB call (trace sẽ rõ).

- Connection pool gần cạn → request đợi connection.

### External service bottleneck

- Timeout khi gọi API bên ngoài.

- Latency outbound tăng mạnh.

- Circuit breaker mở (nếu có).

- Log ghi lại “ReadTimeout / ConnectTimeout”.


Thread pool full thì chuyện gì xảy ra?

- Tất cả thread worker đều bận → request mới phải xếp hàng trong queue.

- Queue đầy → request bị từ chối hoặc chờ quá lâu dẫn tới timeout hàng loạt.

- CPU thường không cao, nhưng latency tăng mạnh.

### Triệu chứng:

- Request pending lâu.

- Không có stacktrace rõ ràng.

- Chỉ có timeout từ client/log.

- Đây là lý do nhiều hệ thống gặp timeout dây chuyền (cascade failure).

Connection pool cạn thì biểu hiện như thế nào?

- API chậm ở đoạn gọi DB/external service.

Log xuất hiện lỗi:

    - Timeout waiting for connection from pool

    - HikariPool-1 - Connection is not available

- Thread bị block → kéo theo thread pool full → API timeout dây chuyền.

- DB server có thể hoạt động bình thường, nhưng app không lấy được connection.

Khi API timeout hàng loạt, biện pháp giảm impact ngay lập tức là gì?

A. Tác vụ ứng cứu khẩn cấp (Immediate mitigation)

1. Tăng timeout phía client/sidecar để giảm request retry/áp lực dồn lên server.

2. Giảm traffic bằng cách:

- Bật rate limit.

- Cho một số client về fallback route.

- Canary roll: chỉ nhận 20–30% traffic trước.

3. Kill các long-running request (tùy hệ thống):

- Restart instance bị đơ (cực đoan nhưng hiệu quả).

4. Tạm thời scale out:

- Tăng số replica trên Kubernetes/EC2.

- Giảm tải trên mỗi instance.

5. Bật circuit breaker để ngăn gọi vào external service bị chết.

6. Disable background jobs đang chiếm tài nguyên (ETL, batch processing).

B. Sau khi hệ thống ổn, tiến hành root cause analysis



Bạn từng gặp thundering herd problem chưa? Xử lý thế nào?

Thundering herd problem: hàng loạt request cùng truy cập một resource (DB, cache, API) tại cùng thời điểm → gây quá tải.

Triệu chứng

Cache miss → hàng nghìn request cùng query DB → DB quá tải.

External service rate-limit → retry storm → chết cả cụm service.

Cách xử lý chuẩn
1. Cache stampede protection

Mutex/Locking cache: chỉ 1 thread được phép rebuild cache.

- Soft TTL + Hard TTL:

  - Soft TTL hết hạn → refresh async.

  - Hard TTL hết hạn mới xóa.

Randomize TTL jitter để tránh hết hạn cùng lúc.

2. Request collapsing / deduplication

- Nhiều request giống nhau → gộp thành 1 request upstream.

3. Circuit breaker + fallback

- Khi upstream overload, trả về cache cũ hoặc fallback data.

4. Exponential backoff cho retry

- Không retry “dồn dập”, giảm burst traffic.

5. Pre-warm cache định kỳ

III. DATABASE SỰ CỐ

DB CPU 100% → bạn xử lý theo thứ tự nào?

Làm sao phát hiện slow query nhanh nhất?

Nếu không tìm thấy slow query nhưng DB vẫn chậm?

Deadlock là gì? Dấu hiệu nhận biết trong production?

Khi có 1 query nguy hiểm đang chạy, bạn có kill không? Vì sao?

Index giúp gì và khi nào thêm index lại phản tác dụng?

Read replica lag gây ra vấn đề gì cho business?

IV. KAFKA / MESSAGE QUEUE

Consumer lag tăng liên tục → nguyên nhân phổ biến?

Consumer chết nhưng offset vẫn commit → chuyện gì đang xảy ra?

Scale consumer mà lag vẫn không giảm → vì sao?

Retry không kiểm soát gây hậu quả gì?
Retry không kiểm soát = hệ thống tự tấn công chính mình
Retry không kiểm soát gây retry storm, làm khuếch đại lỗi ban đầu, cạn tài nguyên, tạo backlog vô hạn và dẫn đến outage dây chuyền. Retry sai chỗ còn gây duplicate side effects và che giấu lỗi logic nghiêm trọng.

Khi backlog quá lớn, bạn xử lý:

Drain message?

Skip?

Reprocess?

DLQ dùng khi nào? Khi nào KHÔNG nên retry?

DLQ được dùng cho những message không thể xử lý thành công bằng retry hợp lý, thường là lỗi dữ liệu hoặc logic. Retry chỉ nên áp dụng cho lỗi tạm thời như network hoặc downstream failure. Retry với lỗi deterministic sẽ gây overload và poison queue.

Nếu Kafka cluster vẫn sống nhưng throughput tụt mạnh?

Kafka cluster vẫn sống nhưng throughput giảm thường do consumer lag, partition bottleneck, ISR shrink, hoặc disk IO chậm. Kafka sẽ tự giảm tốc để bảo toàn tính nhất quán và durability, nên cần kiểm tra toàn pipeline từ producer đến consumer, đặc biệt là consumer và disk.

partition bottleneck?
Partition bottleneck là khi throughput của Kafka bị giới hạn bởi một hoặc vài partition, do mỗi partition chỉ được xử lý tuần tự bởi một consumer trong group. Nguyên nhân thường do ít partition hoặc phân phối key không đều, dẫn tới lag cao dù cluster vẫn còn tài nguyên.

V. KUBERNETES / SERVICE CRASH

Pod bị CrashLoopBackOff → xử lý thế nào?

Liveness vs Readiness probe khác nhau thế nào trong incident?

Khi pod startup rất chậm, nên set probe ra sao?

Config sai khiến pod không start → có nên scale không?

Khi nào nên rollback deployment thay vì debug tiếp?

Service restart liên tục nhưng không crash → vấn đề thường nằm ở đâu?

VI. MEMORY / CPU / RESOURCE LEAK

Memory tăng dần không giảm → bạn nghi ngờ điều gì?

GC ảnh hưởng latency thế nào?

CPU thấp nhưng hệ thống vẫn chậm → giải thích?

File descriptor leak biểu hiện ra sao?

Khi nào nên restart service, khi nào không?

VII. INCIDENT THỰC TẾ & RA QUYẾT ĐỊNH

Một fix đúng kỹ thuật nhưng downtime 5 phút, bạn có deploy không?

Nếu sếp yêu cầu deploy gấp nhưng bạn thấy rủi ro?

Khi incident do lỗi con người, postmortem nên làm thế nào?

Bạn viết postmortem để làm gì?

Đâu là dấu hiệu một team xử lý sự cố kém?

VIII. THIẾT KẾ ĐỂ TRÁNH LẶP LẠI SỰ CỐ

Rate limiting giúp gì trong incident?

Circuit breaker khác retry thế nào?

Backpressure quan trọng ở đâu?

Monitoring thiếu metric nào là cực kỳ nguy hiểm?

Nếu được quay lại, bạn sẽ thiết kế lại hệ thống để tránh sự cố đó thế nào?