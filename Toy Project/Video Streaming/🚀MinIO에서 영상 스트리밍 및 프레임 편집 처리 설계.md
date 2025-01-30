
웹소켓을 통해 실시간으로 영상을 스트리밍하면서 프레임 편집을 하는 경우, **Pre-Signed URL**을 매번 생성하는 것은 비효율적입니다. 대신 더 적합한 접근 방식을 사용할 수 있습니다.

---

## 📚 **1️⃣ Pre-Signed URL이 비효율적인 이유**

- **유효 시간 문제:** Pre-Signed URL은 유효 시간 만료 후 재생성해야 합니다.
- **빈번한 요청:** 실시간 프레임 편집 중 매번 URL을 생성하면 오버헤드가 발생합니다.
- **보안 문제:** 너무 긴 만료 시간은 보안에 취약해질 수 있습니다.

---

## ✅ **2️⃣ 권장 아키텍처**

### 🎯 **웹소켓 + 백엔드 API + MinIO 통합**

1. **프론트엔드 (WebSocket):**
    
    - 웹소켓을 통해 사용자의 편집 요청을 실시간으로 백엔드에 전달합니다.
    - 편집된 프레임 데이터를 다시 웹소켓을 통해 수신합니다.
2. **백엔드 (Spring Boot):**
    
    - MinIO에서 영상을 **스트림(Stream)**으로 읽어옵니다.
    - 스트림 데이터를 WebSocket을 통해 프론트에 전달합니다.
    - 편집된 프레임을 MinIO에 저장합니다.
3. **MinIO:**
    
    - 백엔드에서 API를 통해 영상을 읽고, 수정된 프레임을 다시 저장합니다.
    - 모든 작업은 백엔드가 관리하며 Pre-Signed URL 없이 수행됩니다.

---

## 🛠️ **3️⃣ 스트리밍 예제**

### ✅ **3.1 백엔드 – 영상 스트림 API**

#### 📑 **MinioVideoService.java**

```java
import io.minio.GetObjectArgs;
import io.minio.MinioClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.io.InputStream;

@Service
public class MinioVideoService {

    @Autowired
    private MinioClient minioClient;

    private final String bucket = "video-bucket";

    public InputStream getVideoStream(String videoPath) {
        try {
            return minioClient.getObject(
                GetObjectArgs.builder()
                    .bucket(bucket)
                    .object(videoPath)
                    .build()
            );
        } catch (Exception e) {
            throw new RuntimeException("Failed to stream video: " + e.getMessage());
        }
    }
}
```

---

### ✅ **3.2 웹소켓을 통한 실시간 영상 전송**

#### 📑 **VideoStreamWebSocketHandler.java**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;

import java.io.InputStream;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

@Component
public class VideoStreamWebSocketHandler extends TextWebSocketHandler {

    @Autowired
    private MinioVideoService minioVideoService;

    private final ExecutorService executorService = Executors.newFixedThreadPool(5);

    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage message) {
        String videoPath = message.getPayload();

        executorService.submit(() -> {
            try (InputStream videoStream = minioVideoService.getVideoStream(videoPath)) {
                byte[] buffer = new byte[4096];
                int bytesRead;

                while ((bytesRead = videoStream.read(buffer)) != -1) {
                    session.sendMessage(new TextMessage(new String(buffer, 0, bytesRead)));
                }

                session.close();
            } catch (Exception e) {
                throw new RuntimeException("Error streaming video: " + e.getMessage());
            }
        });
    }
}
```

---

### ✅ **3.3 프론트엔드 – WebSocket 클라이언트 예제**

```javascript
const socket = new WebSocket('ws://localhost:8080/video-stream');

socket.onopen = () => {
    // 스트리밍할 영상 경로 요청
    socket.send('path/to/video.mp4');
};

socket.onmessage = (event) => {
    const blob = new Blob([event.data], { type: 'video/mp4' });
    const url = URL.createObjectURL(blob);
    document.getElementById('videoPlayer').src = url;
};

socket.onerror = (error) => {
    console.error('WebSocket Error:', error);
};

socket.onclose = () => {
    console.log('WebSocket Connection Closed');
};
```

**HTML 예제:**

```html
<video id="videoPlayer" controls autoplay></video>
```

---

## ✅ **4️⃣ 프레임 편집 흐름**

1. **프론트엔드 → 백엔드:**
    
    - 사용자가 특정 프레임을 편집하면 웹소켓을 통해 프레임 데이터와 메타데이터를 백엔드로 전송합니다.
2. **백엔드 → MinIO:**
    
    - 백엔드는 해당 프레임을 MinIO에 저장하거나 업데이트합니다.
3. **MinIO → 프론트엔드:**
    
    - 편집된 프레임을 다시 프론트로 스트림 형태로 보냅니다.

---

## 🛡️ **5️⃣ 보안 고려사항**

1. **WebSocket 인증:**
    
    - WebSocket 연결 시 **JWT 토큰** 또는 **API 키**를 사용해 인증합니다.
2. **경로 검증:**
    
    - 사용자가 접근할 수 있는 파일 경로만 스트리밍되도록 검증합니다.
3. **프레임 데이터 검증:**
    
    - 편집된 프레임 데이터가 유효한지 백엔드에서 검증합니다.

---

## 🚀 **6️⃣ 최적화 포인트**

1. **프레임 단위로 스트림 전송:**
    
    - 전체 영상을 전송하지 않고, 필요한 프레임만 실시간으로 전송합니다.
2. **캐싱 사용:**
    
    - 자주 사용되는 영상이나 프레임을 캐싱하여 성능을 개선합니다.
3. **비동기 처리:**
    
    - 백엔드와 MinIO 간 데이터 입출력은 비동기적으로 처리합니다.

---

## ✅ **7️⃣ 비교 정리**

|**방식**|**설명**|**사용 사례**|
|---|---|---|
|**Pre-Signed URL**|일시적 URL 생성, 직접 접근|짧은 시간 동안 접근 허용|
|**WebSocket 스트리밍**|실시간 프레임 단위 스트리밍|실시간 영상 편집, 다중 사용자|

---

## ✅ **8️⃣ 추천 설계**

1. **Pre-Signed URL:**
    
    - 간단한 이미지나 비디오 파일 접근에 사용.
2. **WebSocket 스트리밍:**
    
    - 실시간으로 프레임 편집이 필요한 경우 사용.
    - 백엔드가 MinIO와 직접 통신.

---

## 📝 **9️⃣ 최종 정리**

- 프레임 단위 실시간 스트리밍은 **WebSocket + MinIO 스트림**을 통해 처리.
- Pre-Signed URL은 특정 파일에 일시적 접근이 필요할 때만 사용.
- **권장 구조:**
    
    ```
    [Frontend] ←(WebSocket)→ [Backend] ←(API)→ [MinIO]
    ```
    

---

이제 실시간 영상 스트리밍과 프레임 편집을 위한 설계가 완성되었습니다! 🚀✨  
혹시 더 구체적인 질문이나 도움이 필요하면 말씀해 주세요! 😊