# Liberty 변수 관리

## 개요

Liberty 변수 관리는 server.xml 및 기타 구성 파일에서 사용되는 변수를 LibriX 관리콘솔에서 중앙 집중식으로 관리하는 기능을 제공합니다. 이를 통해 환경별로 다른 값을 설정하거나, 민감한 정보를 안전하게 관리할 수 있습니다.

## Liberty 변수란?

Liberty 변수는 server.xml에서 `${variable_name}` 형식으로 참조되는 값으로, 구성의 유연성과 재사용성을 높입니다.

### Liberty의 전통적인 변수 관리 방식

Open Liberty에서 변수를 정의하는 전통적인 방법들:

#### 1. server.xml에 직접 정의

```xml
<server>
    <variable name="httpPort" value="9080"/>
    <variable name="httpsPort" value="9443"/>
    <variable name="db.host" value="localhost"/>
    <variable name="db.port" value="5432"/>
    
    <httpEndpoint id="defaultHttpEndpoint"
                  host="*"
                  httpPort="${httpPort}"
                  httpsPort="${httpsPort}"/>
</server>
```

**문제점:**
- server.xml에 하드코딩
- 환경별로 다른 파일 관리 필요
- 민감한 정보(비밀번호 등) 노출 위험
- 변경 시 서버 재시작 필요

#### 2. server.env 파일 사용

```bash
# server.env
HTTP_PORT=9080
HTTPS_PORT=9443
DB_HOST=localhost
DB_PORT=5432
```

server.xml에서 참조:
```xml
<variable name="httpPort" defaultValue="${HTTP_PORT}"/>
```

**문제점:**
- 파일 시스템 직접 편집 필요
- 여러 서버에 일괄 적용 어려움
- 변경 이력 추적 불가
- 중앙 관리 불가능

#### 3. bootstrap.properties 사용

```properties
# bootstrap.properties
http.port=9080
https.port=9443
db.host=localhost
db.port=5432
```

**문제점:**
- 텍스트 파일 수동 편집
- 각 서버마다 개별 관리
- 오타 및 구성 오류 발생 쉬움
- GUI 기반 관리 도구 없음

### LibriX의 변수 관리 방식

LibriX는 WebSphere Application Server 스타일의 GUI 기반 변수 관리를 제공합니다:

**중앙 집중식 관리**
- 모든 서버의 변수를 한 화면에서 관리
- 서버/클러스터 단위로 변수 정의
- 변수 상속 및 오버라이드 지원

**GUI 기반 편집**
- 웹 UI를 통한 직관적 편집
- 실시간 유효성 검사
- 자동 완성 및 도움말 제공

**자동 구성 생성**
- server.xml 자동 업데이트
- bootstrap.properties 자동 생성
- 설정 충돌 자동 감지

**보안 강화**
- 민감한 값 암호화 저장
- 접근 권한 제어
- 변경 이력 감사 로그

## 변수 유형

LibriX는 다음과 같은 변수 유형을 지원합니다:

### 시스템 변수
Liberty 서버에서 자동으로 정의되는 변수:
- `${wlp.install.dir}`: Liberty 설치 디렉토리
- `${wlp.user.dir}`: 사용자 디렉토리
- `${server.config.dir}`: 서버 구성 디렉토리
- `${server.output.dir}`: 서버 출력 디렉토리

### 사용자 정의 변수
관리자가 직접 정의하는 변수:
- 애플리케이션 포트
- 데이터베이스 연결 정보
- 외부 서비스 URL
- 환경별 설정 값

### 환경 변수
운영체제 환경 변수를 Liberty에서 참조:
- `${env.PATH}`
- `${env.JAVA_HOME}`
- 사용자 정의 환경 변수

## Liberty 변수 관리 화면

Liberty 변수 관리 화면에 접근하려면 다음 경로로 이동합니다:

```
환경 → Liberty 변수
```

![Liberty 변수 메뉴](images/liberty_variables/liberty_variables_menu.png)

### 변수 목록 화면

![변수 목록](images/liberty_variables/variables_list.png)

**표시 정보:**
- **변수 이름**: 변수의 고유 식별자
- **값**: 변수에 할당된 값
- **범위**: 변수가 적용되는 범위 (서버, 클러스터, 전역)
- **설명**: 변수에 대한 설명
- **수정 일시**: 마지막 수정 시간

## 변수 생성

### 새 변수 추가

![변수 생성](images/liberty_variables/create_variable.png)

