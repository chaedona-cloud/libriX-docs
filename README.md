# LibriX 관리 문서

LibriX 관리콘솔의 전체 기능에 대한 상세 문서입니다.

## LibriX 소개

LibriX는 Open Liberty 기반의 엔터프라이즈 애플리케이션 서버 관리 솔루션으로, 크게 **LibriX Runtime**과 **LibriX Operator**로 구성됩니다.

### LibriX Runtime
- **Open Liberty** 커스터마이징 패키지
- **Apache HTTP Server** 통합 제공
- 클라우드 네이티브 및 On-Premise 환경 모두 지원
- LibriX Operator와 함께 사용하도록 최적화

### LibriX Operator
- **중앙 집중식 관리 콘솔** 제공
- WebSphere Application Server ND 스타일의 UI/UX
- Liberty Collective Controller 기능을 NodeAgent/DMGR 구조로 재해석
- Open Liberty API 기반으로 구현된 관리 기능

## 제품 배경

### Liberty 계열과 WebSphere Application Server의 차이

**Liberty 계열** (Open Liberty / WebSphere Liberty):
- 클라우드 네이티브 및 컨테이너 환경에 최적화
- server.xml 파일 수동으로 편집
- 복잡한 XML 구조와 구문 이해 필요
- JCache 구현체 구성 파일 별도 작성
- 각 서버마다 개별적으로 설정 파일 관리
- 설정 오류 발견이 어렵고 디버깅 시간 소요
- CI/CD 파이프라인과의 통합에 강점

**WebSphere Application Server** (tWAS):
- On-Premise 환경에 적합한 스탠드얼론 또는 Network Deployment
- ISC(Integrated Solutions Console) 제공 - 풍부한 기능의 웹 기반 관리 콘솔
- Network Deployment의 NodeAgent/DMGR 구조로 여러 서버 중앙 관리
- GUI를 통한 직관적인 설정 관리
- 자동 생성 설정 파일로 빌드 및 배포 자동화 용이

### LibriX의 차별화

LibriX는 두 제품군의 장점을 결합합니다:

**GUI 기반 구성**: 복잡한 XML 편집 불필요
- server.xml 및 관련 설정 파일 자동 생성
- 자동 생성 구성 파일로 빌드 및 배포 자동화 용이

**중앙 관리**: 여러 서버의 설정을 한 곳에서 관리
- tWAS ND 스타일의 중앙집중식 UI/UX
- Liberty Collective Controller 기능 제공

## 문서 구조

<details>
<summary><strong>📁 전체 파일 구조 보기</strong></summary>

```
docs/
├── server/                    # 서버 관리 문서
│   ├── application-server.md      # 애플리케이션 서버 관리
│   ├── web-server.md               # 웹 서버 관리
│   ├── session-server.md           # 세션 서버 관리
│   ├── cluster.md                  # 클러스터 관리
│   └── images/
│       ├── app_server/             # 애플리케이션 서버 이미지 (12개)
│       ├── web_server/             # 웹 서버 이미지 (8개)
│       ├── session_server/         # 세션 서버 이미지 (3개)
│       └── cluster/                # 클러스터 이미지 (5개)
├── application/               # 애플리케이션 관리 문서
│   ├── application-install.md      # 애플리케이션 설치
│   ├── enterprise-application.md   # 엔터프라이즈 애플리케이션 관리
│   └── images/
├── environment/              # 환경 관리 문서  
│   ├── session-domain.md           # 세션 도메인 관리
│   ├── liberty-variables.md        # Liberty 변수 관리
│   ├── virtual-host.md             # 가상호스트 관리
│   ├── shared-library.md           # 공유 라이브러리 관리
│   └── images/
│       ├── session_domain/         # 세션 도메인 이미지 (3개)
│       ├── liberty_variables/      # Liberty 변수 이미지 (4개)
│       ├── virtual_host/           # 가상호스트 이미지 (5개)
│       └── shared_library/         # 공유 라이브러리 이미지 (2개)
├── resource/                  # 리소스 관리 문서
│   ├── jdbc-provider.md            # JDBC 제공자 관리
│   ├── datasource.md               # 데이터소스 관리
│   ├── j2c-connection.md           # J2C 인증 데이터
│   └── images/
├── security/                  # 보안 관리 문서
│   ├── user-management.md          # 사용자 관리
│   ├── j2c-authentication-data.md  # J2C 인증 데이터
│   ├── ssl-configuration.md        # SSL 구성
│   ├── certificate-management.md   # 인증서 관리
│   └── images/
│       ├── user_management/        # 사용자 관리 이미지 (6개)
│       ├── j2c_auth/                # J2C 인증 데이터 이미지 (6개)
│       ├── ssl_configuration/      # SSL 구성 이미지 (4개)
│       └── certificate_management/ # 인증서 관리 이미지 (3개)
├── system-management/         # 시스템 관리 문서
│   ├── deployment-manager.md       # 배치 관리자 관리
│   ├── node-management.md          # 노드 관리
│   ├── node-agent.md               # 노드 에이전트 관리
│   └── images/
│       ├── deployment_manager/     # 배치 관리자 이미지 (4개)
│       ├── node_management/        # 노드 관리 이미지 (2개)
│       └── node_agent/             # 노드 에이전트 이미지 (1개)
├── monitoring/                # 모니터링 문서
│   └── images/
└── troubleshooting/           # 문제 해결 문서
    └── images/
```

