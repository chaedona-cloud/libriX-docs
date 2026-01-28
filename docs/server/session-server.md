# 세션 서버 관리

## 개요

세션 서버는 HTTP 세션 데이터를 저장하고 관리하는 전용 서버입니다. 세션 도메인의 멤버 서버로서 세션 복제 및 공유를 담당하며, LibriX를 통해 중앙에서 관리할 수 있습니다.

## 세션 서버란?

세션 서버는 세션 도메인에 속한 Liberty 서버로, 애플리케이션 서버들이 생성한 HTTP 세션 데이터를 공유하고 복제하는 역할을 합니다. 이를 통해 서버 장애 시에도 사용자 세션이 유지되어 끊김 없는 서비스를 제공할 수 있습니다.

## 세션 서버의 역할

### 세션 데이터 저장

**Database 방식:**
- 관계형 데이터베이스에 세션 저장
- 영구적 보관
- 서버 재시작 후에도 유지

**Memory 방식:**
- 서버 메모리에 세션 저장
- 빠른 성능
- 서버 재시작 시 손실

### 세션 복제

**동기화:**
- 여러 서버 간 세션 데이터 공유
- 실시간 복제
- 자동 페일오버

## Liberty의 전통적인 세션 관리

### 단일 서버 환경

**문제점:**
```
[User] → [Server1]
         └── 세션 저장

Server1 장애 시:
→ 세션 손실
→ 사용자 재로그인 필요
→ 진행 중 작업 손실
```

### 수동 세션 복제 설정

**server.xml 직접 편집:**
```xml
<server>
    <featureManager>
        <feature>servlet-5.0</feature>
        <feature>sessionDatabase-1.0</feature>
    </featureManager>
    
    <dataSource id="SessionDS" jndiName="jdbc/sessions">
        <jdbcDriver libraryRef="PostgreSQL"/>
        <properties.postgresql
            serverName="localhost"
            portNumber="5432"
            databaseName="sessions"/>
    </dataSource>
    
    <httpSessionDatabase dataSourceRef="SessionDS"/>
</server>
```

**문제점:**
- 각 서버마다 동일 설정 반복
- 설정 오류 발생 쉬움
- 동기화 관리 어려움
- 중앙 모니터링 불가

### LibriX의 세션 서버 방식

**GUI 기반 관리:**
- 세션 도메인 생성으로 일괄 구성
- 드래그 앤 드롭으로 서버 추가
- 자동 설정 동기화

**자동 구성:**
- server.xml 자동 생성
- 데이터베이스 테이블 자동 생성
- 세션 복제 자동 활성화

**통합 모니터링:**
- 전체 세션 통계
- 서버별 세션 분포
- 복제 상태 실시간 확인

## 세션 서버 관리 화면

세션 서버 관리 화면에 접근:

```
서버 → 세션 서버
```

또는

```
환경 → 세션 도메인 → (도메인 선택) → 멤버 서버
```

### 세션 서버 목록

**표시 정보:**
- **서버 이름**: 세션 서버 식별자
- **호스트**: 서버가 실행 중인 호스트
- **세션 도메인**: 속한 도메인 이름
- **상태**: 실행 중/중지
- **세션 수**: 현재 저장된 세션 개수
- **메모리 사용**: 세션 데이터 메모리 사용량

## 세션 서버 생성

### 방법 1: 세션 도메인을 통한 생성

세션 도메인 생성 시 멤버 서버가 자동으로 세션 서버가 됩니다:

```
환경 → 세션 도메인 → 새로 만들기
→ 멤버 서버 선택
→ 자동으로 세션 서버 구성
```

### 방법 2: 기존 서버를 세션 서버로 추가

```
1. 세션 도메인 선택
2. "멤버 추가" 버튼 클릭
3. 기존 Liberty 서버 선택
4. 확인
→ 해당 서버가 세션 서버로 구성됨
```

## 세션 서버 설정

### Database 세션 서버

**데이터소스 설정:**
```
데이터소스: jdbc/SessionDS
드라이버: PostgreSQL
호스트: session-db.company.com
포트: 5432
데이터베이스: sessions
사용자: session_user
비밀번호: ********
```

