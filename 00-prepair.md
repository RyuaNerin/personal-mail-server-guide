# 환경 설정

## 패키지 설치

1. set kr mirror (optional)

    - [Pacman Mirrorlist Generator](https://archlinux.org/mirrorlist/) 참조

    ```sh
    # etc/pacman.d/mirrorlist
    ##
    ## Arch Linux repository mirrorlist
    ## Filtered by mirror score from mirror status page
    ## Generated on 2023-09-21
    ##

    ## South Korea
    Server = https://mirror.yuki.net.uk/archlinux/$repo/os/$arch
    Server = https://mirror.siwoo.org/archlinux/$repo/os/$arch
    Server = https://mirror.premi.st/archlinux/$repo/os/$arch
    Server = https://pawe.me/archlinux/$repo/os/$arch
    Server = https://ftp.lanet.kr/pub/archlinux/$repo/os/$arch
    Server = https://ftp.harukasan.org/archlinux/$repo/os/$arch
    Server = https://seoul.mirror.pkgbuild.com/$repo/os/$arch
    Server = https://mirror.funami.tech/arch/$repo/os/$arch

    ## Worldwide
    Server = https://geo.mirror.pkgbuild.com/$repo/os/$arch
    ```

1. Update packages

    ```sh
    pacman -Syu --noconfirm
    ```

1. VPS의 urandom 개선

    1. random entropy 확인

        ```sh
        # cat /proc/sys/kernel/random/entropy_avail
        256
        ```

    1. haveged 설치

        ```sh
        pacman -S haveged
        ```

1. Reboot

    ```sh
    reboot
    ```

1. Install packages

    ```sh
    pacman -S --noconfirm \
        inetutils \
        acme.sh \
        perl-mail-spf \
        perl-netaddr-ip \
        perl-sys-hostname-long \
        dovecot \
        pigeonhole \
        opendmarc \
        opendkim \
        spamassassin \
        amavisd-new \
        postfix \
        postfix-pcre \
        fail2ban
    ```

    - 설치된 버전은 아래외 같습니다.

        |Name|Version|
        |:-:|:-:|
        |`inetutils`|`2.4-1`|
        |`acme.sh`|`3.0.6-2`|
        |`perl-mail-spf`|`2.9.0-11`|
        |`perl-netaddr-ip`|`4.079-14`|
        |`perl-sys-hostname-long`|`1.5-11`|
        |`dovecot`|`2.3.20-2`|
        |`pigeonhole`|`0.5.20-1`|
        |`opendmarc`|`1.4.2-3`|
        |`opendkim`|`2.11.0beta-5`|
        |`spamassassin`|`4.0.0-2`|
        |`amavisd-new`|`2.12.2-1`|
        |`postfix`|`3.8.2-1`|
        |`postfix-pcre`|`3.8.2-1`|
        |`fail2ban`|`1.0.2-4`|

## Filewall

```sh
ufw enable
ufw allow SSH
ufw allow IMAP
ufw allow IMAPS
ufw allow SMTP
ufw allow 'Mail submission'
ufw allow 465/tcp
ufw allow 2525/tcp
```

## DNS

| Name                  | Type | Data                          |
|----------------------:|:----:|-------------------------------|
|        `ryuanerin.kr` |  MX  | `mail.ryuar.in`               |
|          `ryuaner.in` |  MX  | `mail.ryuar.in`               |
|            `ryuar.in` |  MX  | `mail.ryuar.in`               |
|   `_mta-sts.ryuar.in` |  TXT | `v=STSv1; id=20191230T041200` |
| `_smtp._tls.ryuar.in` |  TXT | `v=TLSRPTv1;`                 |

## Set `mail` user

1. set password of `mail`

    ```sh
    passwd mail
    ```

1. `mail` 루트 디렉토리 확인

    ```sh
    # cat /etc/passwd | grep ^mail
    mail:x:8:12::/var/spool/mail:/usr/bin/nologin
    ```

1. Maildir 생성 및 권한 설정

    ```sh
    chown mail:postfix /var/spool/mail
    chmod 02770 /var/spool/mail
    ```
