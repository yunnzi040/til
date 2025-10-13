CORS란 무엇이며 이것에 대해서 설명해보세요.

---

### CORS란?
- **정의**: CORS(Cross-Origin Resource Sharing)는 브라우저의 동일 출처 정책(SOP)을 안전하게 완화하기 위해, 서버가 HTTP 응답 헤더로 교차 출처 접근을 선택적으로 허용하는 표준 메커니즘이다.
- **목적**: 여러 서비스 간 데이터 공유 필요를 충족하기 위해, CORS는 서버가 추가 HTTP 헤더를 사용하여 브라우저에게 특정 출처에서 요청된 리소스에 접근할 수 있는 권한을 부여하도록 알려주는 체제이다.

### 동일 출처 정책(SOP)
- **출처(Origin)** 는 프로토콜(Scheme), 호스트(Host), 포트(Port)로 구성된다. 셋이 모두 같아야 동일 출처다.

| 구성 요소 | 예시 |
| --- | --- |
| 프로토콜(Scheme) | https:// |
| 호스트(Host) | good.com |
| 포트(Port) | :443 |

예시

| 페이지 | 주소 | 관계 |
| --- | --- | --- |
| A사이트 | https://good.com:443 | ✅ 같은 출처 |
| B사이트 | https://good.com:8080 | ❌ 포트 다름 → 다른 출처 |
| C사이트 | http://good.com:443 | ❌ 프로토콜 다름 → 다른 출처 |
| D사이트 | https://evil.com:443 | ❌ 도메인 다름 → 다른 출처 |

- **보안 의미**: 악성 사이트가 사용자의 인증정보(쿠키 등)를 악용하여, 사용자가 이미 로그인한 다른 신뢰 사이트의 민감 데이터를 읽거나 기능을 오용하는 것을 막는다.

---

### CORS 작동 핵심: 요청/응답 헤더
- 브라우저는 교차 출처 요청에 항상 `Origin` 요청 헤더를 포함한다.
- 서버는 이를 검토하고, 허용 시 응답에 CORS 관련 헤더를 포함한다. 브라우저는 해당 헤더를 검증한 뒤 JS 접근 권한을 부여한다.

주요 헤더 요약
- 요청 헤더
  - `Origin`: 요청을 시작한 출처(경로 제외)
  - `Access-Control-Request-Method`, `Access-Control-Request-Headers`: 프리플라이트(OPTIONS)에서 본 요청의 메서드/헤더 의도 전달
- 응답 헤더
  - `Access-Control-Allow-Origin`: 허용 출처(자격증명 포함 시 `*` 금지)
  - `Access-Control-Allow-Credentials`: 쿠키/자격증명 허용 여부(true/false)
  - `Access-Control-Allow-Methods`: 허용 메서드 목록(프리플라이트 응답)
  - `Access-Control-Allow-Headers`: 허용 요청 헤더(프리플라이트 응답)
  - `Access-Control-Max-Age`: 프리플라이트 결과 캐시 시간(초)
  - `Access-Control-Expose-Headers`: 클라이언트 JS가 읽을 수 있도록 노출할 응답 헤더 추가
  - `Vary: Origin`: 캐시 계층에서 출처별로 응답을 안전하게 분기

---

### 예시) 내가 이해한 CORS 에러 흐름
상황
- 사용자는 `https://bad.com` 에 접속해 있음
- `bad.com` 의 JS가 `https://good.com/api/user-info` 로 요청을 보냄(쿠키 동반 가능)

흐름
1. 브라우저가 요청에 `Origin: https://bad.com` 을 포함하여 `good.com` 으로 전송
2. 서버(`good.com`)는 응답을 반환하지만, 응답에 `Access-Control-Allow-Origin: https://bad.com` 이 없거나 불일치하면
3. 브라우저는 네트워크 레벨에서는 응답을 수신하더라도, JS에서 응답 본문 접근을 차단하고 CORS 오류를 발생시킨다

콘솔 에러 예시
```
Access to fetch at 'https://good.com/api/user-info' from origin 'https://bad.com' has been blocked by CORS policy.
```

참고
- `<img>` 같은 태그 요청은 응답 본문을 JS로 읽지 못한다. 네트워크 탭에는 보이더라도, 스크립트 접근은 SOP/CORS 규칙에 따라 차단될 수 있다.