</details>

## 문서 목록

### 1. 서버 관리

<details>
<summary><strong>📄 서버 관리 문서 보기</strong></summary>

#### [애플리케이션 서버 관리](docs/server/application-server.md)
LibriX에 등록된 Open Liberty 서버들을 중앙에서 관리하는 방법을 설명합니다.
- 서버 생성, 시작, 중지, 재시작 작업
- 서버 상태 모니터링
- 서버 설정 및 구성 관리

#### [웹 서버 관리](docs/server/web-server.md)
Apache HTTP Server를 LibriX에서 관리하는 방법을 설명합니다.
- 웹 서버 생성 및 구성
- 가상 호스트 설정
- 로드 밸런싱 및 라우팅 구성

#### [세션 서버 관리](docs/server/session-server.md)
세션 데이터를 공유하는 서버 그룹(세션 도메인)을 LibriX에서 관리하는 방법을 설명합니다.
- 세션 도메인 생성 및 관리
- 세션 복제 설정
- 세션 서버 추가 및 제거

#### [클러스터 관리](docs/server/cluster.md)
여러 애플리케이션 서버를 클러스터로 구성하여 관리하는 방법을 설명합니다.
- 클러스터 생성 및 구성
- 클러스터 멤버 관리
- 워크로드 분산 설정

</details>

### 2. 애플리케이션 관리

<details>
<summary><strong>📄 애플리케이션 관리 문서 보기</strong></summary>

#### [애플리케이션 설치](docs/application/application-install.md)
WAR/EAR 애플리케이션 배포 및 관리 방법을 설명합니다.
- 애플리케이션 업로드 및 배포
- 배포 대상 선택 (서버, 클러스터)
- 애플리케이션 시작/중지

#### [엔터프라이즈 애플리케이션 관리](docs/application/enterprise-application.md)
배포된 애플리케이션의 생명주기 관리 방법을 설명합니다.
- 애플리케이션 업데이트
- 무중단 배포 전략
- 애플리케이션 상태 모니터링

</details>

### 3. 리소스 관리

<details>
<summary><strong>📄 리소스 관리 문서 보기</strong></summary>

#### [JDBC 제공자 관리](docs/resource/jdbc-provider.md)
JDBC 드라이버를 관리하는 JDBC 제공자의 생성, 수정, 삭제 방법을 설명합니다.
- JDBC 제공자 생성
- 8개 데이터베이스 지원
- 데이터베이스 지원 가이드
- **스크린샷: 14개**

#### [데이터소스 관리](docs/resource/datasource.md)
애플리케이션에서 데이터베이스에 연결하기 위한 데이터소스의 생성 및 관리 방법을 설명합니다.
- JDBC 제공자 생성 대화상자
- 8개 데이터베이스 지원
- 연결 풀 구성
- J2C 인증 설정
- **스크린샷: 6개**

</details>

### 4. 환경 관리

<details>
<summary><strong>📄 환경 관리 문서 보기</strong></summary>

#### [세션 도메인 관리](docs/environment/session-domain.md)
세션 데이터를 공유하는 서버 그룹(세션 도메인)을 LibriX에서 관리하는 방법을 설명합니다.
- 세션 도메인 생성
- 세션 복제 설정
- 세션 서버 추가/제거

#### [Liberty 변수 관리](docs/environment/liberty-variables.md)
server.xml에서 사용되는 Liberty 변수를 LibriX에서 관리하는 방법을 설명합니다.
- 변수 정의 및 사용
- 환경별 설정 분리
- 변수 우선순위 관리

#### [가상호스트 관리](docs/environment/virtual-host.md)
하나의 서버에서 여러 도메인을 처리하기 위한 가상호스트 구성 방법을 설명합니다.
- 가상호스트 정의
- 호스트 별칭 설정
- 애플리케이션 매핑

#### [공유 라이브러리 관리](docs/environment/shared-library.md)
여러 애플리케이션에서 공통으로 사용하는 라이브러리를 관리하는 방법을 설명합니다.
- 공유 라이브러리 정의
- 애플리케이션 참조 설정
- 클래스로더 구성

</details>

### 5. 보안 관리

<details>
<summary><strong>📄 보안 관리 문서 보기</strong></summary>

