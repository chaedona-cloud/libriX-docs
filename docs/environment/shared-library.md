# 공유 라이브러리 관리

## 개요

공유 라이브러리는 여러 애플리케이션에서 공통으로 사용하는 JAR 파일이나 클래스를 중앙에서 관리하는 기능입니다. LibriX를 통해 공유 라이브러리를 정의하고, 애플리케이션에 연결하여 코드 중복을 줄이고 유지보수를 간소화할 수 있습니다.

## 공유 라이브러리란?

공유 라이브러리는 여러 애플리케이션이 공통으로 사용하는 Java 라이브러리(JAR 파일)를 서버 레벨에서 정의한 것입니다. 각 애플리케이션에 라이브러리를 포함시키지 않고, 서버의 공유 라이브러리를 참조하여 사용합니다.

### Liberty의 전통적인 라이브러리 관리 방식

#### 1. WEB-INF/lib에 포함 (전통적 방식)

모든 JAR 파일을 각 WAR 파일에 포함:

```
myapp.war
├── WEB-INF/
│   ├── lib/
│   │   ├── commons-lang3-3.12.0.jar
│   │   ├── jackson-databind-2.13.0.jar
│   │   ├── spring-core-5.3.0.jar
│   │   └── (100+ JARs...)
│   ├── classes/
│   └── web.xml
└── index.jsp
```

**문제점:**
- WAR 파일 크기 증가 (수백 MB)
- 동일한 라이브러리 중복 저장
- 버전 업데이트 시 모든 WAR 재빌드
- 메모리 낭비 (각 애플리케이션마다 로드)
- 배포 시간 증가

#### 2. server.xml에 library 직접 정의

```xml
<server>
    <library id="SharedLib">
        <fileset dir="/opt/shared-libs" includes="*.jar"/>
    </library>
    
    <webApplication location="app1.war">
        <classloader commonLibraryRef="SharedLib"/>
    </webApplication>
    
    <webApplication location="app2.war">
        <classloader commonLibraryRef="SharedLib"/>
    </webApplication>
</server>
```

**문제점:**
- 수동 XML 편집 필요
- 파일 경로 하드코딩
- 여러 서버에 동일 설정 반복
- 버전 관리 어려움
- 중앙 관리 불가

### LibriX의 공유 라이브러리 방식

**GUI 기반 관리:**
- 파일 업로드 또는 경로 지정
- 드래그 앤 드롭으로 애플리케이션 연결
- 시각적 의존성 뷰

**자동 구성:**
- server.xml 자동 업데이트
- 클래스로더 설정 자동 적용
- 버전 충돌 자동 감지

**중앙 집중식:**
- 모든 공유 라이브러리를 한 곳에서 관리
- 클러스터 단위 일괄 적용
- 통합 버전 관리

## 공유 라이브러리의 이점

### 디스크 공간 절약

**중복 제거:**
- 공통 라이브러리를 한 번만 저장
- 여러 애플리케이션이 참조
- WAR 파일 크기 대폭 감소

**예제:**
```
기존 방식:
- app1.war: 150MB (100MB는 라이브러리)
- app2.war: 145MB (95MB는 라이브러리)
- app3.war: 155MB (105MB는 라이브러리)
총 450MB

공유 라이브러리 사용:
- 공유 라이브러리: 100MB
- app1.war: 50MB
- app2.war: 50MB
- app3.war: 50MB
총 250MB (약 44% 절감)
```

### 메모리 효율성

**클래스 공유:**
- 라이브러리를 한 번만 메모리 로드
- 모든 애플리케이션이 동일한 인스턴스 사용
- JVM 힙 메모리 절약

### 유지보수 간소화

**중앙 업데이트:**
- 공유 라이브러리만 업데이트
- 모든 애플리케이션에 즉시 반영
- 개별 WAR 재배포 불필요

**버전 관리:**
- 라이브러리 버전 일관성 유지
- 의존성 충돌 최소화

## 공유 라이브러리 관리 화면

공유 라이브러리 관리 화면에 접근:

```
환경 → 공유 라이브러리
```

### 라이브러리 목록

**표시 정보:**
- **라이브러리 이름**: 고유 식별자
- **파일 수**: 포함된 JAR 파일 개수
- **위치**: 라이브러리 파일 경로
- **참조 수**: 사용 중인 애플리케이션 수
- **설명**: 라이브러리 용도

## 공유 라이브러리 생성

### 생성 방법

**1단계: 기본 정보**

- **라이브러리 이름**: 영문, 숫자, 하이픈, 언더스코어
- **설명**: 라이브러리 용도 (선택)

**2단계: 파일 지정**

LibriX는 두 가지 방식을 지원합니다:

#### 방법 1: 파일 업로드

```
파일 선택 → JAR 파일 업로드 → 자동 저장
```

**특징:**
- 파일을 LibriX 서버로 업로드
- 자동으로 공유 디렉토리에 저장
- 파일 경로 자동 관리

**업로드 제한:**
- 최대 파일 크기: 100MB per file
- 지원 형식: .jar, .zip
- 여러 파일 동시 업로드 가능

#### 방법 2: 경로 지정

