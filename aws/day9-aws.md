# Day 9 - Private EC2 Nginx 웹 서버와 내부 통신

## 오늘의 목표

Private EC2에 Nginx 웹 서버를 설치하고,
Public EC2에서 Private IP를 이용해 접속한다.

또한 Security Group과 Nginx 로그를 확인하여
Public Subnet과 Private Subnet 사이의 내부 통신을 검증한다.

---

## 1. Nginx 설치

Private EC2에서 Nginx를 설치했다.

```bash
sudo apt install nginx -y
```

Nginx는 HTTP 요청을 받아 HTML 파일을
사용자에게 전달하는 웹 서버 소프트웨어이다.

---

## 2. Nginx 웹페이지 수정

Nginx의 기본 웹페이지 파일을 수정했다.

```bash
sudo nano /var/www/html/index.html
```

Nginx가 기본적으로 제공하는 파일은 다음과 같다.

```text
/var/www/html/index.html
```

수정한 HTML:

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Private EC2 Nginx</title>
</head>
<body>
    <h1>Private EC2 Nginx Server</h1>
    <p>Public EC2를 경유하여 접속했습니다.</p>
    <p>이 웹 서버는 Private Subnet에서 실행 중입니다.</p>
</body>
</html>
```

`sudo`는 관리자 권한으로 명령어를 실행하는 옵션이고,
`nano`는 터미널에서 사용하는 텍스트 편집기이다.

---

## 3. Nginx 실행 상태 확인

```bash
sudo systemctl status nginx --no-pager
```

확인 결과:

```text
Active: active (running)
```

Nginx 서비스가 정상적으로 실행 중인 것을 확인했다.

---

## 4. Nginx의 80번 포트 확인

```bash
sudo ss -tulnp | grep ':80'
```

확인 결과 Nginx가 다음 주소에서
HTTP 요청을 기다리고 있었다.

```text
0.0.0.0:80
[::]:80
```

의미:

* `LISTEN`: 연결을 기다리는 상태
* `:80`: HTTP 기본 포트
* `0.0.0.0:80`: 모든 IPv4 주소에서 요청 대기
* `[::]:80`: IPv6 주소에서 요청 대기
* `nginx`: 80번 포트를 사용하는 프로그램

---

## 5. Private EC2 내부에서 웹페이지 확인

```bash
curl http://localhost
```

수정한 Nginx HTML 페이지가 정상적으로 출력되었다.

`localhost`는 현재 명령어를 실행하고 있는
Private EC2 자기 자신을 의미한다.

```text
Private EC2
    |
    v
Nginx
    |
    v
/var/www/html/index.html
```

---

## 6. Security Group 설정

Private EC2의 Security Group에
다음 인바운드 규칙을 추가했다.

```text
유형: HTTP
프로토콜: TCP
포트: 80
소스: Public EC2의 Security Group
```

인터넷 전체를 의미하는 `0.0.0.0/0`을 허용하지 않고,
Public EC2의 Security Group만 HTTP 80 포트에
접속할 수 있도록 제한했다.

---

## 7. Public EC2에서 Private EC2의 Nginx 접속

Public EC2에서 다음 명령어를 실행했다.

```bash
curl http://10.0.2.129
```

Private EC2의 Nginx HTML 페이지가 정상적으로 출력되었다.

```text
Public EC2
10.0.1.72
    |
    | HTTP 80
    | VPC 내부 통신
    v
Private EC2
10.0.2.129
    |
    v
Nginx
```

이 통신은 인터넷을 거치지 않고
VPC의 `local` 경로를 이용한 내부 통신이다.

---

## 8. Nginx 접속 로그 확인

Private EC2에서 Nginx 접속 로그를 확인했다.

```bash
sudo tail -n 20 /var/log/nginx/access.log
```

로그 예시:

```text
::1 ... "GET / HTTP/1.1" 200
10.0.1.72 ... "GET / HTTP/1.1" 200
```

로그 해석:

* `::1`: Private EC2 자기 자신에서 접속
* `10.0.1.72`: Public EC2에서 접속
* `GET /`: 기본 웹페이지 요청
* `200`: 정상 응답

Public EC2의 Private IP인 `10.0.1.72`가
로그에 기록된 것을 통해 Public EC2에서
Private EC2의 Nginx로 요청이 전달된 것을 확인했다.

---

## 9. NAT Gateway와 내부 통신의 차이

이번 Nginx 접속에는 NAT Gateway가 필요하지 않았다.

```text
Public EC2 → Private EC2
```

는 VPC 내부의 `local` 경로를 이용하기 때문이다.

NAT Gateway는 다음과 같은 경우에 사용한다.

```text
Private EC2 → 외부 인터넷
```

즉:

```text
NAT Gateway
= Private EC2가 인터넷으로 나가는 출구

local route
= VPC 내부 서버끼리 통신하는 경로
```

---

## 오늘 확인한 최종 구조

```text
내 PC
  |
  | SSH
  v
Public EC2
10.0.1.72
  |
  | VPC 내부 HTTP 80 통신
  v
Private EC2
10.0.2.129
  |
  v
Nginx
  |
  v
/var/www/html/index.html
```

---

## 오늘 배운 핵심

1. Nginx는 웹페이지를 제공하는 웹 서버이다.
2. Nginx의 기본 웹 파일은 `/var/www/html/index.html`이다.
3. HTTP는 기본적으로 80번 포트를 사용한다.
4. `curl localhost`는 현재 서버의 Nginx에 요청한다.
5. `curl http://10.0.2.129`는 Private EC2의 Nginx에 HTTP 요청을 보낸다.
6. Public EC2와 Private EC2는 VPC 내부의 `local` 경로로 통신한다.
7. Security Group의 Source를 Public EC2의 보안 그룹으로 제한할 수 있다.
8. Nginx access.log에서 요청자의 IP와 HTTP 응답 상태를 확인할 수 있다.
9. NAT Gateway는 Private EC2의 외부 인터넷 접속에만 필요하다.

Day 9에서는 Private EC2에 Nginx를 구성하고,
Public EC2에서 Private IP를 통해 웹페이지에 접근하는
내부 통신을 성공적으로 확인했다.
