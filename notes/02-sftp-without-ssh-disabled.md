# SFTP Without SSH Disabled

Linux 서버를 NAS와 같이 운용하는 경우, NAS만 사용하게끔 지정된 유저들에게 SFTP만을 허용하여 shell에 접속하여 명령을 실행하는 등의 행위를 하지 못하도록 하고, 원격 스토리지 용도로만 사용할 수 있도록 설정하였다.

## User Account Config

먼저, NAS 용도로만 서버를 이용하도록 할 유저들을 `nas` 그룹에 넣는다. `nasuser` 사용자를 NAS 용도로만 이용하도록 설정해보자.

```sh
$ sudo addgroup nas
$ sudo adduser nasuser
```

다음, `nasuser` 사용자의 데이터를 저장할 디렉토리를 생성하자. 예를 들어, `/media/drive/nasuser/storage/`에 저장한다고 가정하자. 이 때 `/media/drive/nasuser/`은 반드시 **소유자가 `root`**이어야 한다.

```sh
$ sudo mkdir -p /media/drive/nasuser/
$ sudo chown -R nasuser:nasuser /media/drive/nasuser/storage/
$ sudo chmod 700 /media/drive/nasuser/storage/
```

## SSH config

SSH 설정 파일인 /etc/ssh/sshd_config에서 `nas` 그룹 사용자들의 SFTP만 허용하고, root directory를 설정한다. 먼저, `Subsystem sftp` 부분을 다음과 같이 변경한다.

```
Subsystem       sftp    internal-sftp
```

다음, 파일 하단에 다음과 같은 설정을 추가한다.

```
Match Group nas
    ChrootDirectory /media/drive/storage/%u/
    X11Forwarding no
    AllowTcpForwarding no
    AllowAgentForwarding no
    ForceCommand internal-sftp
```

이후 SSH 서비스를 재시작한다.

```sh
$ sudo service ssh restart
```

## Bash disable

SSH 설정 변경만으로도 SSH 접속이 비활성화 되지만, /etc/passwd 파일에서 `nasuser`의 마지막 항목인 `/bin/bash`를 `/bin/false`로 수정하여 bash도 비활성화 한다.

## Results

SFTP를 이용하여 서버에 접속하면 클라이언트에게는 `storage` 디렉토리만 보이며, `pwd` 커맨드를 실행해보면 현재 디렉토리를 루트(`/`)로 인식한다는 것을 확인할 수 있다.

## Reference

* [Stackoverflow: Allow SFTP but disallow SSH?](https://serverfault.com/questions/354615/allow-sftp-but-disallow-ssh)
