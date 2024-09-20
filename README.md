# PAM 비밀번호 규제 실습


PAM(PAM 모듈) 중 하나인 pam_pwquality 모듈을 설치

```
sudo nano /etc/pam.d/common-password  # Ubuntu/Debian
sudo nano /etc/pam.d/system-auth  # RHEL/CentOS
```

설정 파일 열기
`sudo vi /etc/pam.d/common-password`


다음과 같이 수정
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

- retry - 최대 비밀번호 변경 시도 횟수
- minlen - 최소 비밀번호 길이
- difok - 이전 비밀번호와 달라야하는 최소 글자수



기존 비밀번호는 유지되며, 비밀번호 변경 시에 설정을 참고하여 비밀번호를 규제한다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/3de15034-236d-43f9-9f62-106629074d8b/15210847-fcf5-4f12-9b98-cc8e1b446bcb/image.png)

설정한 비밀번호 최소 길이보다 짧은 경우

불용어 사전에 있는 글자를 포함하거나

이전 비밀번호와 달라야하는 최소 글자수를 만족하지 못하는 경우 변경 실패 처리
