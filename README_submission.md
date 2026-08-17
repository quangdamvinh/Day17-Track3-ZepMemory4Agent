# Lab 17 — Multi-Memory Agent with Zep

## 1. Kết quả benchmark

Memory-enabled agent đạt **11/11 PASS**, evidence hit rate **100%**, latency retrieval trung bình **618.5 ms** và token reduction trung bình **20.2%**. Toàn bộ `pytest` cũng PASS.

So với no-memory baseline chỉ đạt **2/11 PASS (18.2%)**, memory-enabled tăng **81.8 điểm phần trăm evidence hit rate** và thêm 9 case PASS. Điều này cho thấy memory retrieval đặc biệt quan trọng với thông tin nằm ngoài conversation hiện tại.

## 2. Phân tích memory

**Layer quan trọng nhất trong bộ test này là long-term memory**, vì có nhiều case nhất và xử lý các thông tin cần duy trì qua session như preference, TODO và project context. E02, E03, E08 và E09 đều PASS, trong đó E09 còn kiểm tra user isolation.

Không có layer nào có hit rate thấp hơn các layer khác: **short-term, long-term, episodic và semantic đều đạt 100% trên các case được giao**. Vì vậy, điểm yếu không nằm ở retrieval accuracy của một layer cụ thể mà chủ yếu ở chi phí retrieval, đặc biệt với long-term.

Case retrieve nhiều token nhất là **E03 với 745 tokens**, tiếp theo là E02 với 744 tokens. Điều này cho thấy Context Block có thể chứa nhiều thông tin liên quan hơn mức tối thiểu cần thiết cho một query.

Case **E07 (mixed)** cần kết hợp **long-term + semantic memory**: long-term cung cấp preference `Python`, trong khi semantic cung cấp rule `Idempotency-Key`.

## 3. Token budget và no-memory

Memory-enabled chỉ giảm trung bình **20.2% tokens**, nhưng vẫn đạt 100% evidence hit rate. Ngược lại, no-memory đạt **81.8% token reduction** vì gần như không retrieve context, nhưng chỉ đạt 18.2% hit rate. Vì vậy, token reduction không thể được đánh giá độc lập; giảm context quá mạnh có thể khiến agent rẻ hơn nhưng mất evidence cần thiết.

## 4. Recency và compaction

E08 kiểm tra **recency/conflict handling**: preference mới hơn về `BLUEBIRD-42` phải được ưu tiên, với `TypeScript` và `NestJS` được recall chính xác. E10 cho thấy compaction cần giữ các thông tin có giá trị lâu dài như constraint/deadline thông qua durable notes, thay vì chỉ giữ các recent turns.

## 5. Zep và memory safety

Zep Context Block giúp xây dựng managed cross-session memory và retrieval theo user, trong khi Redis + Qdrant cho phép tự kiểm soát storage nhưng phải tự xây nhiều logic memory hơn. Trade-off chính là **simplicity và managed graph/context của Zep** so với **control, customization và self-hosting của Redis + Qdrant**.

Để chống memory poisoning, durable memory cần có **provenance, user-scoped isolation, validation và giới hạn loại thông tin được phép ghi vào memory**. Các instruction độc hại không nên mặc nhiên trở thành durable memory chỉ vì chúng xuất hiện trong conversation.