# BÀI 1: THỰC HÀNH TỐI ƯU HÓA LƯU TRỮ GIỎ HÀNG

## What-if Scenario & Multiple Options

## 1. Phân tích ngắn gọn bối cảnh

Trong hệ thống thương mại điện tử SpeedyCart, giỏ hàng tạm thời là thành phần có tần suất đọc/ghi rất cao vì người dùng liên tục thêm sản phẩm, xóa sản phẩm hoặc thay đổi số lượng. Nếu mọi thao tác này đều ghi trực tiếp vào SQL Database thì hệ thống dễ bị nghẽn, tăng độ trễ và làm giảm trải nghiệm người dùng.

Vì vậy, bài toán không chỉ là chọn một công nghệ lưu trữ nhanh hơn, mà cần đánh giá đa chiều giữa tốc độ, độ an toàn dữ liệu, khả năng khôi phục khi lỗi và độ phức tạp triển khai. Do đó, prompt gửi cho AI cần được thiết kế theo hướng **Multiple Options**, **Trade-offs**, **What-if Scenario** và **Implementation** để AI không trả lời chung chung mà đưa ra phương án có thể áp dụng thực tế.

---

## 2. Prompt tối ưu do nhóm thiết kế

### 5 thành phần của Prompt

**1. Vai trò:**
AI đóng vai trò Kiến trúc sư hệ thống.

**2. Bối cảnh:**
Hệ thống SpeedyCart có giỏ hàng tạm thời đọc/ghi nhiều, SQL Database đang bị nghẽn.

**3. Nhiệm vụ:**
Đề xuất nhiều phương án lưu trữ, so sánh trade-off, phân tích kịch bản Redis crash và cung cấp cấu hình Java Spring Boot.

**4. Ràng buộc/yêu cầu:**
Có ít nhất 3 phương án, có bảng so sánh, có phân tích RDB/AOF, fallback SQL và code RedisTemplate an toàn.

**5. Định dạng đầu ra:**
Trình bày theo các mục rõ ràng: phân tích, bảng so sánh, đề xuất kiến trúc, kịch bản What-if, mã nguồn Java.

### Nội dung Prompt hoàn chỉnh

Bạn hãy đóng vai trò là một **System Architect** có kinh nghiệm thiết kế hệ thống thương mại điện tử quy mô lớn.

Bối cảnh:
SpeedyCart là một website thương mại điện tử. Chức năng giỏ hàng tạm thời của khách hàng có tần suất đọc/ghi rất lớn vì người dùng liên tục thêm sản phẩm, xóa sản phẩm và thay đổi số lượng. Hiện tại hệ thống đang ghi trực tiếp mọi thay đổi vào SQL Database, dẫn đến nghẽn cơ sở dữ liệu, tăng độ trễ và làm giảm trải nghiệm người dùng.

Yêu cầu phân tích và thiết kế giải pháp:

1. Hãy đề xuất ít nhất 3 phương án công nghệ để lưu trữ dữ liệu giỏ hàng tạm thời, ví dụ:

    * SQL Database truyền thống
    * In-memory Redis Cache
    * Client-side Session/Cookie
    * Có thể bổ sung phương án Hybrid nếu phù hợp

2. Hãy lập bảng so sánh chi tiết các phương án theo các tiêu chí:

    * Tốc độ đọc/ghi
    * Tính đồng nhất dữ liệu
    * Khả năng chịu lỗi/mất mát dữ liệu
    * Độ phức tạp triển khai
    * Tình huống phù hợp để sử dụng

3. Hãy phân tích kịch bản What-if:
   “Chuyện gì xảy ra nếu cụm máy chủ Redis đột ngột bị crash hoặc mất nguồn điện?”
   Hãy giải thích rủi ro mất dữ liệu giỏ hàng và đề xuất cách giảm thiểu bằng:

    * Cấu hình Redis Persistence RDB
    * Cấu hình Redis Persistence AOF
    * Kết hợp fallback ghi tạm xuống SQL Database
    * Cơ chế khôi phục dữ liệu giỏ hàng sau khi Redis hoạt động trở lại

4. Hãy đề xuất kiến trúc tối ưu cho SpeedyCart:

    * Luồng ghi dữ liệu khi người dùng cập nhật giỏ hàng
    * Luồng đọc dữ liệu khi người dùng mở lại giỏ hàng
    * Cơ chế đồng bộ Redis và SQL
    * TTL hợp lý cho giỏ hàng tạm thời

5. Hãy cung cấp đoạn mã cấu hình kết nối Redis bằng Java Spring Boot sử dụng RedisTemplate an toàn, bao gồm:

    * RedisConnectionFactory
    * RedisTemplate<String, Object>
    * Serializer phù hợp cho key và value
    * Code rõ ràng, dễ bảo trì, có comment giải thích ngắn gọn