```
파일 경로: /opt/shared-libs/mylib
패턴: *.jar
```

**특징:**
- 기존 디렉토리의 파일 사용
- 심볼릭 링크 지원
- 와일드카드 패턴 사용

**경로 예제:**
```
단일 파일:
/opt/libs/commons-lang3-3.12.0.jar

디렉토리 전체:
/opt/libs/*.jar

특정 패턴:
/opt/libs/spring-*.jar
```

**3단계: 적용 범위**

```
○ 전역 (모든 서버)
○ 서버별
○ 클러스터별
```

**4단계: 생성**

## 애플리케이션에 공유 라이브러리 연결

### 연결 방법

**방법 1: 애플리케이션 설치 시 지정**

애플리케이션 설치 마법사에서:

```
고급 설정 → 공유 라이브러리 → 선택
```

**방법 2: 기존 애플리케이션 수정**

```
엔터프라이즈 애플리케이션 → 선택 → 편집 
→ 공유 라이브러리 → 추가
```

**방법 3: LibriX 관리 콘솔**

```
공유 라이브러리 → 선택 → "애플리케이션 연결"
→ 애플리케이션 선택 → 확인
```

### server.xml 자동 생성

LibriX가 자동으로 생성하는 설정:

```xml
<server>
    <!-- 공유 라이브러리 정의 -->
    <library id="MySharedLib">
        <fileset dir="/opt/shared-libs/mylib" includes="*.jar"/>
    </library>
    
    <!-- 애플리케이션에 연결 -->
    <webApplication location="myapp.war">
        <classloader commonLibraryRef="MySharedLib"/>
    </webApplication>
</server>
```

## 클래스로더 설정

### Parent First vs Parent Last

공유 라이브러리 사용 시 클래스로더 위임 모델 선택:

#### Parent First (기본값)

```
부모 클래스로더 (공유 라이브러리)
    ↓ 먼저 검색
애플리케이션 클래스로더 (WEB-INF/lib)
```

**특징:**
- 공유 라이브러리의 클래스 우선
- 일관된 버전 사용
- 충돌 가능성 낮음

**사용 시기:**
- 표준 라이브러리 사용
- 버전 일관성 중요
- 대부분의 경우

#### Parent Last

```
애플리케이션 클래스로더 (WEB-INF/lib)
    ↓ 먼저 검색
부모 클래스로더 (공유 라이브러리)
```

**특징:**
- 애플리케이션의 클래스 우선
- 버전 충돌 회피
- 독립적인 라이브러리 버전 사용

**사용 시기:**
- 특정 버전 라이브러리 필요
- 공유 라이브러리와 충돌
- 레거시 애플리케이션

### 설정 예제

```xml
<!-- Parent First (기본값) -->
<webApplication location="app1.war">
    <classloader commonLibraryRef="SharedLib" 
                 delegation="parentFirst"/>
</webApplication>

<!-- Parent Last -->
<webApplication location="app2.war">
    <classloader commonLibraryRef="SharedLib" 
                 delegation="parentLast"/>
</webApplication>
```

## 공유 라이브러리 수정

### 라이브러리 업데이트

**파일 교체:**
1. 공유 라이브러리 선택
2. "파일 관리" 버튼
3. 새 JAR 파일 업로드 또는 기존 파일 삭제
4. 저장

**버전 업그레이드:**
```
기존: commons-lang3-3.12.0.jar
신규: commons-lang3-3.13.0.jar

절차:
1. 새 버전 업로드
2. 구 버전 삭제 (선택)
3. 애플리케이션 재시작 (자동 또는 수동)
```

### 영향받는 애플리케이션 확인

LibriX는 변경 전 영향 분석 제공:

```
이 라이브러리를 사용 중인 애플리케이션:
- myapp1 (Server1, Server2)
- myapp2 (Cluster1)
- myapp3 (Server3)

변경 사항 적용을 위해 애플리케이션 재시작이 필요합니다.
```

## 공유 라이브러리 삭제

### 삭제 절차

1. **사용 확인**
   - 참조하는 애플리케이션이 없는지 확인

2. **연결 해제**
   - 모든 애플리케이션에서 라이브러리 제거

3. **삭제 실행**
   - "삭제" 버튼 클릭
   - 확인

### 주의사항

**사용 중인 라이브러리:**
LibriX는 사용 중인 라이브러리 삭제 방지:

```
오류: 이 라이브러리는 다음 애플리케이션에서 사용 중입니다:
- myapp1
- myapp2

먼저 애플리케이션에서 라이브러리 참조를 제거하세요.
```

## Liberty와 LibriX 비교

| 항목 | Open Liberty | LibriX |
|------|-------------|--------|
| 라이브러리 정의 | XML 수동 편집 | GUI 기반 생성 |
| 파일 관리 | 파일 시스템 직접 관리 | 업로드 또는 경로 지정 |
| 애플리케이션 연결 | XML 편집 | 드래그 앤 드롭 |
| 버전 관리 | 수동 | 통합 관리 |
| 의존성 확인 | 없음 | 시각적 의존성 뷰 |
| 영향 분석 | 수동 확인 | 자동 분석 제공 |

