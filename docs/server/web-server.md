# 웹 서버 관리

## 개요

웹 서버 관리는 LibriX에서 Apache HTTP Server를 중앙 집중식으로 관리하는 기능을 제공합니다. Apache HTTP Server를 Liberty 서버 앞단에 배치하여 정적 콘텐츠 제공, 로드 밸런싱, SSL 종료 등의 역할을 수행할 수 있습니다.

## 웹 서버 아키텍처

### 전형적인 구성

```
[클라이언트]
    ↓
[Apache HTTP Server] (포트 80/443)
    ↓
[mod_proxy 또는 mod_jk]
    ↓
[Liberty 서버들] (포트 9080/9443)
```

### 웹 서버의 역할

**정적 콘텐츠 제공:**
- HTML, CSS, JavaScript, 이미지 등
- Liberty 부담 감소
- 빠른 응답 속도

**로드 밸런싱:**
- 여러 Liberty 서버에 트래픽 분산
- 세션 스티키 지원
- 헬스 체크

**SSL/TLS 종료:**
- HTTPS 트래픽 처리
- SSL 인증서 관리
- Liberty는 HTTP로 통신

**보안:**
- 방화벽 역할
- DDoS 방어
- 접근 제어

## Liberty와의 연동 방식

### 전통적인 Apache 설정 방법

#### 1. mod_proxy 사용

**수동 설정:**
```apache
# httpd.conf
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so

<VirtualHost *:80>
    ServerName www.company.com
    
    ProxyPass /app http://liberty-server:9080/app
    ProxyPassReverse /app http://liberty-server:9080/app
    
    # 로드 밸런싱
    <Proxy balancer://mycluster>
        BalancerMember http://server1:9080 route=server1
        BalancerMember http://server2:9080 route=server2
        ProxySet stickysession=JSESSIONID
    </Proxy>
    
    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/
</VirtualHost>
```

**문제점:**
- 복잡한 설정 파일 수동 편집
- 서버 추가 시마다 설정 변경
- 설정 오류 발생 쉬움
- 중앙 관리 불가

#### 2. mod_jk 사용

```apache
# workers.properties
worker.list=loadbalancer

worker.server1.type=ajp13
worker.server1.host=server1
worker.server1.port=9009

worker.server2.type=ajp13
worker.server2.host=server2
worker.server2.port=9009

worker.loadbalancer.type=lb
worker.loadbalancer.balance_workers=server1,server2
worker.loadbalancer.sticky_session=true
```

**문제점:**
- 별도 workers.properties 파일 관리
- AJP 프로토콜 설정 복잡
- Liberty와 동기화 어려움

### LibriX의 웹 서버 관리 방식

**GUI 기반 설정:**
- 웹 UI를 통한 직관적 설정
- 드래그 앤 드롭으로 서버 추가
- 실시간 설정 검증

**자동 구성:**
- Apache 설정 파일 자동 생성
- Liberty 서버 자동 감지
- 헬스 체크 자동 구성

**중앙 집중 관리:**
- 모든 웹 서버를 한 화면에서 관리
- Liberty 서버 목록 자동 동기화
- 통합 모니터링

## 웹 서버 관리 화면

웹 서버 관리 화면에 접근:

```
서버 → 웹 서버
```

### 웹 서버 목록

**표시 정보:**
- **웹 서버 이름**: 고유 식별자
- **호스트**: 웹 서버가 설치된 호스트
- **상태**: 실행 중/중지
- **포트**: HTTP/HTTPS 포트
- **백엔드 서버**: 연결된 Liberty 서버 수

## 웹 서버 생성

### 생성 전 준비사항

**Apache HTTP Server 설치:**
- 대상 호스트에 Apache 설치 필요
- LibriX Agent가 Apache 관리 권한 필요

**지원 버전:**
- Apache HTTP Server 2.4.x

### 생성 절차

**1단계: 기본 정보**

- **웹 서버 이름**: 고유 식별자
- **호스트**: Apache가 설치된 서버
- **Apache 경로**: Apache 설치 디렉토리
  - 예: `/usr/local/apache2`
  - 예: `/etc/httpd`

**2단계: 포트 설정**

```
HTTP 포트: 80
HTTPS 포트: 443
```

**방화벽 확인:**
- 포트가 개방되어 있는지 확인
- 80/443 포트는 root 권한 필요

**3단계: 백엔드 서버 선택**

연결할 Liberty 서버 선택:
- 개별 서버 선택
- 클러스터 전체 선택
- 드래그 앤 드롭으로 추가

**4단계: 로드 밸런싱 설정**

**밸런싱 방식:**
```
○ Round Robin (순차)
○ Least Connections (최소 연결)
○ IP Hash (클라이언트 IP 기반)
```

**세션 스티키:**
```
☑ 세션 스티키 활성화
쿠키 이름: JSESSIONID
```