**단계:**

1. **"새로 만들기" 버튼 클릭**

2. **기본 정보 입력**
   - **변수 이름**: 영문, 숫자, 언더스코어(_), 점(.) 사용 가능
   - **값**: 변수에 할당할 값
   - **설명**: 변수의 용도 설명 (선택사항)

3. **적용 범위 선택**
   ```
   ○ 전역 (모든 서버)
   ○ 서버별 (특정 서버)
   ○ 클러스터별 (클러스터의 모든 멤버)
   ```

4. **보안 옵션**
   - ☑ **민감한 정보**: 비밀번호 등 민감한 값은 암호화 저장
   - ☐ **읽기 전용**: 변경 불가능한 변수로 설정

5. **저장**

### 변수 이름 규칙

**권장 명명 규칙:**
- 소문자와 점(.) 사용: `db.host`, `app.port`
- 의미 있는 이름 사용: `database.connection.url`
- 환경 접두사: `dev.db.host`, `prod.db.host`

**예제:**
```
http.port=9080
https.port=9443
db.host=localhost
db.port=5432
db.name=myapp
app.context.root=/myapp
log.level=INFO
```

## 변수 사용

### server.xml에서 변수 참조

변수는 `${variable_name}` 형식으로 참조합니다:

```xml
<server>
    <!-- HTTP 포트 -->
    <httpEndpoint id="defaultHttpEndpoint"
                  httpPort="${http.port}"
                  httpsPort="${https.port}"/>
    
    <!-- 데이터소스 -->
    <dataSource id="myDataSource">
        <jdbcDriver libraryRef="PostgreSQL"/>
        <properties.postgresql
            serverName="${db.host}"
            portNumber="${db.port}"
            databaseName="${db.name}"
            user="${db.user}"
            password="${db.password}"/>
    </dataSource>
    
    <!-- 애플리케이션 -->
    <webApplication location="myapp.war"
                    contextRoot="${app.context.root}"/>
</server>
```

### 기본값 설정

변수가 정의되지 않았을 때 사용할 기본값 지정:

```xml
<variable name="http.port" defaultValue="9080"/>
<variable name="db.host" defaultValue="localhost"/>
```

### 변수 간 참조

다른 변수를 참조하여 새 변수 정의:

```xml
<variable name="db.url" 
          value="jdbc:postgresql://${db.host}:${db.port}/${db.name}"/>
```

## 변수 범위 및 우선순위

### 변수 범위

**1. 전역 범위**
- 모든 서버에 적용
- 공통 설정에 사용
- 예: 기본 로그 레벨, 공통 라이브러리 경로

**2. 클러스터 범위**
- 특정 클러스터의 모든 멤버에 적용
- 클러스터별 설정에 사용
- 예: 클러스터 전용 데이터베이스, 서비스 URL

**3. 서버 범위**
- 특정 서버에만 적용
- 서버 고유 설정에 사용
- 예: HTTP 포트, 호스트 이름

### 우선순위

변수 해석 순서 (높은 우선순위 → 낮은 우선순위):

1. **서버별 변수**
2. **클러스터별 변수**
3. **전역 변수**
4. **server.xml의 기본값**
5. **시스템 변수**

**예제:**
```
전역: http.port=9080
클러스터: http.port=9090
서버: http.port=9100

→ 서버에서 사용되는 값: 9100
```

## 변수 수정

![변수 수정](images/liberty_variables/edit_variable.png)

### 수정 절차

1. **변수 목록에서 수정할 변수 선택**
2. **"편집" 버튼 클릭**
3. **값 또는 설정 변경**
4. **저장**

### 변경 사항 적용

**자동 적용:**
- 대부분의 변수 변경은 즉시 적용
- 실행 중인 서버에 동적으로 반영

**서버 재시작 필요:**
- 일부 구성 변경은 서버 재시작 필요
- 재시작이 필요한 경우 경고 메시지 표시

## 변수 삭제

### 삭제 절차

1. **삭제할 변수 선택**
2. **"삭제" 버튼 클릭**
3. **확인 대화상자에서 확인**

### 주의사항

**삭제 전 확인:**
- 변수를 참조하는 구성이 있는지 확인
- 삭제 시 영향받는 서버 목록 확인
- 테스트 환경에서 먼저 검증

**의존성 확인:**
LibriX는 변수가 사용 중인 경우 경고를 표시:
```
경고: 이 변수는 다음 위치에서 사용 중입니다:
- server.xml: httpEndpoint
- dataSource: myDataSource

계속 진행하시겠습니까?
```

