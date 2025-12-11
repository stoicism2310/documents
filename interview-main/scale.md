#### I. Query & Index Optimization (Scale bằng phần mềm)

* Tối ưu SQL, sử dụng đúng index (B-tree, GiST, GIN), tránh full-table-scan, use EXPLAIN to analyze plans.
* Reindex, VACUUM/ANALYZE định kỳ (Postgres) để giữ planner accurate.

#### II. Caching (Layer ngoài DB)

* In-memory cache: Redis, Memcached. Cache kết quả truy vấn, session, config.
* Materialized views / result caches: DB-level cached views (periodic refresh).
* Ưu: giảm load đọc, cải thiện latency.
* Nhược: cache invalidation complexity.

#### III. Proxy / Middleware & Load balancing

* Connection poolers: pgbouncer (Postgres), ProxySQL (MySQL) — giảm overhead connect/close, quản lý client connections.
* Query routing/load balancing: HAProxy/nginx/pgpool/nginx-proxysql phân luồng read đến replicas, write đến primary.
* Benefits: giảm connection churn, cân bằng read load.

#### IV. Partitioning (Table partitioning) — trong một node/Cluster SQL

Ý tưởng: chia table lớn thành nhiều partition (vật lý) theo cột (date, range, list, hash). Partition nằm trên cùng DB server (hoặc có thể map tới tablespaces).

* Ưu: giảm lượng dữ liệu quét, cải thiện vacuum/maintenance, dễ xóa/archiving partition cũ (fast DROP).
* Nhược: không giải quyết write scale sang nhiều máy; vẫn phụ thuộc I/O server.
* Khi dùng: very large tables (time-series, logs, orders), để archive/ttl, tăng tốc queries theo partition key.

#### V. Sharding (Horizontal partitioning) — Scale ghi & đọc

Ý tưởng: phân chia dữ liệu theo key (ví dụ user_id) sang nhiều node độc lập (shards). Mỗi shard là một DB độc lập.

Kiểu shard phổ biến:

* Range sharding: theo khoảng (e.g. user_id 1-1M trên shard A).
* Hash sharding: hash(key) mod N → phân rải đều.
* Directory sharding: service giữ bảng map key → shard.
* Ưu: mở rộng write throughput (mỗi shard xử lý writes riêng).
* Nhược: phức tạp: cross-shard JOIN/transaction khó khăn; re-sharding (thêm/bớt shard) phức tạp; routing layer cần chính xác.
* Thiết kế quan trọng: chọn shard key phù hợp (đảm bảo phân phối đều, tránh hotspot).

#### VI. Replication — Scale đọc (Read scaling) & High availability

Ý tưởng: giữ nhiều bản sao dữ liệu để phục vụ đọc và dự phòng.

Primary → Replica (Master–Slave / Primary–Replica):

* Viết vào primary; replica nhận bản ghi theo WAL/binlog.
* Replica phục vụ read queries, báo cáo, backup.
* Ưu: dễ triển khai, tăng throughput đọc, nâng cao HA.
* Nhược: ghi vẫn bó vào primary (write bottleneck); replication delay (eventual consistency).

Synchronous vs Asynchronous replication:

* Synchronous: trasaction chờ replica xác nhận → consistency tốt, nhưng latency tăng.
* Asynchronous: nhanh hơn nhưng replica có thể lag (mất cập nhật khi failover).

Ví dụ Postgres (khái niệm):

* Physical streaming replication (wal shipping) hoặc logical replication (CREATE PUBLICATION, CREATE SUBSCRIPTION).
* Khi dùng: read-heavy workloads, reports, analytics, HA.

#### Redis

Tại sao sử dụng Redis: Hiệu suất cao và tính đồng thời cao.

redis lưu trữ dữ liệu trong Ram thay vì ổ dứng -> tốc độ truy vấn cao hơn so với DB ( lưu trữ trên ổ cứng)
redis có nhiều cấu trúc dữ liệu tối ưu cho hiệu suất: keyvalue, hash, stream ...

* Hiêu suất cao: Khi client truy cập dữ liệu DB lần đầu, nó thực hiện đọc từ đĩa cứng -> tốc độ chậm.
Dữ liệu đc người dùng truy cập này sẽ được lưu vào bộ đêm -> từ lần sau dữ liệu được truy vấn từ cache -> tốc độ nhanh hơn.
* Tính đồng thời cao:
các yêu cầu mà redis có thể thỏa mãn trực tiếp cao hơn nhiều so với truy cập trực tiếp DB
-> chuyển 1 phần data từ db lên redis.
* Non-blocking I/O: Xử lý được nhiều kết nối cùng lúc
* Redis chỉ dùng 1 thread chính để xử lý mọi request theo thứ tự.

➡️ Không có 2 thread cùng truy cập 1 dữ liệu
➡️ Không có lock
➡️ Không có tranh chấp tài nguyên
➡️ Không có switching qua lại giữa các thread

👉 Kết quả: tốc độ cực nhanh và ổn định.

* Redis vs Memcached

| Công nghệ                    | Redis                                                       | Memcached                          |
| ---------------------------- | ----------------------------------------------------------- | ---------------------------------- |
| Loại DB                      | In-memory Data Structure Store                              | In-memory Key-Value Cache          |
| Kiểu dữ liệu                 | Nhiều loại (string, hash, list, set, zset, bitmap, stream…) | Chỉ string                         |
| Persistence (lưu xuống disk) | Có (RDB, AOF)                                               | Không                              |
| Cluster                      | Có                                                          | Có (nhưng kém mature hơn)          |

| Tính năng             | Redis | Memcached |
| --------------------- | ----- | --------- |
| String                | ✔     | ✔         |
| Hash (object)         | ✔     | ✘         |
| List (queue)          | ✔     | ✘         |
| Set                   | ✔     | ✘         |
| Sorted Set            | ✔     | ✘         |
| Bitmap                | ✔     | ✘         |
| HyperLogLog           | ✔     | ✘         |
| Streams (event queue) | ✔     | ✘         |

Redis giải quyết cơ chế hết hạn dữ liệu như nào

* xóa khi truy cập: khi client thao tác với 1 key, nếu redis kiểm tra thấy ttl nó đã hết -> xóa
* xóa định kỳ: chạy lệnh xóa các key hết TTL (10 lần /s)
* hết memory: xóa theo chính sách (ưu tiên TTL, hết hạn)