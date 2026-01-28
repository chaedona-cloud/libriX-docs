# 애플리케이션 서버 관리

## 개요

애플리케이션 서버 메뉴는 LibriX 관리콘솔에 등록된 Open Liberty 서버들을 중앙에서 관리할 수 있는 기능을 제공합니다. 이 메뉴에서는 여러 대의 Liberty 서버를 등록하고, 시작, 중지, 재시작 등의 생명주기 관리 작업을 수행할 수 있습니다.

## 애플리케이션 서버 목록

애플리케이션 서버 메뉴를 선택하면 현재 Liberty에 등록된 모든 애플리케이션 서버의 목록이 표시됩니다.

**표시 정보:**
- **서버 이름**: Liberty 서버의 고유 식별자
- **호스트 이름**: 서버가 실행 중인 물리적/가상 호스트
- **상태**: 실행 중(녹색), 중지(회색), 알 수 없음(회색)
- **포트**: HTTP/HTTPS 포트 번호
- **Liberty 버전**: 설치된 Liberty 버전
- **시작 시간**: 서버가 시작된 시각
- **애플리케이션 수**: 배포된 애플리케이션 개수

## Liberty의 전통적인 서버 관리 방식

### 명령줄 도구

Open Liberty에서 서버를 관리하는 전통적인 방법:

```bash
# 서버 생성
server create myServer

# 서버 시작
server start myServer

# 서버 상태 확인
server status myServer

# 서버 중지
server stop myServer
```

**문제점:**
- 각 서버에 직접 접속 필요
- 여러 서버 관리 시 반복 작업
- 중앙 집중식 관리 불가
- 실시간 모니터링 어려움
- GUI 인터페이스 없음

### WebSphere Liberty Admin Center

**제한적 기능:**
- 단일 서버만 관리 가능
- 기본적인 시작/중지 기능
- 제한된 모니터링
- 고급 기능 부족

### LibriX의 서버 관리 방식

**중앙 집중 관리:**
- 모든 서버를 한 화면에서 관리
- 여러 서버 동시 제어
- 실시간 상태 모니터링

**GUI 기반 작업:**
- 웹 UI를 통한 직관적 관리
- 클릭 한 번으로 작업 수행
- 시각적 상태 표시

**자동화 기능:**
- 자동 재시작 스케줄
- 헬스 체크 자동화
- 알림 설정

## 새 애플리케이션 서버 추가

### 단계 1: 호스트 선택

새 서버를 추가하려면 먼저 서버가 실행될 호스트를 선택합니다.

**호스트 이름 또는 IP 주소:**
```
예: localhost.localdomain
예: server01.company.com
예: 192.168.1.100
```

### 단계 2: 새 서버 확인

다음은 선택(할) 호스트에서 사용 가능한 새 애플리케이션 서버 목록이 표시됩니다.

**옵션:**
- **기존 서버 선택**: 이미 설치된 Liberty 서버 등록
- **새 서버 생성**: LibriX에서 새로운 Liberty 서버 생성

### 단계 3: 서버 유형 선택

**서버 유형:**

#### 애플리케이션 서버
- 표준 Liberty 서버
- WAR/EAR 애플리케이션 실행
- 대부분의 경우 선택

#### 웹 서버
- Apache HTTP Server
- 정적 콘텐츠 제공
- 로드 밸런서 역할

### 단계 4: 서버 구성

**기본 정보:**
```
서버 이름: myAppServer
설명: 운영 환경 애플리케이션 서버
```

**네트워크 설정:**
```
HTTP 포트: 9080
HTTPS 포트: 9443
호스트: *
```

**JVM 설정:**
```
최소 힙 크기: 512m
최대 힙 크기: 2048m
```

### 단계 5: 서버 등록 완료

설정을 완료하면 LibriX가 자동으로:
- Liberty 서버 생성 (새 서버의 경우)
- server.xml 기본 구성 생성
- LibriX Agent 연결 설정
- 서버 목록에 추가

## 서버 작업

### 서버 시작

**단일 서버 시작:**
1. 서버 선택
2. "시작" 버튼 클릭
3. 시작 진행 상황 표시
4. 완료 후 상태가 "실행 중"으로 변경

**시작 옵션:**
```
○ 일반 시작
○ 디버그 모드 시작 (JDWP 포트 활성화)
```

**자동 생성되는 명령:**
```bash
# LibriX가 내부적으로 실행
server start myAppServer
```

### 서버 중지

**중지 방법:**

#### 정상 중지 (Graceful Stop)
```
현재 처리 중인 요청 완료 후 중지
- 안전한 방법
- 시간이 더 소요
- 권장 방법
```

