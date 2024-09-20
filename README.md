# 🔑(Linux)PAM 비밀번호 규제 실습


<br>

## 💡PAM 설명

<br>
<br>

### 🛠기존 방식의 문제
PAM을 도입하기 전의 리눅스 시스템에서는 각 응용 프로그램이 사용자 인증을 자체적으로 처리해야 했다. 이 과정에서 시스템에 저장된 사용자 정보를 활용하려면, 응용 프로그램이 /etc/passwd 같은 중요한 시스템 파일에 접근할 권한이 필요했기 때문에 보안 침해의 위험이 존재했다.


### 🌟PAM 장점
소프트웨어 개발과 인증 및 안전한 권한 부여 체계를 분리하기 위해 만들어졌기 때문에, 이를 통해 인증을 수행하면 응용 프로그램에서 직접 인증 로직을 구현할 필요가 없어 개발이 간소화되고, 시스템 파일인 passwd 파일 등을 열람하지 않아도 되는 장점이 있다.


### 🛡PAM 관리 방법:
PAM 설정 파일: PAM은 /etc/pam.d/ 디렉터리에 있는 설정 파일을 통해 관리된다. 각 응용 프로그램이나 서비스(예: login, sshd, passwd)는 별도의 PAM 설정 파일을 갖고 있다.

📄예시:

/etc/pam.d/sshd: SSH 로그인에 대한 PAM 설정
/etc/pam.d/passwd: 비밀번호 변경에 대한 PAM 설정
🔧PAM 모듈: PAM은 다양한 모듈을 사용해 특정 인증 규칙을 적용한다. 각 모듈은 특정 인증 기능을 제공한다. 

### 📚대표적인 모듈

- pam_unix.so: 기본적인 사용자 인증을 담당하며, /etc/passwd나 /etc/shadow 파일을 사용해 인증한다.
- am_pwquality.so: 비밀번호의 복잡성 규칙(최소 길이, 복잡성, 사전 단어 체크 등)을 설정한다.
- pam_tally2.so: 로그인 실패 횟수를 제한해 계정 잠금 기능을 제공힌다.
- PAM 규칙 설정 방식: 각 PAM 설정 파일에는 4가지 컨트롤 플래그가 있으며, 이를 통해 인증 절차가 어떻게 처리될지 정의할 수 있다.


### 컨트롤 플래그
auth: 사용자의 인증 절차를 정의 (예: 비밀번호 입력 요구)
account: 계정의 접근 제한을 정의 (예: 계정이 활성화된 상태인지 확인)
password: 비밀번호 변경 시 적용될 규칙 정의
session: 로그인 세션 설정 (예: 로그인 시 필요한 환경 설정)


<br>
<br>

<br>

## 📝실습

<br>
<br>

### 🔒비밀번호 규제 설정을 위한 PAM(PAM 모듈) 중 하나인 pam_pwquality 모듈을 설치

<br>
<br>

```
sudo nano /etc/pam.d/common-password  # Ubuntu/Debian
sudo nano /etc/pam.d/system-auth  # RHEL/CentOS
```

<br>
<br>

### ⚙️설정 파일 열기
```
sudo vi /etc/pam.d/common-password
```

<br>
<br>

### 🔍pw 조건을 설정하는 부분을 찾는다.
`password requisite pam_pwquality`


<br>

### 🔧다음과 같이 수정
```
#
# /etc/pam.d/common-password - password-related modules common to all services
#
# This file is included from other service-specific PAM config files,
# and should contain a list of modules that define the services to be
# used to change user passwords.  The default is pam_unix.

# Explanation of pam_unix options:
# The "yescrypt" option enables
#hashed passwords using the yescrypt algorithm, introduced in Debian
#11.  Without this option, the default is Unix crypt.  Prior releases
#used the option "sha512"; if a shadow password hash will be shared
#between Debian 11 and older releases replace "yescrypt" with "sha512"
#for compatibility .  The "obscure" option replaces the old
#`OBSCURE_CHECKS_ENAB' option in login.defs.  See the pam_unix manpage
#for other options.

# As of pam 1.0.1-6, this file is managed by pam-auth-update by default.
# To take advantage of this, it is recommended that you configure any
# local modules either before or after the default block, and use
# pam-auth-update to manage selection of other modules.  See
# pam-auth-update(8) for details.

# here are the per-package modules (the "Primary" block)
password        requisite                       pam_pwquality.so retry=3 minlen=8 difok=3 # 이부분 설정 수정
password        [success=1 default=ignore]      pam_unix.so obscure use_authtok try_first_pass yescrypt
# here's the fallback if no module succeeds
password        requisite                       pam_deny.so
# prime the stack with a positive return value if there isn't one already;
# this avoids us returning an error just because nothing sets a success code
# since the modules above will each just jump around
password        required                        pam_permit.so
# and here are more per-package modules (the "Additional" block)
# end of pam-auth-update config
```

<br>

 - retry - 최대 비밀번호 변경 시도 횟수
 - minlen - 최소 비밀번호 길이
 - difok - 이전 비밀번호와 달라야하는 최소 글자수


<br>
<br>


### ❗기존 비밀번호는 유지되며, 비밀번호 변경 시에 설정을 참고하여 비밀번호를 규제한다.
<br>

### 🔄비밀번호 변경 시도
```
passwd <user 명>
```
![11111](https://github.com/user-attachments/assets/390e4c0c-763c-4728-b44d-5e0a0a2d12ef)


<br>

- 설정한 비밀번호 최소 길이보다 짧은 경우 ❌

- 불용어 사전에 있는 글자를 포함하는 경우 ❌

- 이전 비밀번호와 달라야하는 최소 글자수를 만족하지 못하는 경우 ❌

실패한 모습



### ✅설정완료
