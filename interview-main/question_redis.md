# Redis

## Q1. Redis tiếp nhận vài chục nghìn request/s → bạn scale thế nào?

* Redis Cluster
* Master–Replica + Read Replica
* Pipeline + batching
* Short TTL
* Sharding key
* Connection pooling

## Q2. Khi TTL không hoạt động (key không tự xóa), nguyên nhân?

* Key được rewrite (overwrite) mà không set lại TTL
* Sử dụng PERSIST
* Memory-policy noeviction
* RDB/AOF restore mất TTL.

## Q3. Làm sao tránh hàng triệu key expire cùng lúc → gây spike CPU?

* Random TTL
* Use EXPIRE key ttl khác nhau
* Spread TTL.

## Q4. Khi DB update nhưng Redis cache cũ chưa được xóa → inconsistency, xử lý như nào?

* Cache Aside: Write DB → Delete cache (Use DB transaction + cache delete after commit)
* Write-through: Update Redis trước → DB sau
    Ưu: strong consistency; reads luôn fresh.
    Nhược: write latency tăng (because cache -> DB)
            phức tạp nếu DB update fail (cần rollback cache)
            khó scale writes (cache becomes single point).
* Write-behind (asynchronous): Update cache, queue the DB write to background worker (batched).
    Ưu: thấp latency for writer, batch DB writes.
    Nhược: nếu worker fail → dữ liệu trong cache nhưng chưa vào DB
* Event-driven: Dùng Kafka để đồng bộ cache invalidation.

Quy trình:
Service thay đổi DB → emit event
Consumer(s) subscribe event → nhận và invalidate cache (DEL) hoặc publish updated value to cache.
Optionally consumers can also push updated entity into cache (cache-warming).

Ưu:
Giải pháp phân tán, bền bỉ; dễ scale; tách concerns: DB write không phụ thuộc vào cache.
Khi dùng Kafka + CDC (Debezium), bạn có toàn bộ stream of changes (useful cho multiple consumers).

Nhược:
Thêm infra (Kafka), thêm latency (eventual consistency).
Duplicate events

Implementation notes
Emit after DB commit — dùng transactional outbox pattern để đảm bảo event không bị mất khi transaction rollback.
consumer phải xử lý event có thể lặp
Ordering: nếu có nhiều updates cùng entity, consumer xử lý theo order (partition key = entity id trên Kafka).

