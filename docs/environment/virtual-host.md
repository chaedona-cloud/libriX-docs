# 가상호스트 관리

## 개요

가상호스트(Virtual Host)는 하나의 Liberty 서버에서 여러 도메인 이름을 처리할 수 있도록 하는 기능입니다. LibriX의 가상호스트 관리를 통해 도메인별로 다른 애플리케이션을 매핑하고, 호스트 기반 라우팅을 구성할 수 있습니다.

## 가상호스트란?

가상호스트는 단일 서버에서 여러 도메인 또는 호스트 이름을 구분하여 처리하는 메커니즘입니다. 각 가상호스트에 별도의 애플리케이션을 매핑하여, 도메인에 따라 다른 콘텐츠를 제공할 수 있습니다.

### Liberty의 전통적인 가상호스트 설정 방식

Open Liberty에서 가상호스트를 설정하는 전통적인 방법:

#### server.xml 수동 편집

```xml
<server>
    <!-- 기본 가상호스트 -->
    <virtualHost id="default_host">
        <hostAlias>*:9080</hostAlias>
        <hostAlias>*:9443</hostAlias>
    </virtualHost>
    
    <!-- 커스텀 가상호스트 -->
    <virtualHost id="api_host">
        <hostAlias>api.company.com:9080</hostAlias>
        <hostAlias>api.company.com:9443</hostAlias>
    </virtualHost>
    
    <virtualHost id="admin_host">
        <hostAlias>admin.company.com:9080</hostAlias>
        <hostAlias>admin.company.com:9443</hostAlias>
    </virtualHost>
    
    <!-- 애플리케이션 매핑 -->
    <webApplication location="api.war" 
                    contextRoot="/api"
                    virtualHostRef="api_host"/>
    
    <webApplication location="admin.war" 
                    contextRoot="/admin"
                    virtualHostRef="admin_host"/>
</server>
```

**문제점:**
- **복잡한 XML 편집**: 모든 설정을 수동으로 작성
- **오류 발생 쉬움**: 호스트 별칭 구문 오류 빈번
- **중앙 관리 불가**: 여러 서버의 가상호스트 개별 설정
- **가시성 부족**: 전체 구조 파악 어려움
- **변경 추적 어려움**: 설정 이력 관리 수동

### LibriX의 가상호스트 방식

**GUI 기반 설정:**
- 폼 기반 입력으로 간편한 설정
- 호스트 별칭 자동 검증
- 시각적 매핑 관리

**자동 구성:**
- server.xml 자동 생성
- 구문 오류 방지
- 설정 충돌 자동 감지

**중앙 관리:**
- 모든 가상호스트를 한 화면에서 관리
- 클러스터 단위 일괄 적용
- 통합 모니터링

## 가상호스트의 이점

### 멀티 도메인 서비스

**단일 서버에서 여러 사이트:**
```
www.company.com → 기업 홈페이지
api.company.com → REST API 서버
admin.company.com → 관리자 콘솔
shop.company.com → 쇼핑몰
```

**리소스 효율화:**
- 물리적 서버 절약
- 포트 공유 (동일한 80/443 포트)
- 통합 관리

### 애플리케이션 격리

**도메인별 독립성:**
- 각 도메인에 다른 애플리케이션
- Context Root 충돌 방지
- 보안 경계 설정

### 유연한 라우팅

**호스트 기반 라우팅:**
- DNS를 통한 트래픽 분산
- 서비스별 독립적 스케일링
- A/B 테스팅 용이

## 가상호스트 관리 화면

가상호스트 관리 화면에 접근:

```
환경 → 가상호스트
```

### 가상호스트 목록

**표시 정보:**
- **가상호스트 이름**: 고유 식별자
- **호스트 별칭**: 처리하는 도메인 목록
- **매핑된 애플리케이션**: 연결된 애플리케이션 수
- **설명**: 가상호스트 용도

## 가상호스트 생성

### 생성 절차

**1단계: 기본 정보**

- **가상호스트 이름**: 영문, 숫자, 언더스코어, 하이픈
- **설명**: 가상호스트 용도 (선택)

**2단계: 호스트 별칭 추가**

호스트 별칭은 `hostname:port` 형식으로 지정합니다.

#### 호스트 별칭 형식

**구체적인 호스트:**
```
api.company.com:9080
admin.company.com:9443
www.example.org:80
```