#### 즉시 중지 (Force Stop)
```
즉시 서버 프로세스 종료
- 빠른 중지
- 데이터 손실 가능성
- 긴급 상황에만 사용
```

### 서버 재시작

**재시작 옵션:**

```
○ 정상 재시작: 중지 → 대기 → 시작
○ 빠른 재시작: 최소 대기 시간
```

**사용 시기:**
- 설정 변경 후
- 애플리케이션 업데이트 후
- 메모리 정리 필요 시

### 서버 삭제

**삭제 절차:**
1. 서버 선택
2. "삭제" 버튼 클릭
3. 확인 대화상자
4. 삭제 실행

**삭제 옵션:**
```
☐ 서버 디렉토리 삭제
☐ 로그 파일 삭제
☑ LibriX 목록에서만 제거
```

## 서버 상세 정보

### 기본 정보 탭

**서버 정보:**
- 서버 이름
- 호스트 이름
- Liberty 버전
- JDK 버전
- 설치 경로

**네트워크 정보:**
- HTTP 엔드포인트
- HTTPS 엔드포인트
- 관리 포트
- 바인드 주소

### 구성 탭

**server.xml 편집:**
- 웹 기반 XML 편집기
- 구문 강조 표시
- 자동 완성
- 유효성 검사

**주요 설정 항목:**
```xml
<server>
    <featureManager>
        <feature>servlet-5.0</feature>
        <feature>jdbc-4.3</feature>
        <feature>jndi-1.0</feature>
    </featureManager>
    
    <httpEndpoint id="defaultHttpEndpoint"
                  httpPort="9080"
                  httpsPort="9443"
                  host="*"/>
    
    <applicationManager autoExpand="true"/>
</server>
```

### 애플리케이션 탭

**배포된 애플리케이션 목록:**
- 애플리케이션 이름
- Context Root
- 상태 (시작/중지)
- 파일 크기
- 배포 시간

**빠른 작업:**
- 애플리케이션 시작/중지
- 애플리케이션 재시작
- 애플리케이션 업데이트
- 애플리케이션 제거

### 모니터링 탭

**실시간 메트릭:**
```
CPU 사용률: 45%
메모리 사용: 1.2 GB / 2.0 GB (60%)
힙 메모리: 800 MB / 1024 MB (78%)
활성 스레드: 25
요청 처리량: 150 req/sec
평균 응답 시간: 45ms
```

**그래프:**
- CPU 사용률 추이
- 메모리 사용량 추이
- 요청 처리량 추이
- 응답 시간 추이

### 로그 탭

**로그 파일 목록:**
- messages.log
- console.log
- trace.log
- ffdc/ (First Failure Data Capture)

**로그 뷰어 기능:**
- 실시간 로그 스트리밍
- 로그 레벨 필터링
- 키워드 검색
- 로그 다운로드

## Liberty Features 관리

### Feature Manager

**설치된 Features 확인:**
```
현재 설치된 Features:
- servlet-5.0
- jsp-2.3
- jdbc-4.3
- jndi-1.0
- cdi-3.0
```

**Feature 추가:**
1. "Feature 추가" 버튼
2. 카테고리에서 선택 또는 검색
3. 선택 후 적용
4. 자동으로 server.xml 업데이트

**Feature 카테고리:**
- Web (Servlet, JSP, JSF)
- Database (JDBC, JPA)
- Enterprise (EJB, CDI, JTA)
- Security (JWT, OAuth, SAML)
- Messaging (JMS)
- RESTful Services (JAX-RS)

### 자동 Feature 감지

**스마트 추천:**
```
배포된 애플리케이션 분석 결과:
- JPA 사용 감지 → jpa-2.2 Feature 추천
- RESTful API 감지 → jaxrs-2.1 Feature 추천
```

## JVM 설정

### JVM 옵션

**메모리 설정:**
```bash
-Xms512m          # 초기 힙 크기
-Xmx2048m         # 최대 힙 크기
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m
```

**GC 설정:**
```bash
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:ParallelGCThreads=4
```

**디버깅:**
```bash
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=7777
```

### 환경 변수

**server.env 파일:**
```bash
JAVA_HOME=/opt/java/jdk-11
WLP_USER_DIR=/opt/liberty/usr
LOG_DIR=/var/log/liberty
```

## 보안 설정

### 관리자 인증

**사용자 레지스트리:**
```xml
<basicRegistry id="basic">
    <user name="admin" 
          password="{xor}Lz4sLCgwLTs=" />
</basicRegistry>
```

### SSL/TLS 설정