Yêu cầu đầu ra:

* Trả lời có cấu trúc rõ ràng.
* Có bảng so sánh.
* Có phân tích ưu điểm, nhược điểm và trade-off.
* Có phần What-if Scenario.
* Có đoạn mã Java Spring Boot hoàn chỉnh.
* Kết luận nên chọn phương án nào cho SpeedyCart và vì sao.

---

## 3. Minh chứng chạy thực tế: Log phản hồi của AI

### 3.1. Phân tích tổng quan

Với bài toán giỏ hàng tạm thời của SpeedyCart, yêu cầu quan trọng nhất là tốc độ đọc/ghi nhanh, giảm tải cho SQL Database và vẫn hạn chế mất dữ liệu khi có sự cố. Giỏ hàng là dữ liệu có tính tạm thời, nhưng vẫn ảnh hưởng trực tiếp đến trải nghiệm mua hàng và doanh thu. Vì vậy, không nên chỉ lưu hoàn toàn trong SQL hoặc hoàn toàn trên client, mà nên thiết kế theo hướng kết hợp.

Một giải pháp phù hợp là dùng Redis làm nơi lưu trữ chính cho giỏ hàng tạm thời vì Redis có tốc độ đọc/ghi rất nhanh. Tuy nhiên, Redis là in-memory storage nên cần cấu hình persistence và có cơ chế fallback xuống SQL Database để giảm nguy cơ mất dữ liệu khi Redis crash.

---

### 3.2. Các phương án công nghệ lưu trữ giỏ hàng tạm thời

#### Phương án 1: SQL Database truyền thống

SQL Database có ưu điểm là dữ liệu bền vững, dễ truy vấn và đảm bảo tính toàn vẹn dữ liệu tốt. Tuy nhiên, với giỏ hàng có tần suất cập nhật liên tục, việc ghi trực tiếp mọi thay đổi xuống SQL sẽ gây áp lực lớn lên database, làm tăng latency và giảm khả năng mở rộng.

Phương án này phù hợp với hệ thống nhỏ, lượng truy cập thấp hoặc dùng để lưu bản sao dự phòng của giỏ hàng.

#### Phương án 2: Redis In-memory Cache

Redis lưu dữ liệu trong bộ nhớ nên tốc độ đọc/ghi rất nhanh, phù hợp với dữ liệu tạm thời như shopping cart. Redis cũng hỗ trợ TTL để tự động xóa giỏ hàng sau một khoảng thời gian không hoạt động.

Nhược điểm là nếu không cấu hình persistence, dữ liệu có thể bị mất khi Redis crash hoặc mất điện. Vì vậy cần kết hợp RDB, AOF hoặc fallback xuống SQL.

#### Phương án 3: Client-side Session/Cookie

Phương án này lưu giỏ hàng ở phía client, ví dụ trong cookie, localStorage hoặc session storage. Ưu điểm là giảm tải cho server và triển khai đơn giản. Tuy nhiên, dữ liệu dễ bị sửa đổi, giới hạn dung lượng và khó đồng bộ giữa nhiều thiết bị.

Phương án này chỉ phù hợp với giỏ hàng đơn giản, người dùng chưa đăng nhập hoặc hệ thống nhỏ.

#### Phương án 4: Hybrid Redis + SQL Database

Đây là phương án kết hợp: Redis dùng để đọc/ghi nhanh trong thời gian thực, SQL dùng làm nơi lưu dự phòng hoặc lưu giỏ hàng quan trọng của người dùng đã đăng nhập.

Phương án này cân bằng giữa hiệu năng và độ an toàn dữ liệu, phù hợp với hệ thống thương mại điện tử có lượng truy cập lớn.

---

### 3.3. Bảng so sánh các phương án

| Phương án                  | Tốc độ đọc/ghi                  | Tính đồng nhất dữ liệu           | Khả năng chịu lỗi/mất dữ liệu       | Độ phức tạp triển khai | Tình huống phù hợp                      |
| -------------------------- | ------------------------------- | -------------------------------- | ----------------------------------- | ---------------------- | --------------------------------------- |
| SQL Database               | Trung bình đến chậm khi tải cao | Cao                              | Tốt, dữ liệu bền vững               | Thấp đến trung bình    | Hệ thống nhỏ, cần dữ liệu ổn định       |
| Redis In-memory            | Rất nhanh                       | Trung bình, cần thiết kế đồng bộ | Có rủi ro nếu không bật persistence | Trung bình             | Giỏ hàng tạm thời, traffic lớn          |
| Client-side Session/Cookie | Nhanh, giảm tải server          | Thấp, dễ bị lệch dữ liệu         | Phụ thuộc trình duyệt người dùng    | Thấp                   | Giỏ hàng đơn giản, khách chưa đăng nhập |
| Hybrid Redis + SQL         | Nhanh và an toàn hơn            | Cao hơn nếu đồng bộ tốt          | Tốt nếu có persistence và fallback  | Cao hơn                | Website thương mại điện tử quy mô lớn   |