## 모범 사례

### 라이브러리 분류

**카테고리별 구성:**

```
Common:
- commons-lang3-3.13.0.jar
- commons-io-2.11.0.jar
- commons-collections4-4.4.jar

Logging:
- slf4j-api-1.7.36.jar
- logback-classic-1.2.11.jar

Web:
- spring-web-5.3.23.jar
- spring-webmvc-5.3.23.jar

Database:
- hikaricp-5.0.1.jar
- commons-dbcp2-2.9.0.jar
```

### 버전 명명 규칙

**라이브러리 이름에 버전 포함:**

```
DO:
- SharedLib-Common-2.x
- SharedLib-Spring-5.3
- SharedLib-Hibernate-5.6

DON'T:
- SharedLib
- MyLib
- CommonLib
```

### 의존성 관리

**최소 의존성 원칙:**
- 필요한 라이브러리만 포함
- 과도한 라이브러리 추가 지양
- 정기적으로 미사용 라이브러리 정리

**버전 일관성:**
- 동일한 라이브러리의 여러 버전 혼용 방지
- Spring Framework 등 통합 프레임워크는 전체 버전 일치

## 일반적인 공유 라이브러리 예제

### 유틸리티 라이브러리

```
라이브러리: SharedLib-Utils
포함 파일:
- commons-lang3-3.13.0.jar
- commons-io-2.11.0.jar
- commons-collections4-4.4.jar
- guava-31.1-jre.jar

용도: 일반적인 유틸리티 기능
```

### 로깅 라이브러리

```
라이브러리: SharedLib-Logging
포함 파일:
- slf4j-api-1.7.36.jar
- logback-classic-1.2.11.jar
- logback-core-1.2.11.jar

용도: 통합 로깅 프레임워크
```

### Spring Framework

```
라이브러리: SharedLib-Spring-5.3
포함 파일:
- spring-core-5.3.23.jar
- spring-context-5.3.23.jar
- spring-beans-5.3.23.jar
- spring-web-5.3.23.jar
- spring-webmvc-5.3.23.jar

용도: Spring 기반 애플리케이션
```

### 데이터베이스 연결

```
라이브러리: SharedLib-JDBC
포함 파일:
- postgresql-42.5.0.jar
- mysql-connector-java-8.0.30.jar
- ojdbc8-21.7.0.0.jar
- HikariCP-5.0.1.jar

용도: 데이터베이스 드라이버 및 커넥션 풀
```

## 문제 해결

### ClassNotFoundException

**증상:** 애플리케이션 시작 시 클래스를 찾을 수 없음

**원인 및 해결:**

1. **공유 라이브러리 미연결**
   - 애플리케이션에 라이브러리 연결 확인
   - server.xml의 commonLibraryRef 확인

2. **JAR 파일 누락**
   - 공유 라이브러리에 필요한 JAR 포함 확인
   - 파일 경로 및 패턴 확인

3. **클래스로더 위임 문제**
   - parentFirst/parentLast 설정 확인
   - 필요 시 parentLast로 변경

### NoClassDefFoundError

**증상:** 런타임에 클래스 정의를 찾을 수 없음

**원인 및 해결:**

1. **의존성 누락**
   - 필요한 모든 JAR 파일 포함 확인
   - 전이적 의존성(transitive dependencies) 확인

2. **버전 충돌**
   - 동일 라이브러리의 여러 버전 확인
   - 버전 호환성 검증

### ClassCastException

**증상:** 클래스 캐스팅 오류

**원인 및 해결:**

1. **클래스로더 분리**
   - 동일 클래스가 다른 클래스로더에서 로드됨
   - parentFirst 사용 권장

2. **버전 불일치**
   - 공유 라이브러리와 WEB-INF/lib의 버전 확인
   - 중복 제거

## 성능 고려사항

### 메모리 사용

**모니터링:**
- 공유 라이브러리 로드 후 힙 사용량 확인
- PermGen/Metaspace 크기 적절히 설정

**JVM 옵션:**
```bash
# Java 8
-XX:MaxPermSize=256m

# Java 11+
-XX:MaxMetaspaceSize=256m
```

### 시작 시간

**최적화:**
- 필수 라이브러리만 포함
- 큰 JAR 파일 분할 고려
- 지연 로딩(Lazy Loading) 활용

## 보안

### 라이브러리 검증

**DO:**
- 공식 저장소에서 다운로드
- 체크섬 검증
- 취약점 스캔 (OWASP Dependency Check)

**DON'T:**
- 출처 불명의 JAR 사용
- 구버전 방치
- 미검증 라이브러리 사용

### 접근 제어

**파일 권한:**
```bash
# 공유 라이브러리 디렉토리
chown -R liberty:liberty /opt/shared-libs
chmod 755 /opt/shared-libs
chmod 644 /opt/shared-libs/*.jar
```

## 관련 문서

- [Liberty 변수 관리](liberty-variables.md)
- [세션 도메인 관리](session-domain.md)
- [가상호스트 관리](virtual-host.md)
- [애플리케이션 설치](../application/application-install.md)
- [서버 관리](../server/application-server.md)