**세션 테이블:**
LibriX가 자동 생성:
```sql
CREATE TABLE sessions (
    id VARCHAR(128) PRIMARY KEY,
    app_name VARCHAR(128),
    user_name VARCHAR(128),
    small VARCHAR(3000),
    medium BYTEA,
    large BYTEA,
    creation_time BIGINT,
    access_time BIGINT,
    max_inactive BIGINT,
    properties VARCHAR(3000)
);
```

**쓰기 빈도:**
```
○ END_OF_SERVLET_SERVICE (기본, 가장 안전)
○ MANUAL_UPDATE (명시적 호출 시)
○ TIME_BASED_WRITE (주기적)
```

### Memory-to-Memory 세션 서버

**네트워크 설정:**

**멀티캐스트:**
```
주소: 239.255.0.1
포트: 7777
TTL: 1
```

**유니캐스트 (방화벽 환경):**
```
Server1: 192.168.1.10:7777
Server2: 192.168.1.11:7777
Server3: 192.168.1.12:7777
```

**복제 모드:**
```
○ BOTH: 세션 송수신 (기본)
○ CLIENT: 세션 송신만
○ SERVER: 세션 수신만
```

## 세션 서버 작업

### 시작/중지

**개별 서버:**
```
서버 선택 → 시작/중지 버튼
```

**도메인 전체:**
```
세션 도메인 선택 → 모든 멤버 시작/중지
```

### 세션 데이터 조회

**세션 통계:**
- 총 세션 수
- 활성 세션 수
- 만료 세션 수
- 평균 세션 크기

**개별 세션 정보:**
```
세션 ID: ABC123...
생성 시간: 2026-01-28 10:00:00
마지막 접근: 2026-01-28 10:15:30
소유 서버: server1
세션 크기: 2.5 KB
사용자: user@company.com
```

### 세션 무효화

**개별 세션:**
```
세션 선택 → 무효화 버튼
→ 해당 사용자 재로그인 필요
```

**전체 세션:**
```
서버 또는 도메인 선택 → 모든 세션 무효화
→ 긴급 보안 조치 시 사용
```

**사용 사례:**
- 보안 위협 대응
- 애플리케이션 업그레이드 후
- 메모리 정리

## 세션 복제 모니터링

### 복제 상태

LibriX는 실시간 복제 상태 제공:

**복제 메트릭:**
- 복제 성공률: 99.5%
- 평균 복제 시간: 15ms
- 복제 대기 큐: 0건
- 실패 건수: 2건

**서버 간 복제 흐름:**
```
Server1 → Session Created
  ↓
Database (또는 다른 서버 메모리)
  ↓
Server2, Server3 → Session Replicated
```

### 알림 설정

**복제 실패 알림:**
```
조건: 복제 실패율 > 5%
알림: 이메일, Slack
```

**세션 동기화 지연:**
```
조건: 복제 시간 > 1초
알림: 관리자 대시보드
```

## 세션 페일오버

### 자동 페일오버 동작

**시나리오:**
```
1. User → Server1 (세션 저장)
2. Server1 장애 발생
3. User → Server2 (자동 라우팅)
4. Server2가 Database에서 세션 로드
5. 사용자는 끊김 없이 계속 서비스 이용
```

**요구사항:**
- 세션 스티키 쿠키 설정
- 로드 밸런서 헬스 체크
- 세션 복제 활성화

### 페일오버 테스트

**테스트 절차:**
```
1. Server1에 로그인
2. Server1 중지
3. 새로고침 (Server2로 라우팅)
4. 세션 유지 확인
```

## Database vs Memory 비교

| 항목 | Database 복제 | Memory-to-Memory |
|------|--------------|-----------------|
| 성능 | 중간 | 빠름 |
| 영속성 | 영구 보관 | 휘발성 |
| 확장성 | 높음 | 제한적 |
| 네트워크 오버헤드 | 중간 | 높음 |
| 설정 복잡도 | 낮음 | 높음 |
| 권장 환경 | 운영 환경 | 개발/테스트 |

## Liberty와 LibriX 비교