**5단계: 헬스 체크 설정**

```
헬스 체크 URL: /health
간격: 30초
타임아웃: 5초
실패 임계값: 3회
```

**6단계: SSL/TLS 설정 (선택)**

```
☑ HTTPS 활성화
인증서 파일: /path/to/cert.crt
개인키 파일: /path/to/private.key
인증서 체인: /path/to/chain.crt (선택)
```

**7단계: 생성**

LibriX가 자동으로 수행:
- Apache 설정 파일 생성
- mod_proxy 또는 mod_jk 구성
- 백엔드 서버 목록 설정
- Apache 재시작

## 자동 생성되는 Apache 설정

### mod_proxy 기반 설정

```apache
# LibriX가 자동 생성한 설정
# /etc/httpd/conf.d/librix-web-server.conf

LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so

<VirtualHost *:80>
    ServerName www.company.com
    
    # 정적 콘텐츠
    DocumentRoot /var/www/html
    
    # Liberty 서버로 프록시
    <Proxy balancer://librix-cluster>
        BalancerMember http://server1.company.com:9080 route=server1
        BalancerMember http://server2.company.com:9080 route=server2
        BalancerMember http://server3.company.com:9080 route=server3
        ProxySet lbmethod=byrequests
        ProxySet stickysession=JSESSIONID|jsessionid
    </Proxy>
    
    # 헬스 체크
    <Location /balancer-manager>
        SetHandler balancer-manager
        Require ip 127.0.0.1
    </Location>
    
    # 애플리케이션 라우팅
    ProxyPass /balancer-manager !
    ProxyPass /static !
    ProxyPass / balancer://librix-cluster/
    ProxyPassReverse / balancer://librix-cluster/
</VirtualHost>

<VirtualHost *:443>
    ServerName www.company.com
    
    SSLEngine on
    SSLCertificateFile /path/to/cert.crt
    SSLCertificateKeyFile /path/to/private.key
    SSLCertificateChainFile /path/to/chain.crt
    
    # 동일한 프록시 설정
    ProxyPass / balancer://librix-cluster/
    ProxyPassReverse / balancer://librix-cluster/
</VirtualHost>
```

## 웹 서버 작업

### 시작/중지/재시작

**시작:**
1. 웹 서버 선택
2. "시작" 버튼 클릭
3. Apache 프로세스 시작

**중지:**
1. 웹 서버 선택
2. "중지" 버튼 클릭
3. 진행 중인 요청 완료 후 중지

**재시작:**
- 설정 변경 후 자동 재시작
- 수동 재시작 가능

### 백엔드 서버 관리

**서버 추가:**
1. 웹 서버 선택
2. "백엔드 서버 관리"
3. Liberty 서버 선택
4. 추가

**서버 제거:**
1. 제거할 서버 선택
2. "제거" 버튼
3. 확인

**실시간 적용:**
- 서버 추가/제거 즉시 반영
- Graceful reload로 무중단

### 가중치 조정

**서버별 트래픽 비율:**
```
Server1: 가중치 3 (30%)
Server2: 가중치 5 (50%)
Server3: 가중치 2 (20%)
```

**사용 시기:**
- 서버 성능 차이
- 점진적 배포 (Canary)
- A/B 테스팅

## 정적 콘텐츠 제공

### DocumentRoot 설정

```apache
<VirtualHost *:80>
    DocumentRoot /var/www/html
    
    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    # 정적 파일은 Apache에서 직접 제공
    ProxyPass /static !
    ProxyPass /images !
    ProxyPass /css !
    ProxyPass /js !
    
    # 나머지는 Liberty로
    ProxyPass / balancer://librix-cluster/
</VirtualHost>
```

### 캐싱 설정

```apache
# 정적 리소스 캐싱
<LocationMatch "\.(jpg|jpeg|png|gif|css|js)$">
    Header set Cache-Control "max-age=86400, public"
</LocationMatch>

# mod_cache 사용
CacheEnable disk /
CacheRoot /var/cache/apache2/proxy
CacheMaxFileSize 1000000
CacheIgnoreNoLastMod On
```

## SSL/TLS 설정

### 인증서 관리

**Let's Encrypt 통합:**
LibriX는 Let's Encrypt 자동 갱신 지원

**수동 인증서:**
```
1. 인증서 업로드
2. 개인키 업로드
3. 체인 인증서 업로드 (선택)
4. 자동 적용
```

### SSL 보안 강화

```apache
# 강력한 SSL 설정
SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite HIGH:!aNULL:!MD5
SSLHonorCipherOrder on

# HSTS
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"

# 기타 보안 헤더
Header always set X-Frame-Options "SAMEORIGIN"
Header always set X-Content-Type-Options "nosniff"
Header always set X-XSS-Protection "1; mode=block"
```

### HTTP to HTTPS 리다이렉트

