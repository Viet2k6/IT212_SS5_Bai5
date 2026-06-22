# BÀI 5: Sáng tạo (Thiết kế Quy trình & Prompt cho Bộ Giới hạn Tần suất - Rate Limiter)

## 1. Mô tả ý đồ thiết kế quy trình 3 bước

Mục tiêu của quy trình là tận dụng AI theo hướng tư duy kiến trúc phần mềm thay vì yêu cầu viết mã ngay từ đầu.

Quy trình gồm 3 bước:

- Bước 1: Yêu cầu AI đóng vai kiến trúc sư hệ thống để phân tích nhiều thuật toán Rate Limiting khác nhau và đánh giá ưu nhược điểm của từng giải pháp.
- Bước 2: Đưa thêm ngữ cảnh hệ thống phân tán (Distributed System) để AI nhận diện các vấn đề phát sinh khi triển khai trên nhiều máy chủ và đề xuất Redis làm bộ nhớ dùng chung.
- Bước 3: Sau khi đã lựa chọn được kiến trúc phù hợp, yêu cầu AI áp dụng Chain-of-Thought (CoT) để suy luận từng bước và sinh mã nguồn Java Spring Boot hoàn chỉnh, bao gồm xử lý ngoại lệ và cơ chế dự phòng (Fallback).

Cách tiếp cận này giúp giảm nguy cơ AI sinh mã sai kiến trúc và đảm bảo giải pháp có thể triển khai thực tế.

---

# 2. Bước 1 - Prompt tư vấn thuật toán

## Prompt

```text
Bạn là một Software Architect chuyên thiết kế API Gateway cho các hệ thống tài chính quy mô lớn.

Bối cảnh:

Tôi đang xây dựng hệ thống SafePay API Gateway.

Yêu cầu nghiệp vụ:

- Mỗi API Key chỉ được gửi tối đa 100 requests trong vòng 1 phút.
- Nếu vượt quá giới hạn phải trả về HTTP 429 (Too Many Requests).

Nhiệm vụ:

Hãy đề xuất ít nhất 2 thuật toán Rate Limiting phổ biến:

1. Fixed Window
2. Token Bucket

Đối với mỗi thuật toán:

- Giải thích nguyên lý hoạt động.
- Mô tả cách triển khai bằng Java.
- Phân tích ưu điểm.
- Phân tích nhược điểm.
- Đánh giá khả năng mở rộng.

Sau đó lập bảng Trade-offs so sánh:

- Độ chính xác
- Hiệu năng
- Độ phức tạp triển khai
- Khả năng mở rộng
- Phù hợp với hệ thống tài chính

Cuối cùng hãy đưa ra khuyến nghị thuật toán phù hợp nhất cho SafePay và giải thích lý do.
```

---

# 3. Bước 2 - Prompt phân tích hệ thống phân tán

## Prompt

```text
Tiếp tục với giải pháp Rate Limiting đã chọn.

Bối cảnh mới:

Hệ thống SafePay hiện được triển khai trên 3 máy chủ Spring Boot chạy song song phía sau Load Balancer.

Nếu bộ đếm request được lưu trong RAM cục bộ của từng server thì giới hạn 100 requests/phút sẽ không còn chính xác.

Nhiệm vụ:

1. Phân tích vì sao Local Memory Rate Limiter bị sai trong môi trường Distributed System.

2. Thiết kế kiến trúc Rate Limiting sử dụng Redis làm kho lưu trữ tập trung.

3. Giải thích:

- Redis lưu dữ liệu gì.
- Key được thiết kế như thế nào.
- TTL được sử dụng ra sao.
- Cách đồng bộ dữ liệu giữa các node.

4. Phân tích:

- Ưu điểm.
- Nhược điểm.
- Các rủi ro khi Redis bị quá tải hoặc mất kết nối.

5. Đưa ra sơ đồ kiến trúc ASCII minh họa luồng:

Client -> Load Balancer -> API Gateway -> Redis -> Application
```

---

# 4. Bước 3 - Prompt sinh mã nguồn bằng CoT

## Prompt