---

### 3.4. Phân tích trade-off

Nếu chọn SQL Database hoàn toàn, hệ thống dễ đảm bảo dữ liệu nhưng khó đáp ứng hiệu năng cao khi lượng người dùng lớn. Nếu chọn Redis hoàn toàn, tốc độ rất tốt nhưng cần quan tâm đến persistence và khôi phục dữ liệu. Nếu chọn client-side storage, hệ thống nhẹ hơn nhưng giảm khả năng kiểm soát dữ liệu và dễ gặp vấn đề bảo mật.

Phương án Hybrid Redis + SQL có độ phức tạp triển khai cao hơn nhưng phù hợp nhất với SpeedyCart vì giải quyết đồng thời hai vấn đề: tăng tốc độ đọc/ghi và giảm rủi ro mất dữ liệu.

---

### 3.5. What-if Scenario: Redis bị crash hoặc mất điện

Nếu cụm Redis đột ngột bị crash hoặc mất nguồn điện, dữ liệu giỏ hàng đang nằm trong RAM có thể bị mất nếu Redis chưa kịp ghi xuống ổ đĩa. Khi người dùng quay lại website, giỏ hàng có thể bị trống hoặc thiếu sản phẩm, gây ảnh hưởng đến trải nghiệm và tỷ lệ chuyển đổi đơn hàng.

Để giảm thiểu rủi ro này, có thể áp dụng các cơ chế sau:

#### Cơ chế 1: Redis RDB Persistence

RDB tạo snapshot dữ liệu Redis tại một thời điểm nhất định. Khi Redis khởi động lại, hệ thống có thể phục hồi dữ liệu từ snapshot gần nhất.

Ưu điểm:

* Ít ảnh hưởng đến hiệu năng.
* Phù hợp để backup định kỳ.
* File RDB nhỏ gọn, dễ phục hồi.

Nhược điểm:

* Có thể mất dữ liệu phát sinh sau snapshot gần nhất.
* Không phù hợp nếu yêu cầu gần như không mất dữ liệu.

#### Cơ chế 2: Redis AOF Persistence

AOF ghi lại các lệnh thay đổi dữ liệu vào file log. Khi Redis khởi động lại, Redis có thể chạy lại các lệnh trong AOF để phục hồi dữ liệu.

Ưu điểm:

* Giảm nguy cơ mất dữ liệu hơn RDB.
* Có thể cấu hình appendfsync everysec để cân bằng hiệu năng và độ an toàn.

Nhược điểm:

* File AOF có thể lớn hơn RDB.
* Có thể ảnh hưởng đến hiệu năng ghi nếu cấu hình quá chặt.

#### Cơ chế 3: Kết hợp RDB + AOF

Có thể bật cả RDB và AOF để vừa có snapshot định kỳ, vừa có log ghi lệnh gần thời gian thực. Cách này tăng khả năng phục hồi dữ liệu khi Redis gặp sự cố.

#### Cơ chế 4: Fallback ghi tạm xuống SQL Database

Khi người dùng cập nhật giỏ hàng, hệ thống ghi chính vào Redis. Đồng thời, với các thao tác quan trọng hoặc theo chu kỳ, hệ thống ghi bản sao xuống SQL Database.

Khi Redis bị lỗi:

1. Ứng dụng phát hiện Redis không khả dụng.
2. Tạm thời đọc/ghi giỏ hàng từ SQL Database.
3. Khi Redis hoạt động trở lại, hệ thống nạp lại dữ liệu giỏ hàng từ SQL vào Redis.
4. Tiếp tục dùng Redis làm tầng đọc/ghi nhanh.

---

### 3.6. Kiến trúc đề xuất cho SpeedyCart

Phương án tối ưu là **Hybrid Redis + SQL Database**.

#### Luồng ghi khi người dùng cập nhật giỏ hàng

1. Người dùng thêm/xóa/sửa số lượng sản phẩm.
2. API Cart Service nhận request.
3. Cart Service ghi dữ liệu vào Redis với key dạng: `cart:{userId}`.
4. Redis đặt TTL, ví dụ 24 giờ hoặc 7 ngày tùy loại người dùng.
5. Hệ thống ghi bất đồng bộ bản sao xuống SQL Database theo cơ chế background job hoặc message queue.
6. Nếu Redis lỗi, hệ thống fallback ghi trực tiếp xuống SQL.

#### Luồng đọc khi người dùng mở lại giỏ hàng