## 환경별 변수 관리

### 개발/테스트/운영 환경 분리

**전략 1: 변수 이름에 환경 접두사**
```
dev.db.host=localhost
test.db.host=test-db.company.com
prod.db.host=prod-db.company.com
```

server.xml:
```xml
<variable name="db.host" defaultValue="${env.db.host}"/>
```

**전략 2: 서버 그룹별 변수 정의**
- 개발 서버: `http.port=9080`
- 테스트 서버: `http.port=19080`
- 운영 서버: `http.port=80`

**전략 3: 외부 속성 파일 사용**
각 환경에 맞는 bootstrap.properties 배포

## 민감한 정보 관리

### 비밀번호 및 암호화

**암호화 저장:**
LibriX에서 "민감한 정보" 옵션 선택 시:
- 값이 AES-256으로 암호화
- 화면에 마스킹 처리 (`******`)
- 로그에 기록되지 않음

**예제:**
```
변수 이름: db.password
값: ******** (암호화됨)
민감한 정보: ☑
```

### Liberty의 비밀번호 암호화

Liberty의 securityUtility 도구와 통합:

```bash
# 비밀번호 인코딩
securityUtility encode myPassword

# 출력: {xor}Lz4sLCgwLTs=
```

server.xml에서 사용:
```xml
<variable name="db.password" value="{xor}Lz4sLCgwLTs="/>
```

## Liberty와 LibriX 비교

| 항목 | Open Liberty | LibriX |
|------|-------------|--------|
| 변수 정의 | 파일 편집 (xml, properties) | GUI 기반 웹 인터페이스 |
| 범위 관리 | 서버별 개별 설정 | 전역/클러스터/서버 계층 구조 |
| 민감 정보 | 수동 암호화 필요 | 자동 암호화 및 마스킹 |
| 다중 서버 | 각 서버 개별 관리 | 중앙 집중식 관리 |
| 변경 적용 | 서버 재시작 필요 (대부분) | 동적 적용 (대부분) |
| 변경 이력 | 없음 | 감사 로그 자동 기록 |

## 모범 사례

### 변수 명명 규칙

**DO:**
- 소문자와 점 사용: `db.host`
- 의미 있는 이름: `database.connection.timeout`
- 환경 구분: `prod.api.endpoint`

**DON'T:**
- 대문자와 언더스코어: `DB_HOST` (환경 변수와 혼동)
- 모호한 이름: `var1`, `temp`
- 특수문자: `db@host`, `port#1`

### 변수 구조화

**계층적 구조 사용:**
```
# 데이터베이스 관련
db.host=localhost
db.port=5432
db.name=myapp
db.user=admin
db.password=******

# HTTP 관련
http.port=9080
http.timeout=30000

# 애플리케이션 관련
app.name=MyApplication
app.version=1.0.0
app.context.root=/myapp
```

### 보안

**민감한 정보 분리:**
- 비밀번호, API 키는 별도 변수로 관리
- "민감한 정보" 옵션 항상 사용
- 최소 권한 원칙 적용

**예제:**
```
✓ db.password (민감한 정보: ☑)
✓ api.key (민감한 정보: ☑)
✗ db.host (민감한 정보: ☐)
✗ http.port (민감한 정보: ☐)
```

## 문제 해결

### 변수가 해석되지 않음

**증상:** `${variable_name}` 그대로 표시

**원인 및 해결:**
1. **변수가 정의되지 않음**
   - 변수 목록에서 확인
   - 올바른 범위에 정의되었는지 확인

2. **변수 이름 오타**
   - server.xml에서 변수 이름 확인
   - 대소문자 구분 주의

3. **순환 참조**
   - 변수가 서로를 참조하는지 확인
   - 의존성 체인 점검

### 변수 변경이 적용되지 않음

**원인 및 해결:**
1. **서버 재시작 필요**
   - 일부 설정은 재시작 필요
   - 경고 메시지 확인

2. **캐시 문제**
   - 서버 재시작으로 캐시 초기화

3. **범위 우선순위**
   - 더 높은 우선순위의 변수가 있는지 확인

## 관련 문서

- [세션 도메인 관리](session-domain.md)
- [공유 라이브러리 관리](shared-library.md)
- [가상호스트 관리](virtual-host.md)
- [서버 관리](../server/application-server.md)