**와일드카드 사용:**
```
*:9080                    # 모든 호스트, 9080 포트
*.company.com:9080        # company.com의 모든 서브도메인
api.company.com:*         # 모든 포트 (비권장)
*:*                       # 모든 호스트, 모든 포트 (default_host)
```

**여러 별칭 추가:**
```
api.company.com:80
api.company.com:443
api.example.com:80
api.example.com:443
```

#### 호스트 별칭 우선순위

Liberty는 다음 순서로 가상호스트를 매칭:

1. **정확한 호스트 및 포트**: `api.company.com:9080`
2. **정확한 호스트, 와일드카드 포트**: `api.company.com:*`
3. **와일드카드 호스트, 정확한 포트**: `*.company.com:9080`
4. **완전 와일드카드**: `*:*`

**예제:**
```
요청: http://api.company.com:9080

매칭 순서:
1. api.company.com:9080 (있으면 선택)
2. api.company.com:* (있으면 선택)
3. *.company.com:9080 (있으면 선택)
4. *:9080 (있으면 선택)
5. *:* (default_host)
```

**3단계: 생성**

## 애플리케이션을 가상호스트에 매핑

### 매핑 방법

#### 방법 1: 애플리케이션 설치 시 지정

애플리케이션 설치 마법사에서:

```
고급 설정 → 가상호스트 → 선택
```

#### 방법 2: 기존 애플리케이션 수정

```
엔터프라이즈 애플리케이션 → 선택 → 편집
→ 가상호스트 → 변경
```

#### 방법 3: LibriX 관리 콘솔

```
가상호스트 → 선택 → "애플리케이션 매핑"
→ 애플리케이션 선택 → 확인
```

### server.xml 자동 생성

LibriX가 자동으로 생성하는 설정:

```xml
<server>
    <!-- 가상호스트 정의 -->
    <virtualHost id="api_host">
        <hostAlias>api.company.com:9080</hostAlias>
        <hostAlias>api.company.com:9443</hostAlias>
    </virtualHost>
    
    <!-- 애플리케이션 매핑 -->
    <webApplication location="api.war"
                    contextRoot="/api"
                    virtualHostRef="api_host"/>
</server>
```

## 기본 가상호스트 (default_host)

Liberty는 기본적으로 `default_host`를 제공합니다:

```xml
<virtualHost id="default_host">
    <hostAlias>*:9080</hostAlias>
    <hostAlias>*:9443</hostAlias>
</virtualHost>
```

### 특징

**자동 생성:**
- 명시적 정의 없이 자동 존재
- httpEndpoint의 포트 자동 포함

**기본 매핑:**
- 가상호스트를 지정하지 않은 애플리케이션
- 모든 호스트 및 포트 요청 처리

### default_host 커스터마이징

LibriX에서 default_host 수정:

```
가상호스트 → default_host → 편집
```

**변경 예:**
```
기존: *:9080, *:9443
변경: localhost:9080, localhost:9443

효과: 외부 호스트 이름 요청 차단
```

## 가상호스트 수정

### 호스트 별칭 추가/제거

**별칭 추가:**
1. 가상호스트 선택
2. "편집" 버튼
3. "별칭 추가"
4. 호스트:포트 입력
5. 저장

**별칭 제거:**
1. 제거할 별칭 선택
2. "제거" 버튼
3. 확인

**주의사항:**
- 별칭 제거 시 해당 도메인 접근 불가
- 트래픽이 있는 경우 DNS 업데이트 후 제거 권장

### 애플리케이션 매핑 변경

**매핑 변경:**
- 애플리케이션을 다른 가상호스트로 이동
- 즉시 적용 (서버 재시작 불필요)

**매핑 해제:**
- default_host로 자동 이동

## 가상호스트 삭제

### 삭제 절차

1. **매핑 확인**
   - 연결된 애플리케이션 확인

2. **매핑 해제**
   - 모든 애플리케이션 연결 제거 또는
   - 다른 가상호스트로 이동

3. **삭제 실행**
   - "삭제" 버튼 클릭
   - 확인

### 주의사항

**default_host 삭제 불가:**
- default_host는 시스템 가상호스트
- 수정만 가능

**사용 중인 가상호스트:**
```
오류: 이 가상호스트는 다음 애플리케이션에서 사용 중:
- myapp1
- myapp2

먼저 애플리케이션의 매핑을 변경하세요.
```

## 실제 사용 사례