1. Cart Service kiểm tra Redis theo `cart:{userId}`.
2. Nếu có dữ liệu trong Redis, trả về ngay cho người dùng.
3. Nếu Redis không có dữ liệu, kiểm tra SQL Database.
4. Nếu SQL có dữ liệu, nạp lại vào Redis và trả về cho người dùng.
5. Nếu cả Redis và SQL đều không có, trả về giỏ hàng rỗng.

#### TTL đề xuất

* Khách chưa đăng nhập: TTL 24 giờ.
* Khách đã đăng nhập: TTL 7 ngày đến 30 ngày.
* Khi người dùng đặt hàng thành công: xóa cart khỏi Redis và cập nhật trạng thái trong SQL.

---

### 3.7. Mã nguồn cấu hình RedisTemplate bằng Java Spring Boot

```java
package com.speedycart.config;

import com.fasterxml.jackson.annotation.JsonTypeInfo;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.jsontype.BasicPolymorphicTypeValidator;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {

    @Value("${spring.data.redis.host:localhost}")
    private String redisHost;

    @Value("${spring.data.redis.port:6379}")
    private int redisPort;

    @Value("${spring.data.redis.password:}")
    private String redisPassword;

    /**
     * Cấu hình RedisConnectionFactory sử dụng Lettuce.
     * Lettuce là Redis client phổ biến trong Spring Boot.
     */
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration config =
                new RedisStandaloneConfiguration(redisHost, redisPort);

        if (redisPassword != null && !redisPassword.isBlank()) {
            config.setPassword(redisPassword);
        }

        return new LettuceConnectionFactory(config);
    }

    /**
     * Cấu hình RedisTemplate an toàn hơn so với Java default serialization.
     * Key dùng StringRedisSerializer để dễ đọc trong Redis.
     * Value dùng JSON serializer để dễ debug và tránh phụ thuộc Java native serialization.
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(
            RedisConnectionFactory redisConnectionFactory
    ) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);

        StringRedisSerializer stringSerializer = new StringRedisSerializer();

        ObjectMapper objectMapper = new ObjectMapper();

        BasicPolymorphicTypeValidator typeValidator =
                BasicPolymorphicTypeValidator.builder()
                        .allowIfSubType("com.speedycart")
                        .allowIfSubType("java.util")
                        .build();

        objectMapper.activateDefaultTyping(
                typeValidator,
                ObjectMapper.DefaultTyping.NON_FINAL,
                JsonTypeInfo.As.PROPERTY
        );

        GenericJackson2JsonRedisSerializer jsonSerializer =
                new GenericJackson2JsonRedisSerializer(objectMapper);

        template.setKeySerializer(stringSerializer);
        template.setHashKeySerializer(stringSerializer);

        template.setValueSerializer(jsonSerializer);
        template.setHashValueSerializer(jsonSerializer);

        template.afterPropertiesSet();
        return template;
    }
}
```

---

### 3.8. Ví dụ cấu hình application.yml

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password:
      timeout: 3000ms
```

---

### 3.9. Gợi ý cấu hình Redis Persistence

```conf
# Bật RDB snapshot
save 900 1
save 300 10
save 60 10000

# Bật AOF
appendonly yes

# Ghi AOF mỗi giây để cân bằng giữa hiệu năng và an toàn dữ liệu
appendfsync everysec
```

Ý nghĩa:

* `save 900 1`: Sau 900 giây nếu có ít nhất 1 thay đổi thì tạo snapshot.
* `save 300 10`: Sau 300 giây nếu có ít nhất 10 thay đổi thì tạo snapshot.
* `save 60 10000`: Sau 60 giây nếu có ít nhất 10000 thay đổi thì tạo snapshot.
* `appendonly yes`: Bật AOF để ghi lại các thao tác thay đổi dữ liệu.
* `appendfsync everysec`: Ghi log AOF xuống đĩa mỗi giây, giúp giảm nguy cơ mất dữ liệu.

---

### 3.10. Kết luận đề xuất

Đối với SpeedyCart, phương án phù hợp nhất là sử dụng **Hybrid Redis + SQL Database**.

Redis đảm nhiệm vai trò lưu trữ giỏ hàng tạm thời để tăng tốc độ đọc/ghi và giảm tải cho SQL Database. SQL Database đóng vai trò lưu trữ dự phòng, giúp khôi phục dữ liệu khi Redis gặp sự cố. Ngoài ra, cần bật Redis Persistence bằng RDB/AOF và xây dựng cơ chế fallback để đảm bảo giỏ hàng của khách hàng không bị biến mất hoàn toàn khi hệ thống gặp lỗi.

Phương án này có độ phức tạp triển khai cao hơn nhưng phù hợp với hệ thống thương mại điện tử cần hiệu năng cao, khả năng mở rộng tốt và trải nghiệm người dùng ổn định.