#### [사용자 관리](docs/security/user-management.md)
LibriX 관리콘솔의 사용자 계정 관리 방법을 설명합니다.
- 사용자 계정 관리
- J2C 인증 데이터
- SSL/TLS 구성
- 인증서 관리

</details>

### 6. 시스템 관리

<details>
<summary><strong>📄 시스템 관리 문서 보기</strong></summary>

#### [배치 관리자 관리](docs/system-management/deployment-manager.md)
중앙 관리 서버인 배치 관리자를 관리하는 방법을 설명합니다.

#### [노드 관리](docs/system-management/node-management.md)
물리적/가상 머신 노드를 관리하는 방법을 설명합니다.

#### [노드 에이전트 관리](docs/system-management/node-agent.md)
각 노드에서 실행되는 노드 에이전트를 관리하는 방법을 설명합니다.

</details>

### 7. 모니터링

### 8. 문제 분석

---

## LibriX의 가치 제안

### Open Liberty/WebSphere Liberty와의 관계

LibriX는 Open Liberty 및 WebSphere Liberty를 기반으로 하며, Liberty의 경량형 기능을 훨씬 더 쉽게 제공합니다.

Liberty의 전통적인 방식:
- server.xml 파일을 수동으로 편집
- 복잡한 XML 구조와 구문 이해 필요
- JCache 구현체 구성 파일 별도 작성
- 각 서버마다 개별적으로 설정 파일 관리
- 설정 오류 발견이 어렵고 디버깅 시간 소요

LibriX의 접근 방식:
- **GUI 기반 구성**: 복잡한 XML 편집 없이 직관적으로 설정
- **자동 구성 생성**: server.xml 및 관련 설정 파일 자동 생성
- **중앙 관리**: 여러 서버의 설정을 한 곳에서 관리
- **즉각적인 검증**: 설정 입력 시 실시간 유효성 검사
- **일관성 보장**: 자동 생성으로 인한 설정 오류 최소화

### 애플리케이션 관리

- WAR/EAR 애플리케이션 배포
- 생명주기 관리 (시작, 중지, 업데이트)
- 무중단 배포 전략
- 가상호스트 및 컨텍스트 루트 구성

### 리소스 관리

- JDBC 제공자 관리 (8개 주요 DB 지원)
- 데이터소스 관리 (연결 풀 구성)
- J2C 인증 변경 관리

### 보안 관리

- 사용자 계정 관리
- J2C 인증 데이터
- SSL/TLS 구성
- 인증서 관리

### 모니터링

- 서버 모니터링 (CPU, 메모리, 스레드)
- 연결 풀 모니터링
- JNDI 네임스페이스 현황
- 실시간 대시보드 (텍스트/그래픽)
- 성능 추세 분석

### 문제 분석

- 덤프 생성 (스레드/힙/시스템)
- 실시간 로그 탐지
- 메모리 누수 탐지
- 성능 병목 분석

---

## 지원 데이터베이스

LibriX는 다음 데이터베이스를 공식 지원합니다:

- **Apache Derby** - Embedded 및 Network 모드
- **IBM DB2** - 11g, z/OS, i (JCC 및 CLI 드라이버)
- **Oracle Database** - 11g ~ 21c (Thin 및 OCI 드라이버)
- **MySQL / MariaDB** - MySQL 5.7, 8.0 및 MariaDB
- **PostgreSQL** - PostgreSQL 9.x ~ 15.x
- **Microsoft SQL Server** - SQL Server 2012 이상
- **IBM Informix** - Informix 11.x, 12.x, 14.x
- **SAP Sybase** - Sybase ASE

---

## 최근 업데이트

### 2026-01-27 ⭐ 신규

- ✅ 문제 해결 문서 추가 (1360줄)
- ✅ 3가지 덤프 유형 상세 설명 (스레드/힙/시스템)
- ✅ 실전 문제 해결 시나리오 2개
- ✅ 분석 도구 가이드 (Eclipse MAT, FastThread, IBM TMDA)
- ✅ 7개 스크린샷 추가

### 2026-01-27

- ✅ 모니터링 문서 추가 (624줄)
- ✅ 5개 모니터링 기능 문서화
- ✅ 11개 스크린샷 추가

### 2026-01-20

- ✅ 보안 관리 문서 4개 추가

---

## Github 업로드 방법

```bash
# 1. 압축 해제
unzip librix-troubleshooting-docs.zip

# 2. GitHub 저장소 이동
cd your-github-repo

# 3. 파일 복사
cp -r troubleshooting/* docs/

# 4. README.md 업데이트
cp README.md .

# 5. Git에 추가 및 커밋
git add README.md docs/troubleshooting/
git commit -m "Add Troubleshooting documentation (Dump Generation)"
git push
```

---

## 기여

문서 개선이나 오류 발견 시 이슈를 남겨주세요.
