# SSH Authentication and sudo without Passwords

SSH 인증 방식을 비밀번호 인증 방식에서 키 인증 방식으로 변경하고, sudo 권한에도 비밀번호 없이 진입할 수 있도록 설정하였다.

## Settings for SSH

원래 SSH 키 접근 방식은 클라이언트의 키를 서버의 authorized_keys 파일에 등록하고, 로그인 시 클라이언트 키를 제공하여 인증하는 방식이다. 그러나 모든 디바이스를 매번 등록해야 하는 번거로움을 피하기 위해 AWS EC2 서비스와 같이 하나의 키를 생성하여 서버에 등록하고 여러 클라이언트에서 하나의 키 파일을 사용하도록 하였다.

먼저 키를 생성하기 위한 key generator 스크립트를 작성하였다.

```sh
#!/bin/bash

if [ -z "$1" ]; then
        echo "Provide SSH server name"
        exit 1
fi

ssh-keygen -t rsa -P "" -f "./$1" -m pem -C "$1"
cat "$1.pub"
mv "./$1" "./keys/$1.pem"
mv "./$1.pub" -t ./keys/
```

사용 방법은 다음과 같다. 스크립트 실행 후 ./keys/ 디렉토리 내에 .pub, .pem 파일이 생성되며, pub 파일의 내용이 출력된다. (실행 위치는 컴퓨터와 전혀 무관하다. 제 3의 컴퓨터에서도 가능)

```sh
$ ./generator.sh server-name
```

이제 pub 파일의 내용을 서버의 ~/.ssh/authorized_keys 파일에 추가하고, pem 파일은 클라이언트의 ~/.ssh 디렉토리에 복사한 뒤 사용한다. 또한, 서버에서 SSH config 파일 (/etc/ssh/sshd_config)에서 `PasswordAuthentication` 값을 `no`로 변경하였다.

## Settings for sudo

AWS와 같이 sudo 권한에 진입할 때 비밀번호 인증을 bypass 하도록 설정하였다.

```sh
$ echo "$USER ALL=(ALL:ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/$USER
```

위 명령어를 실행하면 현재 로그인 한 유저는 sudo 권한 진입 시 비밀번호 인증을 생략하게 된다. `$USER`의 값을 다른 유저의 아이디로 변경하여 실행하면 해당 유저 역시 비밀번호 인증을 생략한다.
