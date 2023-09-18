# openDKIM

- [OpenDMARC - ArchWiki](https://wiki.archlinux.org/title/OpenDMARC#DMARC_Record)

1. `/etc/opendkim/opendkim.conf` 생성 및 편집

    ```txt
    # /etc/opendkim/opendkim.conf
    BaseDirectory           /var/lib/opendkim
    Domain                  ryuar.in
    KeyFile                 /etc/opendkim/myselector.private
    Selector                myselector
    Socket                  local:/run/opendkim/opendkim.sock
    Syslog                  Yes
    TemporaryDirectory      /run/opendkim
    UMask                   002
    RequireSafeKeys         false
    ```

1. BaseDirectory 생성 및 권한 설정

    ```sh
    mkdir -p /var/lib/opendkim
    chown opendkim:postfix /var/lib/opendkim
    ```

1. 키 생성

    ```sh
    opendkim-genkey -r -s myselector -d ryuar.in
    ```

1. `myselector.txt` 확인

    ```txt
    # cat /etc/opendkim/myselector.txt
    myselector._domainkey   IN      TXT     ( "v=DKIM1; k=rsa; s=email; "
            "p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDr4x55IEi26Yft6/U/H+wGw3z/VwrVboDKCPZ4ctKz0h2ELpya+wJ2rEw2TIzAnGB8Lw5KqjirHkE/4TvPAw434urVckqFSocSqeiHIcOofA6uORH9ieTVUGJ0WXe06abaZ1EoHoWeTrczfgPPf+GmIlBUgQj1QVWhJc4e2hUKmQIDAQAB" )  ; ----- DKIM key myselector for ryuar.in
    ```

1. DNS 설정

    | Name                             | Type | Data                                |
    |---------------------------------:|:----:|-------------------------------------|
    |      `_adsp._domainkey.ryuar.in` | TXT  | `dkim=all`                          |
    | `myselector._domainkey.ryuar.in` | TXT  | `v=DKIM1; k=rsa; s=email;p=.......` |

1. 권한 설정

    ```sh
    chown opendkim:postfix -R /etc/opendkim
    chmod g+rx /etc/opendkim
    chmod g+r /etc/opendkim/myselector.private
    ```

1. opendmarc 데몬 설정 작성

    ```txt
    # /etc/systemd/system/opendkim.service
    [Unit]
    Description=OpenDKIM daemon
    After=network.target remote-fs.target nss-lookup.target

    [Service]
    Type=forking
    User=opendkim
    Group=postfix
    ExecStart=/usr/bin/opendkim -x /etc/opendkim/opendkim.conf
    RuntimeDirectory=opendkim
    RuntimeDirectoryMode=0750

    [Install]
    WantedBy=multi-user.target
    ```

1. 데몬 활성화 및 시작

    ```txt
    systemctl daemon-reload
    systemctl enable opendkim
    systemctl start opendkim
    ```