### 사례 1: 멀티테넌트 SaaS

**시나리오:** 각 고객사별 서브도메인 제공

```
customer1.saas.com → Customer1 테넌트
customer2.saas.com → Customer2 테넌트
customer3.saas.com → Customer3 테넌트
```

**구성:**
```xml
<virtualHost id="customer1_host">
    <hostAlias>customer1.saas.com:443</hostAlias>
</virtualHost>

<virtualHost id="customer2_host">
    <hostAlias>customer2.saas.com:443</hostAlias>
</virtualHost>

<webApplication location="saas-app.war"
                contextRoot="/"
                virtualHostRef="customer1_host"/>

<webApplication location="saas-app.war"
                contextRoot="/"
                virtualHostRef="customer2_host"/>
```

**장점:**
- 동일 애플리케이션, 도메인별 격리
- 테넌트별 독립적 설정 가능
- 브랜딩 차별화

### 사례 2: API vs 웹 분리

**시나리오:** API와 웹 UI 분리 배포

```
www.company.com → 웹 UI
api.company.com → REST API
```

**구성:**
```xml
<virtualHost id="web_host">
    <hostAlias>www.company.com:80</hostAlias>
    <hostAlias>www.company.com:443</hostAlias>
</virtualHost>

<virtualHost id="api_host">
    <hostAlias>api.company.com:80</hostAlias>
    <hostAlias>api.company.com:443</hostAlias>
</virtualHost>

<webApplication location="web-ui.war"
                contextRoot="/"
                virtualHostRef="web_host"/>

<webApplication location="rest-api.war"
                contextRoot="/api"
                virtualHostRef="api_host"/>
```

**장점:**
- 서비스별 독립적 배포
- API 버전 관리 용이
- CORS 정책 명확화

### 사례 3: 개발/스테이징/운영 분리

**시나리오:** 단일 서버에서 여러 환경 호스팅

```
dev.myapp.com → 개발 환경
staging.myapp.com → 스테이징 환경
www.myapp.com → 운영 환경
```

**구성:**
```xml
<virtualHost id="dev_host">
    <hostAlias>dev.myapp.com:9080</hostAlias>
</virtualHost>

<virtualHost id="staging_host">
    <hostAlias>staging.myapp.com:9080</hostAlias>
</virtualHost>

<virtualHost id="prod_host">
    <hostAlias>www.myapp.com:80</hostAlias>
    <hostAlias>www.myapp.com:443</hostAlias>
</virtualHost>

<webApplication location="myapp-dev.war"
                contextRoot="/"
                virtualHostRef="dev_host"/>

<webApplication location="myapp-staging.war"
                contextRoot="/"
                virtualHostRef="staging_host"/>

<webApplication location="myapp-prod.war"
                contextRoot="/"
                virtualHostRef="prod_host"/>
```

**장점:**
- 리소스 절약
- 환경 간 격리
- 배포 파이프라인 단순화

## Liberty와 LibriX 비교

| 항목 | Open Liberty | LibriX |
|------|-------------|--------|
| 가상호스트 정의 | XML 수동 편집 | GUI 기반 폼 |
| 호스트 별칭 추가 | XML 편집 | 입력 폼 |
| 별칭 검증 | 수동 (오류 시 서버 실패) | 자동 검증 |
| 애플리케이션 매핑 | XML 편집 | 드래그 앤 드롭 |
| 전체 구조 확인 | XML 파싱 필요 | 시각적 다이어그램 |
| 충돌 감지 | 없음 | 자동 경고 |

## 모범 사례

### 명명 규칙

**가상호스트 이름:**

```
DO:
- web_host
- api_host
- admin_host
- customer1_host

DON'T:
- host1
- vh1
- test
```

### 호스트 별칭 설계

**구체적으로 정의:**

```
DO:
- api.company.com:443
- www.company.com:80
- admin.company.com:443

DON'T:
- *:* (너무 광범위)
- *.com:80 (너무 광범위)
```

**포트 명시:**

```
DO:
- api.company.com:443
- api.company.com:80

DON'T:
- api.company.com:* (불필요한 포트 개방)
```

### 보안

**HTTPS 강제:**
- 민감한 서비스는 HTTPS만 허용
- HTTP는 HTTPS로 리다이렉트

```xml
<virtualHost id="secure_host">
    <hostAlias>secure.company.com:443</hostAlias>
    <!-- HTTP는 제외 -->
</virtualHost>
```