```apache
<VirtualHost *:80>
    ServerName www.company.com
    Redirect permanent / https://www.company.com/
</VirtualHost>
```

## 모니터링

### Apache 상태 확인

LibriX는 실시간 모니터링 제공:

**서버 상태:**
- CPU 사용률
- 메모리 사용량
- 활성 연결 수
- 초당 요청 수

**백엔드 서버 상태:**
- 각 서버 응답 시간
- 헬스 체크 상태
- 요청 분산 비율
- 오류율

### 로그 관리

**실시간 로그 조회:**
```
액세스 로그: /var/log/httpd/access_log
에러 로그: /var/log/httpd/error_log
```

**로그 레벨 조정:**
```apache
LogLevel warn
CustomLog logs/access_log combined
ErrorLog logs/error_log
```

## Liberty와의 비교

| 항목 | Liberty Admin Center | LibriX 웹 서버 관리 |
|------|---------------------|-------------------|
| Apache 관리 | 불가능 | 중앙 집중 관리 |
| 로드 밸런싱 | 수동 설정 | GUI 기반 설정 |
| SSL 인증서 | 수동 관리 | 통합 관리 |
| 헬스 체크 | 수동 구성 | 자동 구성 |
| 모니터링 | 별도 도구 필요 | 통합 대시보드 |

## 모범 사례

### 아키텍처

**DO:**
- Apache를 DMZ에 배치
- Liberty는 내부 네트워크에 배치
- SSL 종료는 Apache에서 수행

**DON'T:**
- Liberty를 직접 인터넷에 노출
- 불필요한 프록시 레벨 추가

### 성능 최적화

**정적 콘텐츠 분리:**
```apache
# 이미지, CSS, JS는 Apache에서 직접
ProxyPass /static !
Alias /static /var/www/static

# 동적 콘텐츠만 Liberty로
ProxyPass / balancer://liberty/
```

**압축 활성화:**
```apache
LoadModule deflate_module modules/mod_deflate.so

<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/plain text/xml
    AddOutputFilterByType DEFLATE application/javascript
    AddOutputFilterByType DEFLATE text/css
</IfModule>
```

**연결 재사용:**
```apache
KeepAlive On
KeepAliveTimeout 5
MaxKeepAliveRequests 100

# 백엔드 연결 재사용
ProxyPass / balancer://liberty/ keepalive=On
```

### 보안

**접근 제어:**
```apache
<Location /admin>
    Require ip 10.0.0.0/8
    Require ip 192.168.1.0/24
</Location>
```

**Rate Limiting:**
```apache
LoadModule ratelimit_module modules/mod_ratelimit.so

<Location />
    SetOutputFilter RATE_LIMIT
    SetEnv rate-limit 400
</Location>
```

## 문제 해결

### Apache가 시작되지 않음

**원인 및 해결:**

1. **포트 충돌**
   ```bash
   netstat -tulpn | grep :80
   lsof -i :80
   ```

2. **설정 오류**
   ```bash
   apachectl configtest
   httpd -t
   ```

3. **권한 문제**
   - 80/443 포트는 root 권한 필요
   - SELinux 확인

### 백엔드 서버 연결 실패

**원인 및 해결:**

1. **Liberty 서버 미실행**
   - Liberty 서버 상태 확인

2. **방화벽**
   ```bash
   telnet server1 9080
   nc -zv server1 9080
   ```

3. **프록시 타임아웃**
   ```apache
   ProxyTimeout 300
   ```

### 502 Bad Gateway

**원인:**
- Liberty 서버 응답 없음
- 백엔드 서버 다운

**해결:**
1. Liberty 로그 확인
2. 헬스 체크 확인
3. 네트워크 연결 확인

## 성능 튜닝

### Apache MPM 설정

**Worker MPM (권장):**
```apache
<IfModule mpm_worker_module>
    StartServers             3
    MinSpareThreads         75
    MaxSpareThreads        250
    ThreadsPerChild         25
    MaxRequestWorkers      400
    MaxConnectionsPerChild   0
</IfModule>
```

**Event MPM (고성능):**
```apache
<IfModule mpm_event_module>
    StartServers             3
    MinSpareThreads         75
    MaxSpareThreads        250
    ThreadsPerChild         25
    MaxRequestWorkers      400
    MaxConnectionsPerChild   0
</IfModule>
```

### 모니터링 메트릭

**주요 지표:**
- 초당 요청 수 (RPS)
- 평균 응답 시간
- 에러율
- CPU/메모리 사용률
- 동시 연결 수

**알림 설정:**
- 에러율 > 5%
- 응답 시간 > 3초
- CPU 사용률 > 80%

## 관련 문서

- [애플리케이션 서버 관리](application-server.md)
- [클러스터 관리](cluster.md)
- [가상호스트 관리](../environment/virtual-host.md)
- [SSL 구성](../security/ssl-configuration.md)