**키 스토어 구성:**
```xml
<keyStore id="defaultKeyStore"
          password="Liberty"
          location="key.p12"
          type="PKCS12"/>

<ssl id="defaultSSLConfig"
     keyStoreRef="defaultKeyStore"
     trustStoreRef="defaultTrustStore"
     sslProtocol="TLSv1.2"/>
```

### 역할 기반 접근 제어

**보안 역할 매핑:**
```xml
<application-bnd>
    <security-role name="Admin">
        <user name="admin"/>
        <group name="administrators"/>
    </security-role>
</application-bnd>
```

## Liberty vs LibriX 비교

| 항목 | Liberty 명령줄 | Admin Center | LibriX |
|------|--------------|--------------|--------|
| 서버 관리 | SSH + 명령어 | 웹 UI (단일) | 웹 UI (다중) |
| 시작/중지 | 수동 명령 | 클릭 | 클릭 |
| 설정 편집 | 텍스트 에디터 | 제한적 | 완전한 편집기 |
| 모니터링 | 별도 도구 | 기본 | 고급 대시보드 |
| 로그 조회 | cat/tail | 기본 뷰어 | 고급 뷰어 |
| 여러 서버 | 스크립트 | 불가 | 중앙 관리 |

## 모범 사례

### 서버 명명 규칙

**권장 형식:**
```
{환경}-{서비스}-{번호}

예:
prod-web-01
prod-api-01
dev-test-01
staging-app-01
```

### 포트 할당

**표준 포트 범위:**
```
개발 환경: 9080-9089
테스트 환경: 9090-9099
스테이징: 9100-9109
운영 환경: 9110-9119

또는 외부에서는 80/443 (Apache 프록시)
```

### JVM 메모리 설정

**권장 크기:**
```
소형 애플리케이션:
  -Xms512m -Xmx1g

중형 애플리케이션:
  -Xms1g -Xmx2g

대형 애플리케이션:
  -Xms2g -Xmx4g
```

**컨테이너 환경:**
```
JVM 최대 힙 = 컨테이너 메모리 × 75%
예: 4GB 컨테이너 → -Xmx3g
```

### 로그 관리

**로그 로테이션:**
```xml
<logging 
    maxFileSize="50"
    maxFiles="10"
    traceSpecification="*=info:com.example.*=debug"/>
```

**로그 레벨:**
```
개발: DEBUG
테스트: INFO
운영: WARNING
```

## 문제 해결

### 서버가 시작되지 않음

**증상:** 시작 버튼 클릭 후 오류

**확인 사항:**
1. **포트 충돌**
   ```bash
   netstat -tulpn | grep 9080
   ```

2. **JDK 경로**
   ```bash
   echo $JAVA_HOME
   java -version
   ```

3. **server.xml 오류**
   - LibriX 로그 확인
   - messages.log 확인

4. **권한 문제**
   ```bash
   ls -la /opt/liberty/usr/servers/myServer
   ```

### 서버 응답 없음

**증상:** "알 수 없음" 상태

**원인 및 해결:**
1. **Agent 통신 문제**
   - LibriX Agent 상태 확인
   - 네트워크 연결 확인

2. **서버 행(Hang)**
   - Thread dump 생성
   - CPU/메모리 확인

3. **방화벽**
   - 관리 포트 개방 확인

### 메모리 부족

**증상:** OutOfMemoryError

**해결:**
1. **힙 크기 증가**
   ```bash
   -Xmx4g
   ```

2. **메모리 누수 분석**
   - Heap dump 생성
   - MAT로 분석

3. **세션 타임아웃**
   - 세션 타임아웃 감소
   - 세션 정리

## 성능 튜닝

### 스레드 풀

**Executor 설정:**
```xml
<executor id="DefaultExecutor"
          name="LibertyExecutor"
          coreThreads="50"
          maxThreads="100"
          keepAlive="60s"/>
```

### 연결 풀

**데이터소스 최적화:**
```xml
<dataSource id="DefaultDataSource">
    <connectionManager
        minPoolSize="10"
        maxPoolSize="50"
        connectionTimeout="30s"
        maxIdleTime="30m"/>
</dataSource>
```

### 애플리케이션 로딩

**자동 확장:**
```xml
<applicationManager 
    autoExpand="true"
    startTimeout="5m"
    stopTimeout="30s"/>
```

## 관련 문서

- [애플리케이션 서버 목록](application-servers.md) - 여러 서버 통합 관리
- [클러스터 관리](cluster.md) - 서버 그룹 관리
- [웹 서버 관리](web-server.md) - Apache HTTP Server 관리
- [애플리케이션 배포](../application/application-install.md)
- [모니터링](../monitoring/) - 상세 모니터링
- [문제 해결](../troubleshooting/) - 문제 진단 및 해결
