## Relational Databases (SQL)

Tên gọi khác:

- RDBMS (Relational Database Management System)

- SQL Database

Ví dụ phổ biến:

- MySQL
- PostgreSQL
- Oracle Database
- SQL Server


➤ Đặc điểm

- Lưu dữ liệu trong bảng (tables) với hàng (rows) và cột (columns).
- Có cấu trúc rõ ràng (schema cố định).
- Hỗ trợ JOIN mạnh mẽ giữa nhiều bảng.
- Dùng SQL để truy vấn dữ liệu.
- Đảm bảo ACID → rất ổn định và nhất quán.

➤ Khi nào nên dùng SQL?
- Dữ liệu có quan hệ rõ ràng.
- Cần JOIN phức tạp.
- Cần giao dịch (transaction) mạnh mẽ.
- Yêu cầu tính nhất quán cao


## Non-Relational Databases (NoSQL)

Tên gọi khác:

- NoSQL database

Ví dụ phổ biến:

- DynamoDB (key-value + document)
- Cassandra (column store)
- MongoDB (document store)
- CouchDB (document store)
- Neo4j (graph database)
- Redis (key-value)
- HBase

➤ 4 loại chính của NoSQL

- Key-value store
Redis, DynamoDB
→ Truy xuất cực nhanh (low latency)

- Document store
MongoDB, CouchDB
→ Lưu JSON, BSON, XML…

- Column store
Cassandra, HBase
→ Lưu dữ liệu dạng cột, tốt cho phân tán và big data

- Graph store
Neo4j, Amazon Neptune
→ Lưu quan hệ dạng đồ thị (mạng xã hội, recommendation)

➤ Đặc điểm NoSQL

- Không có schema cố định (schema-less).
- Không hỗ trợ JOIN hoặc hỗ trợ rất hạn chế.
- Tốc độ đọc/ghi rất cao.
- Dễ mở rộng ngang (horizontal scaling).
- Lưu trữ linh hoạt, phù hợp dữ liệu phi cấu trúc.


NoSQL phù hợp khi:

✔ Ứng dụng yêu cầu độ trễ cực thấp (super-low latency)

Ví dụ:

- Cache (Redis)

- Session store

- Real-time ranking

✔ Dữ liệu không có cấu trúc hoặc không quan hệ

Ví dụ:

- JSON động

- Log data

- IoT data

- Document linh hoạt

✔ Bạn chỉ cần serialize/deserialize dữ liệu

Ví dụ: dữ liệu dạng:

- JSON

- XML

- YAML

→ Dùng document DB như MongoDB rất phù hợp.

✔ Cần lưu lượng dữ liệu cực lớn (massive scale)

Ví dụ:

- Social network posts

- Logs

- Sensor streaming

- Big data analytics

- SQL khó mở rộng theo kiểu này → NoSQL tốt hơn.


SQL là lựa chọn mặc định vì:

- ổn định 40+ năm
- mạnh về ACID và JOIN
- quen thuộc với hầu hết developer

NoSQL phù hợp khi:

- cần hiệu năng cực nhanh
- cần lưu khối lượng dữ liệu rất lớn
- dữ liệu phi cấu trúc hoặc linh hoạt
- cần scale ngang




# Scaling

## Vertical Scaling (Scale Up)
✔ Ưu điểm

- Đơn giản → không cần thay đổi nhiều về kiến trúc.
- Phù hợp khi traffic còn nhỏ.
- Dễ cài đặt, dễ quản lý.

✖ Nhược điểm nghiêm trọng

- Có giới hạn vật lý
Không thể nâng cấp vô hạn.
→ Một server chỉ nâng cấp đến mức nào đó rồi hết chỗ.

- Không có redundancy (dự phòng)
Nếu server duy nhất chết, toàn bộ website/app down.

- Không chịu tải lớn
- Khi traffic tăng đột biến → CPU/RAM đạt ngưỡng → website chậm hoặc sập.

➡️ Vì vậy vertical scaling chỉ phù hợp cho hệ thống nhỏ hoặc đơn giản.


