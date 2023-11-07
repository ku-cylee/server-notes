# Proxy Server using Squid

Squid를 이용하여 프록시 서버를 설치하였다. 

## 설치 및 기본 설정

먼저 다음과 같이 `squid`와 `apache2-utils`를 설치한다.

```
$ sudo apt install -y squid apache2-utils
```

## 프록시 서버 기본 설정

`/etc/squid/squid.conf` 파일에서, `TAG: http_port` 밑에 다음과 같이 `http_port`를 설정한다. (예시: 포트 7070)

```
http_port 7070
```

## 인증 설정

프록시 서버에 접근할 수 있는 계정을 생성하고, 접근을 허용하는 과정이다. 먼저, 계정 생성 및 인증은 `apache2-utils`를 통해 설치된 `htpasswd` 프로그램을 사용한다. 

```
$ sudo htpasswd -c /etc/squid/squid_password username
```

이 명령어는 `username`을 계정명으로 갖는 계정을 생성하며, 그 뒤에 입력하는 비밀번호를 암호화하여 `/etc/squid/squid_password`에 저장한다. `-c` 플래그는 파일을 새로 생성할 때 사용되며, 사용자를 "추가"할 때는 해당 플래그는 생략해도 된다.

다음은 프록시 서버에 계정 인증 방법과 접근 허용을 설정하는 방법이다. 위의 `squid.conf` 파일에서, `TAG: auth_param` 부분에 다음과 같은 설정을 추가한다. 이 설정은 `basic_ncsa_auth` 프로그램이 `/etc/squid/squid_password` 파일을 바탕으로 사용자를 인증하도록 하고, 프록시 서버에 접근할 때 반드시 인증을 요하도록 강제한다.

```
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/squid_password
auth_param basic realm proxy
acl authenticated proxy_auth REQUIRED
```

그 뒤, `squid.conf` 파일에서 `http_access allow localhost` 밑에 `http_access allow authenticated`를 추가하여 인증된 사용자의 접근을 허용하고, squid 프로그램을 다음과 같이 재시작한다.

```
$ sudo service squid restart
```

## 여담

원래는 SSH 프록시도 겸하도록 서버를 설정하려고 했으나, SSH Config 파일에서 프록시를 통한 접속을 허용하는 `ProxyJump`나 `ProxyCommand` 등의 설정이 비밀번호 기억 기능을 지원하지 않아, SSH 프록시 설정은 하지 않고 필요한 경우 프록시 서버에 SSH로 접속한 뒤, 해당 서버에서 접속하고자 하는 서버에 SSH 접속하는 방법을 사용하기로 하였다.

## 참고 자료

* https://jjeongil.tistory.com/1623
* https://simplificandoredes.com/en/squid-user-authentication/
