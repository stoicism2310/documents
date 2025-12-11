# Database

## Q1. Một giao dịch đang UPDATE nhiều rows, nhưng bị treo do deadlock. Làm sao để xử lý?

Topic: lock, deadlock

Deadlock xảy ra khi 2 transaction giữ lock chéo nhau → PostgreSQL sẽ tự chọn 1 transaction để kill.

Cách xử lý:

* Giữ thứ tự thao tác nhất quán giữa các transaction.
* Hạn chế transaction dài (giảm idle in transaction)
    Tránh việc mở transaction nhưng ko commit or giữ connection quá lâu mà ko thao tác bất cứ gì
* Giảm phạm vi lock (UPDATE theo primary key):  
    Hệ thống sẽ xác định ngay được bản ghi cần thay đổi và thực hiện lock chỉ bản ghi đó.
    Nếu update ko dùng pk -> số bản ghi lock quá nhiều có thể dẫn tới leo thang lock, lock cả table.
* Dùng SKIP LOCKED hoặc NOWAIT.
* Retry khi bị deadlock.

## Q2. Khi UPDATE/INSERT quá nhiều, table bị lock và request user bị đơ. Làm sao xử lý?

* Tách batch: UPDATE theo từng batch rows.
* Dùng partition để giảm lock scope.
* Giảm ngoài giờ: background job.
* Index đúng để tránh sequential scan (quét full bảng) gây SHARE LOCK lớn.
    Khóa Chia sẻ (S-Lock): Nhiều giao dịch có thể giữ một khóa chia sẻ trên cùng một dữ liệu, cho phép tất cả cùng đọc dữ liệu đó. Thao tác ghi (cập nhật) sẽ bị chặn cho đến khi tất cả khóa chia sẻ được giải phóng.
    Khóa Độc quyền (X-Lock): Chỉ một giao dịch có thể giữ khóa độc quyền.

## Q3. Một bảng lớn bị VACUUM lâu và khiến query chậm, bạn làm gì?

* Kiểm tra autovacuum có chạy không.
* Kiếm tra các thông số vacuum config đã phù hợp với kích thước của bảng chưa.
* Chạy VACUUM FULL trong maintenance window.
    Quá trình vacuum không giúp giải phóng disk space. Mục đích của nó là có thể tái sử dụng vùng space đó, chống phân mảnh table
    Vaccuum full -> giải phóng bộ nhớ và giảm kích thước tệp
* Điều tra transaction dài làm cản VACUUM.

## Q4. Query chạy rất chậm, làm thế nào để biết root-cause?

* EXPLAIN ANALYZE → xem có dùng index không.
* Kiểm tra sequential scan → thêm index nếu cần.
* Kiểm tra:
    Sort Method: external merge Disk → tăng work_mem.
    Hash Cond quá lớn → cần phân trang.
    Rows Removed by Filter nhiều → thiếu điều kiện.

## Q5. Khi nào không nên tạo index?

* Table quá nhỏ (<10k rows).
* Column thay đổi thường xuyên (UPDATE/INSERT cost cao).
* Column có độ phân tán thấp (ví dụ boolean, giới tính).
* Khi query không dùng index → dùng function không indexable.

## Q6. Query JSONB chậm?

Khi query bằng: metadata->>'city' = 'Hanoi'

PostgreSQL phải: Mở từng row -> Giải mã JSONB -> Tìm key -> So sánh value
➡️ Scan toàn bảng
➡️ Không dùng index → query chậm
-> Tạo GIN index với jsonb
hạn chế query sâu trong jsonb (nếu cần query nhiều -> tách cột)

## Q7. Hệ thống nhiều service, PSQL bị quá tải: CPU 100%, connection bắn lên 2000. Làm sao xử lý?

* Giảm max_pool_size của app.
    mỗi db có số lượng max_connection nhất định
    max_pool_size quá cao -> quá tải cpu, ram, chậm vacuum, tăng deadlock -> treo DB or downtime
    max_pool_size quá thấp -> request chờ conn lâu, timeout trên service
    max_pool_size = (CPU cores * 2) + hiệu suất IO
* Dùng PgBouncer ở chế độ transaction pooling.
* Refactor các query nặng.
* Cache dữ liệu đọc nhiều (Redis).

## Q8. Một bảng có 500 triệu bản ghi, query theo ngày rất chậm. Solution?

* Dùng partition theo RANGE của created_at.
* Tạo PK + index trên mỗi partition.
* Query tự động pruning partition → nhanh hơn.

## Q9. Khi insert vào bảng partitioned nhưng không khớp partition key sẽ bị lỗi. Xử lý sao?

* Tạo default partition.
* Validate input trước khi insert.

## Q10. Cần thêm column mới cho bảng lớn (100M rows). Làm sao thêm mà không lock hệ thống quá lâu?

* Thêm column không default (rất nhanh):
* Sau đó update dần theo batch.
* Cuối cùng set default

## Q11. Bảng bị xóa nhầm (DELETE), cần phục hồi nhanh. Làm thế nào?

* Nếu có PITR: Restore WAL logs tại timestamp → lấy dữ liệu → copy lại.
* Nếu không: Check replica → copy từ replica về.
* Nếu không: Nếu table có logical replication → apply lại.

## Q12. Làm sao giảm dump/restore time cho DB lớn (100GB+)

* Dùng pg_dump với --jobs=4.
* Index dump riêng.
* ZFS snapshot (nếu dùng ZFS).
* Replica + failover thay vì restore.

