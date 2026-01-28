# 세션 도메인 관리

## 개요

세션 도메인은 여러 Liberty 서버 간에 HTTP 세션 데이터를 공유하기 위한 서버 그룹입니다. LibriX의 세션 도메인 관리 기능을 통해 세션 복제를 구성하고, 고가용성(High Availability) 환경을 구축할 수 있습니다.

## 세션 도메인이란?

세션 도메인은 동일한 세션 데이터를 공유하는 Liberty 서버들의 논리적 그룹입니다. 한 서버에서 생성된 세션을 다른 서버에서도 사용할 수 있어, 서버 장애 시에도 사용자 세션이 유지됩니다.

### Liberty의 전통적인 세션 복제 방식

Open Liberty에서 세션 복제를 설정하는 전통적인 방법:

#### 1. server.xml 수동 편집

각 서버의 server.xml에 세션 복제 설정을 추가:

```xml
<server>
    <!-- sessionDatabase Feature 활성화 -->
    <featureManager>
        <feature>servlet-5.0</feature>
        <feature>sessionDatabase-1.0</feature>
    </featureManager>
    
    <!-- 데이터소스 정의 -->
    <dataSource id="SessionDS" jndiName="jdbc/sessions">
        <jdbcDriver libraryRef="PostgreSQL"/>
        <properties.postgresql
            serverName="localhost"
            portNumber="5432"
            databaseName="sessions"/>
    </dataSource>
    
    <!-- HTTP 세션 관리 -->
    <httpSessionDatabase id="SessionDB" 
                         dataSourceRef="SessionDS"/>
    
    <!-- 애플리케이션에 세션 복제 적용 -->
    <webApplication location="myapp.war">
        <session-config>
            <session-properties>
                <property name="SessionDB" value="true"/>
            </session-properties>
        </session-config>
    </webApplication>
</server>
```

**문제점:**
- **복잡한 XML 설정**: 각 서버마다 동일한 설정 반복
- **수동 동기화**: 서버 간 설정 일관성 유지 어려움
- **오류 발생 쉬움**: 구성 오류 시 세션 복제 실패
- **중앙 관리 불가**: 여러 서버 일괄 설정 어려움

#### 2. JCache 사용 (sessionCache)

JCache를 이용한 세션 캐싱:

```xml
<featureManager>
    <feature>sessionCache-1.0</feature>
</featureManager>

<httpSessionCache libraryRef="InfinispanLib">
    <properties infinispan.client.hotrod.server_list="cache1:11222"/>
</httpSessionCache>
```

**문제점:**
- 외부 캐시 서버 (Infinispan, Hazelcast 등) 별도 구축 필요
- 복잡한 캐시 서버 설정 및 관리
- 추가 인프라 비용

### LibriX의 세션 도메인 방식

LibriX는 WebSphere Application Server 스타일의 세션 도메인을 제공합니다:

**GUI 기반 설정**
- 드래그 앤 드롭으로 서버 추가
- 시각적 토폴로지 뷰
- 실시간 세션 복제 상태 모니터링

**자동 구성 생성**
- server.xml 자동 업데이트
- 데이터소스 자동 생성
- 세션 테이블 자동 생성

**중앙 집중 관리**
- 모든 세션 도메인을 한 화면에서 관리
- 일괄 설정 변경
- 통합 모니터링

## 세션 도메인의 이점

### 고가용성 (High Availability)

**서버 장애 대응:**
- Server A 장애 시 → Server B가 세션 인계
- 사용자는 끊김 없이 서비스 이용
- 자동 페일오버

**로드 밸런싱:**
- 여러 서버에 트래픽 분산
- 세션 스티키 불필요
- 유연한 트래픽 관리

### 확장성 (Scalability)

**수평 확장:**
- 서버 추가로 처리 용량 증가
- 세션 데이터 자동 공유
- 동적 서버 추가/제거

### 사용자 경험 개선

**세션 유지:**
- 서버 재시작 중에도 세션 유지
- 로그인 상태 보존
- 장바구니, 작업 상태 유지

## 세션 도메인 관리 화면

세션 도메인 관리 화면에 접근하려면:

```
환경 → 세션 도메인
```

### 세션 도메인 목록

세션 도메인 목록 화면에서는 다음 정보를 확인할 수 있습니다:

**표시 항목:**
- **도메인 이름**: 세션 도메인의 고유 이름
- **멤버 서버**: 도메인에 속한 서버 수
- **복제 방식**: Database 또는 Memory-to-Memory
- **상태**: 활성/비활성
- **세션 수**: 현재 저장된 세션 개수

## 세션 도메인 생성

### 생성 절차

**1단계: 기본 정보 입력**

- **도메인 이름**: 세션 도메인의 고유 식별자
- **설명**: 도메인 용도 설명 (선택)

**2단계: 복제 방식 선택**

LibriX는 두 가지 세션 복제 방식을 지원합니다:

#### Database 복제 (권장)

**특징:**
- 관계형 데이터베이스에 세션 저장
- 영구적인 세션 보관
- 서버 재시작 후에도 세션 유지
- 대규모 환경에 적합

**지원 데이터베이스:**
- PostgreSQL
- MySQL/MariaDB
- Oracle Database
- IBM DB2
- Microsoft SQL Server

**설정 항목:**
- 데이터소스 선택
- 세션 테이블 이름
- 세션 타임아웃
- 쓰기 빈도 (Write Frequency)

#### Memory-to-Memory 복제

**특징:**
- 서버 메모리 간 직접 복제
- 빠른 성능
- 데이터베이스 불필요
- 소규모 환경에 적합

**제약사항:**
- 모든 서버 재시작 시 세션 손실
- 네트워크 오버헤드
- 메모리 사용량 증가

**3단계: 멤버 서버 선택**

세션을 공유할 서버들을 선택합니다:

- 개별 서버 선택
- 클러스터 전체 선택
- 드래그 앤 드롭으로 추가/제거

**4단계: 세션 설정**

```
세션 타임아웃: 30분 (기본값)
세션 쿠키 이름: JSESSIONID
쿠키 경로: /
쿠키 도메인: (비워두면 현재 도메인)
보안 쿠키: ☑ (HTTPS에서만 전송)
HttpOnly 쿠키: ☑ (JavaScript 접근 차단)
```

**5단계: 확인 및 생성**

- 설정 요약 확인
- "생성" 버튼 클릭
- 자동으로 server.xml 업데이트

## Database 세션 복제 상세

### 데이터베이스 설정

**1. 데이터소스 선택**

기존 데이터소스 선택 또는 새로 생성:

```
데이터소스: jdbc/SessionDS
드라이버: PostgreSQL
호스트: session-db.company.com
포트: 5432
데이터베이스: sessions
```

**2. 세션 테이블 자동 생성**

LibriX가 자동으로 세션 테이블을 생성합니다:

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

CREATE INDEX idx_sessions_app ON sessions(app_name);
CREATE INDEX idx_sessions_access ON sessions(access_time);
```

**3. 세션 저장 방식**

**쓰기 빈도 옵션:**

- **END_OF_SERVLET_SERVICE**: 각 요청 완료 시 저장 (기본값, 가장 안전)
- **MANUAL_UPDATE**: 명시적 호출 시에만 저장
- **TIME_BASED_WRITE**: 일정 시간마다 저장

**예제 설정:**
```xml
<httpSessionDatabase writeFrequency="END_OF_SERVLET_SERVICE"
                     writeInterval="120"/>
