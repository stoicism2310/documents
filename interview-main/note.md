- 1M bản ghi cùng update 1 hàng
- query bảng tối ưu nhất
tầng ứng dụng: cache, queue kafka trc

môi trường đo cost 
đồng bộ event kafka queue
đồng bọ event, xử lý nghiệp vụ qua lại

tại sao dùng kafka: 
api nhanh nhất
api có phải lúc nào soogns đâuquá tải -> chết
api chết -> ai retry
request cần trả ngay KQ -> api
trả sau -> kafka
là 1 dạng cdc
- tầng DB ->send: -> outbox pattern
- code : commit tcong -> send

kiểm soát được số lượng thread: thông qua partition (tùy thời điểm lưu lượng cao / thấp -> số lượng queue khác nhau
cơ chế bảo vệ db


tổ hợp nhiều kỹ thuật:
 index, pattition, cutoff định kỳ
caching (validate)
request xử lý sau-> queue
có những ttin cần bổ sung sau -> queue
connection pool
config thread từ hạ tầng -> App
freamework teaching
db ko chịu đc -> đổi DB
rate limitation: giới hạn req trong 1 khoảng tgian (patttern)
theo ip, theo user

expire cache
reload cache

một số exception (number, null...)-> DLQ
exception ko rõ -> retry time 
mục đích ko bị treo, ko bị mất
commit bản tin
tránh lỗi connec timeout DB đẩy vào DB
DB chết -> dừng consumer -> báo VHKT

sự cố:
xem apm, lệnh sql
xem số lượn req
có bị treo bản tin ko, consumer?

dữ liệu có tính ràng buộc
hỗ trợ mạnh join
ES ko làm master data được?
es commit ->index -> mới query đc

golang: sự khác biệt, clean architech