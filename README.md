# 🐧 리눅스 계정 및 파일 관리 학습

리눅스 서버 환경에서 사용자·그룹 관리와 파일 관리를 중심으로 운영 시스템 보안과 협업 효율성을 높이는 방법을 학습 및 정리했습니다.

## 📌 주요 학습 내용

- 사용자 및 그룹 관리

- 그룹 기반 권한 설정과 디렉토리 접근 제어

- 권한 기반 접근 제한 (허용/거부 검증)

- diff를 활용한 파일 변경 추적 및 비교

<br>

## 👥 사용자 및 그룹 관리
#### 사용자 생성
```bash
sudo adduser user1
sudo adduser user2
```

#### 그룹 생성 및 확인
```bash
sudo groupadd engineer
tail /etc/group   # engineer:x:1005
```

#### 그룹에 사용자 추가
```bash
usermod -aG engineer user1
usermod -aG engineer user2
```

#### 그룹 사용자 확인
```bash
tail /etc/group # engineer:x:1026:user1,user2
```
<br>

## 📂 디렉토리 생성 및 권한 관리
#### 디렉토리 생성
- 루트 계정에서 진행

```bash
mkdir -p /home/engineer
```

#### 디렉토리 권한 변경
- 현재 권한 확인
```bash
# 현재 권한
ls -al /home/engineer
drwxr-xr-x 2 root root 4096 Sep  9 15:23 .
drwxr-xr-x 7 root root 4096 Sep  9 15:23 ..
```

- 권한 변경
```bash
# 사용자와 그룹: 읽기/쓰기/실행 허용
# 기타 사용자: 접근 불가
chmod 770 /home/engineer
ls -al /home/engineer/
drwxrwx--- 2 root root 4096 Sep  9 15:23 .
drwxr-xr-x 7 root root 4096 Sep  9 15:23 ..

# 디렉토리 소유권 변경
chown -R :engineer /home/engineer
ls -al /home/engineer/
drwxrwx--- 2 root engineer 4096 Sep  9 15:23 .
drwxr-xr-x 7 root root     4096 Sep  9 15:23 ..
```
<br>

## ✏️ 권한 검증
#### 그룹에 속한 사용자의 접근 : ✅ 파일 생성 및 접근 가능
```bash
# user1이 engineer 그룹 소속일 때
user1@myserver00:~$ touch /home/Engineer/file.txt
user1@myserver00:~$
user1@myserver00:~$ ls /home/Engineer/file.txt
/home/Engineer/file.txt
```
#### 그룹에 속하지 않은 사용자의 접근 : ❌ 접근 거부 (Permission denied)
- 그룹에 추가되지 않은 사용자가  조회, 접근, 파일생성시 접근 거부
```bash
# engineer 그룹에 속하지 않은 사용자
hskim@myserver00:~$ cd /home/engineer
-bash: cd: /home/engineer: Permission denied

hskim@myserver00:~$ touch /home/engineer/test.txt
touch: cannot touch '/home/engineer/test.txt': Permission denied

hskim@myserver00:~$ ls /home/Engineer/
ls: cannot open directory '/home/Engineer/': Permission denied
```
<br>

## 🎯 diff를 이용한 설정 파일 변경 추적

#### 준비: 원본 파일 복사
```bash
sudo cp /etc/ssh/sshd_config /home/ubuntu/sshd_config_duplication

```
#### 변경 내용을 확인하기 위해 /etc/ssh/sshd_config_duplication 파일 내용 수정.

```bash
# 생략...
#
Port 2200
AddressFamily any
ListenAddress 192.168.0.0/24
#ListenAddress ::
# 생략...

```
#### 설정 변경 내용 확인
- (-): 원본 파일에서는 있었지만, 새로운 파일에서는 삭제되거나 변경된 줄을 나타냄
- (+): 새로운 파일에 추가되었거나, 변경된 내용을 나타냄.

```bash
ubuntu@myserver00:~$ sudo diff -u /etc/ssh/sshd_config /home/ubuntu/sshd_config_duplication

--- /etc/ssh/sshd_config        2025-06-10 02:22:39.000000000 +0900
+++ /home/ubuntu/sshd_config_duplication        2025-09-09 16:07:18.743332930 +0900
@@ -20,9 +20,9 @@
 #   systemctl daemon-reload
 #   systemctl restart ssh.socket
 #
-#Port 22
-#AddressFamily any
-#ListenAddress 0.0.0.0
+Port 2200
+AddressFamily any
+ListenAddress 192.168.0.0/24
 #ListenAddress ::
 #HostKey /etc/ssh/ssh_host_rsa_key

```

➡️ **변경 내용**
 - ssh 포트 22번 -> 2200번
 - 특정 IP 대역(192.168.0.0/24)에서만 접속 허용.