---

### 시나리오 1) 단순 요청(Simple Request)
단순 요청은 Preflighted 없이 바로 본 요청을 전송한다. 다음을 모두 만족해야 한다.
- 메서드: GET / HEAD / POST
- `Content-Type`: `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`
- CORS safelisted 요청 헤더만 수동 설정(예: Accept, Accept-Language, Content-Language, Content-Type[제한된 값], Range 등)

동작
1. 브라우저가 `Origin` 포함하여 요청 전송
2. 서버가 응답에 `Access-Control-Allow-Origin`(필요 시 `Access-Control-Allow-Credentials`) 포함
3. 브라우저가 검증 후 JS 접근 허용. 미포함/불일치 시 응답은 수신하더라도 JS 접근 차단(CORS 오류)

### 예시) 단순 요청에서 ACAO 동작을 이해하기 위한 메모
맥락
- 단순 요청의 경우 브라우저는 요청에 `Origin: https://foo.example` 를 포함한다.
- 서버는 응답 시 `Access-Control-Allow-Origin` 으로 허용 출처를 명시한다.

핵심 이해
- 서버는 사전에 설정된 CORS 정책에 따라 허용 범위를 결정한다(특정 출처만 허용 또는 와일드카드 등).
- 허용 범위에 해당하면, 서버는 응답 본문과 함께 `Access-Control-Allow-Origin` 헤더를 포함하여 브라우저로 전송한다.
- 브라우저는 응답의 `Access-Control-Allow-Origin` 값이 요청의 `Origin` 과 일치하면 스크립트가 응답 본문을 읽도록 허용한다.
- 이 값이 없거나 불일치하면 SOP에 의해 JS 접근이 차단된다.

추가 메모
```
Access-Control-Allow-Origin: 서버가 응답에 포함하는 헤더로, 리소스 접근을 허용할 단일 출처를 지정하거나 (자격 증명 요청이 아닐 경우) 와일드카드(*)를 사용하여 모든 출처의 접근을 허용할 수 있다.
```

---

---

### 시나리오 2) 사전 요청(Preflighted Request)
단순 요청 조건을 충족하지 않으면 브라우저는 본 요청 전에 **Preflighted(OPTIONS)** 를 보낸다.

Preflighted 요청 예
```
OPTIONS /api/update-user HTTP/1.1
Host: good.com
Origin: https://bad.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Content-Type
```

Preflighted 응답 예
```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://bad.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 3600
```

서버가 허용을 명시하면 브라우저는 본 요청을 전송한다.

---

### 시나리오 3) 자격 증명 포함 요청(Credentialed Request)
쿠키/HTTP 인증 등 자격증명을 교차 출처로 전송하려면 클라이언트와 서버의 동시 설정이 필요하다.

- 클라이언트
  - fetch: `credentials: 'include'`
  - XHR: `withCredentials = true`
- 서버
  - `Access-Control-Allow-Credentials: true`
  - `Access-Control-Allow-Origin` 에 명시적 출처(요청의 Origin)를 설정해야 하며 `*` 사용 금지

잘못된 예(차단됨)
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```
브라우저 오류: credentials 모드가 include일 때 ACAO가 `*` 일 수 없다.

---

### 자주 겪는 오류와 팁
- `Access-Control-Allow-Origin` 미포함/불일치 → 네트워크 탭에는 응답이 보여도 JS 접근 차단(undefined)
- 크리덴셜 요청에서 `*` 사용 → 차단
- 프리플라이트 응답에 `Access-Control-Allow-Methods/Headers` 누락/불일치 → 본 요청 진행 불가
- 커스텀 응답 헤더를 JS에서 읽어야 한다면 `Access-Control-Expose-Headers` 로 노출
- 캐시 일관성을 위해 `Vary: Origin` 설정 고려

---

### 간단 요약
- SOP는 기본 보안 장치이고, CORS는 이를 통제된 방식으로 완화한다.
- 브라우저는 `Origin` 으로 요청 출처를 알리고, 서버는 CORS 응답 헤더로 허용 범위를 선언한다.
- 단순/사전/자격증명 요청을 구분해 적절한 헤더와 설정을 적용한다.