**관리자 콘솔 격리:**
```xml
<virtualHost id="admin_host">
    <hostAlias>admin.internal.company.com:443</hostAlias>
</virtualHost>
```

- 내부 도메인만 허용
- 외부 접근 차단

## 로드 밸런서와의 통합

### Apache HTTP Server 연동

**시나리오:** Apache가 프론트엔드, Liberty가 백엔드

```
[Apache HTTP Server]
    ↓
[mod_proxy 또는 mod_jk]
    ↓
[Liberty Server]
    ↓
[Virtual Hosts]
```

**Apache 설정:**
```apache
<VirtualHost *:80>
    ServerName www.company.com
    ProxyPass / http://liberty-server:9080/
    ProxyPassReverse / http://liberty-server:9080/
</VirtualHost>

<VirtualHost *:80>
    ServerName api.company.com
    ProxyPass / http://liberty-server:9080/
    ProxyPassReverse / http://liberty-server:9080/
</VirtualHost>
```

**Liberty 설정:**
```xml
<virtualHost id="web_host">
    <hostAlias>www.company.com:9080</hostAlias>
</virtualHost>

<virtualHost id="api_host">
    <hostAlias>api.company.com:9080</hostAlias>
</virtualHost>
```

### 클라우드 로드 밸런서

**AWS ALB, Azure Load Balancer 등:**
- Host 헤더 기반 라우팅 지원
- Liberty 가상호스트와 매핑

**예제:**
```
ALB 규칙:
- Host: www.company.com → Target Group 1 (Liberty)
- Host: api.company.com → Target Group 2 (Liberty)

Liberty:
- virtual_host: www.company.com:9080
- virtual_host: api.company.com:9080
```

## 문제 해결

### 가상호스트가 작동하지 않음

**증상:** 특정 도메인으로 접근 시 404 또는 다른 애플리케이션 표시

**원인 및 해결:**

1. **호스트 별칭 오타**
   - 별칭 철자 확인
   - 포트 번호 확인

2. **DNS 설정**
   - nslookup으로 DNS 확인
   - 호스트 파일(/etc/hosts) 확인

3. **애플리케이션 매핑 누락**
   - 가상호스트에 애플리케이션 연결 확인

4. **우선순위 문제**
   - 더 일반적인 별칭이 먼저 매칭
   - 구체적인 별칭 추가

### 가상호스트 충돌

**증상:** 여러 가상호스트가 동일한 요청 처리

**해결:**
```
충돌 예:
virtualHost1: api.company.com:9080
virtualHost2: *.company.com:9080

→ api.company.com:9080 요청은 virtualHost1이 우선

해결: 더 구체적인 별칭 사용
```

### Host 헤더 불일치

**증상:** 로드 밸런서 뒤에서 가상호스트 작동 안 함

**원인:**
- 로드 밸런서가 Host 헤더 변경
- Liberty에서 다른 호스트 이름 수신

**해결:**
```xml
<!-- 로드 밸런서의 실제 주소 추가 -->
<virtualHost id="web_host">
    <hostAlias>www.company.com:80</hostAlias>
    <hostAlias>lb-internal.company.com:9080</hostAlias>
</virtualHost>
```

## 성능 고려사항

### 가상호스트 개수

**권장사항:**
- 서버당 10~20개 이하 권장
- 너무 많은 가상호스트는 성능 영향

**대규모 멀티테넌트:**
- 와일드카드 사용
- 애플리케이션 레벨에서 테넌트 구분

### 별칭 검색 최적화

**구체적 별칭 우선:**
- 와일드카드 최소화
- 정확한 호스트:포트 명시

```
느림:
*:*, *.com:9080, *:9080

빠름:
api.company.com:9080
www.company.com:9080
```

## 모니터링

### 가상호스트별 트래픽

LibriX는 가상호스트별 통계 제공:

**메트릭:**
- 요청 수
- 응답 시간
- 오류율
- 활성 연결

**알림:**
- 특정 가상호스트 오류 증가
- 응답 시간 지연
- 트래픽 급증

## 관련 문서

- [Liberty 변수 관리](liberty-variables.md)
- [세션 도메인 관리](session-domain.md)
- [공유 라이브러리 관리](shared-library.md)
- [애플리케이션 설치](../application/application-install.md)
- [웹 서버 관리](../server/web-server.md)