```

### 세션 데이터 구조

**세션 크기에 따른 컬럼 사용:**
- **small** (최대 3KB): 작은 세션 데이터
- **medium** (최대 2MB): 중간 크기 세션 데이터  
- **large**: 대용량 세션 데이터

LibriX가 자동으로 적절한 컬럼 선택

## Memory-to-Memory 복제 상세

### 복제 토폴로지

**Peer-to-Peer 방식:**
- 모든 서버가 동등한 역할
- 세션 데이터를 서로 복제
- 특정 서버에 의존하지 않음

**복제 모드:**

1. **CLIENT**: 세션을 다른 서버에 복제만 하고 받지 않음
2. **SERVER**: 세션을 받기만 하고 복제하지 않음
3. **BOTH**: 세션을 복제하고 받음 (기본값)

### 네트워크 설정

**멀티캐스트 설정:**
```
멀티캐스트 주소: 239.255.0.1
포트: 7777
TTL: 1
```

**유니캐스트 설정 (방화벽 환경):**
```
Server1: host1.company.com:7777
Server2: host2.company.com:7777
Server3: host3.company.com:7777
```

## 세션 도메인 수정

### 멤버 서버 추가/제거

**서버 추가:**
1. 세션 도메인 선택
2. "서버 추가" 버튼 클릭
3. 추가할 서버 선택
4. 저장

**서버 제거:**
1. 제거할 서버 선택
2. "제거" 버튼 클릭
3. 확인

**주의사항:**
- 서버 제거 시 해당 서버의 세션은 다른 서버로 이동
- Database 복제: 세션 데이터 보존
- Memory 복제: 세션 손실 가능

### 설정 변경

**변경 가능한 설정:**
- 세션 타임아웃
- 쿠키 설정
- 쓰기 빈도
- 복제 모드

**변경 절차:**
1. 세션 도메인 선택
2. "편집" 버튼 클릭
3. 설정 수정
4. 저장

**적용 방식:**
- 대부분 즉시 적용
- 일부 설정은 서버 재시작 필요

## 세션 모니터링

### 세션 통계

LibriX는 실시간 세션 통계를 제공합니다:

**전체 통계:**
- 총 활성 세션 수
- 서버별 세션 분포
- 평균 세션 크기
- 세션 생성/소멸 속도

**개별 세션 정보:**
- 세션 ID
- 생성 시간
- 마지막 접근 시간
- 소유 서버
- 세션 크기
- 사용자 정보

### 세션 복제 상태

**복제 상태 모니터링:**
- 복제 성공률
- 복제 지연 시간
- 실패한 복제 건수
- 네트워크 트래픽

**알림 설정:**
- 복제 실패율 임계값 초과
- 세션 동기화 지연
- 데이터베이스 연결 오류

## Liberty와 LibriX 비교

| 항목 | Open Liberty | LibriX |
|------|-------------|--------|
| 설정 방법 | XML 수동 편집 | GUI 기반 설정 |
| 세션 도메인 개념 | 없음 (개별 설정) | 중앙 집중 관리 |
| 서버 추가 | 각 서버 개별 설정 | 드래그 앤 드롭 |
| 테이블 생성 | 수동 DDL 실행 | 자동 생성 |
| 모니터링 | 별도 도구 필요 | 통합 대시보드 |
| 문제 진단 | 로그 수동 분석 | 시각적 상태 표시 |

## 모범 사례

### 복제 방식 선택

**Database 복제 사용 시기:**
- 운영 환경
- 세션 데이터 영구 보관 필요
- 서버 재시작 빈번
- 대규모 사용자

**Memory 복제 사용 시기:**
- 개발/테스트 환경
- 높은 성능 필요
- 소규모 세션 데이터
- 임시 세션 사용

### 세션 크기 최적화

**DO:**
- 세션에 필수 데이터만 저장
- 대용량 데이터는 데이터베이스 저장 후 ID만 세션에 보관
- 직렬화 가능한 객체 사용
- 세션 타임아웃 적절히 설정

**DON'T:**
- 파일 스트림이나 네트워크 연결 저장
- 직렬화 불가능한 객체 저장
- 불필요한 대용량 데이터 저장
- 세션에 전체 결과 셋 저장

### 데이터베이스 성능

**인덱스 최적화:**
- 접근 시간(access_time) 인덱스 필수
- 애플리케이션별 인덱스 고려
- 정기적인 인덱스 재구성

**정리 작업:**
```sql
-- 만료된 세션 정리 (일일 실행)
DELETE FROM sessions 
WHERE access_time < (EXTRACT(EPOCH FROM NOW()) * 1000) - 86400000;
```

**연결 풀 크기:**
- 최소 연결: 서버 수 × 2
- 최대 연결: 서버 수 × 10
- 대기 시간: 30초

## 문제 해결

### 세션 복제가 작동하지 않음

**증상:** 서버 간 세션 공유 안 됨

**원인 및 해결:**

1. **Feature 미설치**
   - sessionDatabase-1.0 또는 sessionCache-1.0 확인
   - LibriX에서 자동 설치 확인

2. **데이터베이스 연결 실패**
   - 데이터소스 연결 테스트
   - 방화벽 및 네트워크 확인
   - 데이터베이스 권한 확인

3. **서버 간 네트워크 문제** (Memory 복제)
   - 멀티캐스트 지원 확인
   - 방화벽 포트 개방
   - TTL 설정 확인

### 세션 데이터 손실

**원인 및 해결:**

1. **조기 타임아웃**
   - 세션 타임아웃 설정 확인
   - 로드 밸런서 타임아웃 확인

2. **쓰기 빈도 설정**
   - END_OF_SERVLET_SERVICE 사용 권장
   - writeInterval 감소

3. **데이터베이스 용량**
   - 디스크 공간 확인
   - 오래된 세션 정리

### 성능 저하

**원인 및 해결:**

1. **대용량 세션**
   - 세션 크기 줄이기
   - 세션 데이터 구조 최적화

2. **데이터베이스 병목**
   - 연결 풀 크기 증가
   - 인덱스 최적화
   - 데이터베이스 성능 튜닝

3. **네트워크 지연** (Memory 복제)
   - 네트워크 대역폭 확인
   - 복제 모드 조정
   - 서버 위치 최적화

## 보안 고려사항

### 세션 하이재킹 방지

**보안 쿠키 설정:**
```
Secure: ☑ (HTTPS만)
HttpOnly: ☑ (XSS 방지)
SameSite: Strict
```

**세션 ID 재생성:**
- 로그인 후 세션 ID 변경
- 권한 변경 시 세션 ID 재생성

### 민감한 데이터 보호

**DO:**
- 세션에 민감 정보 최소화
- 신용카드 정보 세션 저장 금지
- 암호화 통신 (HTTPS) 필수

**DON'T:**
- 비밀번호 세션 저장
- 개인 식별 정보 과다 저장

## 관련 문서

- [Liberty 변수 관리](liberty-variables.md)
- [공유 라이브러리 관리](shared-library.md)
- [가상호스트 관리](virtual-host.md)
- [클러스터 관리](../server/cluster.md)
- [데이터소스 관리](../resource/datasource.md)