| 항목 | Open Liberty | LibriX |
|------|-------------|--------|
| 세션 서버 개념 | 없음 (수동 구성) | 명시적 관리 |
| 설정 방법 | XML 편집 | GUI 기반 |
| 자동 구성 | 불가 | 가능 |
| 테이블 생성 | 수동 DDL | 자동 생성 |
| 복제 모니터링 | 별도 도구 | 통합 대시보드 |
| 페일오버 테스트 | 수동 | 자동 검증 |

## 모범 사례

### 세션 서버 배치

**DO:**
- 최소 2대 이상 (HA)
- 동일 네트워크 세그먼트
- 충분한 메모리 할당
- 별도 데이터베이스 사용 (Database 방식)

**DON'T:**
- 단일 서버
- 원격 데이터센터 혼합
- 애플리케이션 서버와 리소스 공유

### 세션 크기 최적화

**DO:**
- 필수 데이터만 세션에 저장
- 대용량 데이터는 DB 저장 후 ID만 세션 보관
- 직렬화 가능한 객체 사용

**DON'T:**
- 파일 스트림 저장
- 네트워크 연결 저장
- 전체 결과 셋 저장

**권장 세션 크기:**
```
최소: < 1KB
권장: 1~10KB
최대: < 50KB
```

### 세션 타임아웃

**설정 예:**
```
일반 웹사이트: 30분
쇼핑몰: 60분
금융 서비스: 10분
관리자 콘솔: 15분
```

## 성능 튜닝

### Database 세션 최적화

**연결 풀 설정:**
```
최소 연결: 서버 수 × 2
최대 연결: 서버 수 × 10
```

**인덱스 최적화:**
```sql
CREATE INDEX idx_access_time ON sessions(access_time);
CREATE INDEX idx_app_name ON sessions(app_name);
```

**만료 세션 정리:**
```sql
-- Cron 작업 (일일 실행)
DELETE FROM sessions 
WHERE access_time < (EXTRACT(EPOCH FROM NOW()) * 1000) - 86400000;
```

### Memory 세션 최적화

**JVM 힙 크기:**
```bash
# 세션 데이터 고려
-Xms4g -Xmx4g
```

**네트워크 버퍼:**
```xml
<channelfw>
    <tcpOptions maxSocketBufferSize="65536"/>
</channelfw>
```

## 문제 해결

### 세션 복제가 작동하지 않음

**증상:** 서버 장애 시 세션 손실

**원인 및 해결:**

1. **Feature 미설치**
   ```xml
   <feature>sessionDatabase-1.0</feature>
   또는
   <feature>sessionCache-1.0</feature>
   ```

2. **데이터소스 오류**
   - 연결 테스트
   - 권한 확인

3. **네트워크 문제** (Memory 방식)
   - 멀티캐스트 지원 확인
   - 방화벽 포트 개방

### 세션 동기화 지연

**증상:** 세션 변경 반영 느림

**원인 및 해결:**

1. **데이터베이스 성능**
   - 연결 풀 크기 증가
   - 쿼리 최적화

2. **네트워크 지연**
   - 서버 위치 확인
   - 네트워크 대역폭 확인

3. **쓰기 빈도 설정**
   ```xml
   <httpSessionDatabase 
       writeFrequency="END_OF_SERVLET_SERVICE"/>
   ```

### 메모리 부족

**증상:** OutOfMemoryError

**원인 및 해결:**

1. **세션 크기 증가**
   - 세션 데이터 최적화
   - 타임아웃 감소

2. **세션 수 증가**
   - JVM 힙 크기 증가
   - 서버 추가

3. **메모리 누수**
   - 힙 덤프 분석
   - 세션 정리 로직 확인

## 보안 고려사항

### 세션 하이재킹 방어

**보안 설정:**
```xml
<httpSession 
    cookieSecure="true"
    cookieHttpOnly="true"
    cookieSameSite="Strict"/>
```

**세션 ID 재생성:**
- 로그인 후
- 권한 변경 시
- 주기적 재생성

### 데이터 암호화

**Database 방식:**
- DB 컬럼 암호화
- 전송 구간 TLS

**Memory 방식:**
- 네트워크 암호화
- VPN 사용

## 관련 문서

- [세션 도메인 관리](../environment/session-domain.md)
- [클러스터 관리](cluster.md)
- [데이터소스 관리](../resource/datasource.md)
- [애플리케이션 서버 관리](application-server.md)