## Horizontal Scaling (Scale Out)

Horizontal scaling = thêm nhiều server vào hệ thống.

Và chia tải giữa chúng.

✔ Lợi ích lớn

- Không giới hạn mở rộng
Có thể thêm bao nhiêu server tùy thích → phù hợp traffic lớn.

- Redundancy (tính dự phòng)
Nếu 1 server chết → các server còn lại tiếp tục phục vụ.

- High availability
Hệ thống hoạt động liên tục, không bị down khi 1 node lỗi.

- Tối ưu chi phí
Có thể dùng nhiều server nhỏ giá rẻ thay vì 1 máy cực mạnh và đắt.

➡️ Horizontal scaling = tiêu chuẩn trong các hệ thống lớn (Google, Facebook, Amazon).   

## Cân bằng tải load balancer


Nếu server down → người dùng không truy cập được.

Nếu quá nhiều người gửi request → server bị quá tải → chậm, lỗi.

✔ Load balancer giải quyết cả 2 vấn đề

Load balancer:

- Ngồi trước các backend server
- Chuyển request tới nhiều server phía sau
- Giúp phân tải (load balancing)
- Giúp failover (nếu 1 server hỏng, vẫn hoạt động)

🔧 Load balancer giúp:

- Tăng throughput (tổng số request/s)
- Cải thiện uptime
- Giảm độ trễ
- Cho phép scale-out dễ dàng

Note: 
- Vertical vs horizontal scaling

- Load balancer + nhiều web server

- Kiến trúc 3-tier hoàn chỉnh (LB → web → DB) 


## load balancer + multiple web servers (load balancer ở web tier)

Khi người dùng vào https://mysite.com, DNS trỏ domain đến public IP của Load Balancer.
Để tăng bảo mật, các web server (server1, server2, server3…) không có public IP.

Failover
- Server1 offline (down)
- Health check của LB báo “unhealthy”
- Load balancer dừng gửi request đến server đó

➡️ Tất cả traffic được chuyển sang server2 (hay server3).

-> Website vẫn hoạt động bình thường — không downtime.

Scalability

Kiến trúc này tăng khả năng mở rộng theo chiều ngang -> tăng thêm server và load balancer sẽ tự động detact và điều hướng request đến server mới



## Database Replication

“Database replication là việc sao chép dữ liệu từ 1 database gốc (master) sang các bản sao (slaves).”

Mục tiêu:

- tăng khả năng đọc (read performance)

- tăng khả năng chịu tải

- có dữ liệu dự phòng (redundancy)

- giảm single-point-of-failure

Master -> WRITE và sync sang slaves
Slave -> READ

tăng lượng slave theo read traffic 

Ví dụ:

- Chỉ ~5–10% request là write

- 90–95% là read (lấy danh sách, chi tiết, search…)

➡️ Do đó:

- Master ít khi bị quá tải

- Slave là nơi chịu đọc liên tục, nên thêm nhiều slave để chia tải


✔ Boost hiệu năng đọc cực mạnh

Thêm bao nhiêu slave tùy ý → mở rộng ngang (scale-out).

✔ Tăng redundancy

Master chết → slave vẫn còn dữ liệu.

✔ Tối ưu cho workload có nhiều đọc

Hầu hết web/app đều như vậy.

✔ Tăng khả năng chịu tải (high availability)


✖ 1. Không giải quyết được “single point of failure” của master

Master chết → không ghi được → hệ thống write bị tê liệt, dù slave vẫn hoạt động.

→ Cần failover hoặc multi-master để nâng cấp tính sẵn sàng.

✖ 2. Có độ trễ replicate (replication lag)

Slave có thể chậm hơn master vài mili-giây → vài giây.

→ Nếu vừa write xong mà đi đọc slave → có thể chưa thấy dữ liệu (read-after-write inconsistency).

✖ 3. Kiến trúc phức tạp hơn

Cần routing read/write

Cần monitor delay replication

Cần backup/restore cả hệ thống