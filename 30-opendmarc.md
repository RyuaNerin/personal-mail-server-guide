# OpenDMARC

- [OpenDMARC - ArchWiki](https://wiki.archlinux.org/title/OpenDMARC#DMARC_Record)

1. DNS 설정

    | Name              | Type | Data                             |
    |:-----------------:|:----:|:--------------------------------:|
    | `_dmarc.ryuar.in` | TXT  | `v=DMARC1; p=reject; sp=reject;` |

1. `ignore.hosts` 생성 및 권한 설정

    ```sh
    # vim /etc/opendmarc/ignore.hosts
    127.0.0.0/8
    ::1/128
    localhost
    ```

    ```sh
    chown opendmarc:postfix /etc/opendmarc/ignore.hosts
    chmod 640 /etc/opendmarc/ignore.hosts
    ```

1. `/etc/opendmarc/opendmarc.conf` 파일 수정

    ```conf
    (...)

    # AuthservID name
    AuthservID ryuar.in

    (...)

    # AutoRestart false
    AutoRestart true

    (...)

    # IgnoreHosts /etc/opendmarc/ignore.hosts
    IgnoreHosts /etc/opendmarc/ignore.hosts

    (...)

    #Socket unix:/var/spool/opendmarc/opendmarc.sock
    Socket unix:/run/opendmarc/opendmarc.sock
    ```

1. 디렉토리 생성 및 권한 설정

    ```sh
    chown opendmarc:postfix -R /etc/opendmarc/

    mkdir /run/opendmarc
    chown opendmarc:postfix /run/opendmarc
    ```

1. 부팅 시 디렉토리 생성 및 권한 설정하도록 설정

    ```sh
    echo "D /run/opendmarc 0750 opendmarc postfix" > /etc/tmpfiles.d/opendmarc.conf
    ```

1. opendmarc 데몬 설정 변경

    ```sh
    # systemctl edit opendmarc
    [Service]
    Group=
    Group=postfix
    ```

1. 데몬 활성화 및 시작

    ```txt
    systemctl daemon-reload
    systemctl enable opendmarc
    systemctl start opendmarc
    ```