```text
Bạn là Senior Java Developer và Distributed System Engineer.

Hãy triển khai Rate Limiter cho hệ thống SafePay bằng:

- Java 17
- Spring Boot 3
- Spring Data Redis
- Lombok
- SLF4J

Yêu cầu:

Mỗi API Key chỉ được gửi tối đa:

100 requests / phút

Nếu vượt quá giới hạn:

- Trả về HTTP 429

Quy trình suy luận (Chain-of-Thought):

Đừng viết code ngay.

Trước tiên hãy phân tích từng bước:

1. Thiết kế cấu trúc dữ liệu Redis.
2. Thiết kế key và TTL.
3. Luồng xử lý request.
4. Cách phát hiện vượt giới hạn.
5. Cơ chế xử lý lỗi Redis.
6. Cơ chế Fallback.

Sau khi hoàn thành phân tích:

Sinh mã nguồn hoàn chỉnh gồm:

- RedisConfig
- RateLimiterService
- RateLimitFilter
- Exception Handler

Ràng buộc:

- Ghi log bằng @Slf4j.
- Không sử dụng biến static toàn cục.
- Xử lý RedisConnectionFailureException.
- Nếu Redis sập:

    + Chuyển sang Local In-Memory Rate Limiter tạm thời.

    + Ghi log WARNING.

- Tuân thủ Clean Code.
- Có chú thích tiếng Việt.
```

---

# 5. Minh chứng thực tế

## Lượt chat 1 - So sánh thuật toán

### Prompt

```text
(Bước 1)
```

### AI trả lời

```text
Thuật toán 1: Fixed Window

Ưu điểm:
- Đơn giản.
- Dễ triển khai.

Nhược điểm:
- Có thể xảy ra burst traffic tại ranh giới thời gian.

Thuật toán 2: Token Bucket

Ưu điểm:
- Mượt hơn.
- Hỗ trợ burst hợp lý.

Nhược điểm:
- Phức tạp hơn.

Kết luận:
Token Bucket phù hợp hơn cho hệ thống SafePay.
```

---

## Lượt chat 2 - Thiết kế phân tán với Redis

### Prompt

```text
(Bước 2)
```

### AI trả lời

```text
Local Memory Rate Limiter bị sai vì:

Server A:
50 requests

Server B:
50 requests

Server C:
50 requests

Tổng:
150 requests

Nhưng mỗi server chỉ thấy 50 requests.

Giải pháp:

Redis làm bộ nhớ tập trung.

Key:

RATE_LIMIT:{apiKey}

Value:

Request Count

TTL:

60 giây

Kiến trúc:

Client
  |
Load Balancer
  |
API Gateway
  |
 Redis
  |
Application
```

---

## Lượt chat 3 - Sinh mã nguồn cuối cùng

### Prompt

```text
(Bước 3)
```

### AI trả lời

```java
@Service
@Slf4j
@RequiredArgsConstructor
public class RateLimiterService {

    private final StringRedisTemplate redisTemplate;

    private final Map<String, AtomicInteger> fallbackCache =
            new ConcurrentHashMap<>();

    public boolean allowRequest(String apiKey) {

        try {

            String key = "RATE_LIMIT:" + apiKey;

            Long count = redisTemplate.opsForValue().increment(key);

            if (count == 1) {
                redisTemplate.expire(key, Duration.ofMinutes(1));
            }

            return count <= 100;

        } catch (RedisConnectionFailureException ex) {

            log.warn("Redis unavailable. Switching to fallback limiter.");

            AtomicInteger counter =
                    fallbackCache.computeIfAbsent(
                            apiKey,
                            k -> new AtomicInteger(0)
                    );

            return counter.incrementAndGet() <= 100;
        }
    }
}
```

---

# 6. Kết luận

Quy trình 3 bước giúp AI đi từ:

1. Phân tích thuật toán.
2. Thiết kế kiến trúc phân tán.
3. Sinh mã nguồn hoàn chỉnh.

Thông qua Multiple Options, Trade-offs, What-if Scenario và Chain-of-Thought, giải pháp cuối cùng đảm bảo:

- Chính xác về mặt nghiệp vụ.
- Hỗ trợ môi trường Distributed System.
- Có khả năng mở rộng.
- Có cơ chế Fallback khi Redis gặp sự cố.
- Phù hợp với các hệ thống tài chính thực tế.

```

```
