# openDKIM

- [OpenDMARC - ArchWiki](https://wiki.archlinux.org/title/OpenDMARC#DMARC_Record)

1. `/etc/opendkim/opendkim.conf` 생성 및 편집

    ```txt
    # /etc/opendkim/opendkim.conf
    BaseDirectory           /var/lib/opendkim
    Domain                  ryuar.in
    KeyFile                 /etc/opendkim/mail2048.private
    Selector                mail2048
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
    opendkim-genkey -r -b 2048 -D /etc/opendkim -s mail2048 --subdomains -d ryuar.in
    ```

1. `/etc/opendkim/mail2048.txt` 확인

    ```txt
    # cat /etc/opendkim/mail2048.txt
    mail2048._domainkey     IN      TXT     ( "v=DKIM1; k=rsa; s=email; "
        "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAuwCHmIoMxHk3cArugm9pCbU/WM1EQm+f2L69PsiJOPEnApMe5zR9mSQwXdfSxuPjeBBLtB/mnNL4gS8E/SqLIFQCGP109uPgv5wGjuXqocnTG+vsQA+jITg8MN9QlwhpAZ7uDwmWf0pknCt9qmOZYCNImGW73NUlGxWHp0rdmV/XGpPCjXjppPEnVFLpJcnADlUnmMF9E2K6ER"
        "/YQUH3m7OJKav9ssCKZFrUWz/iurRGD1nDCpU4etUdHrSRHrhY0bct8RcGqpDg5Q+qxNAZoB0eJuiKsIhplUnOzWO27l1p9kcUO9YkjVhPOpx5OnWHVCIAFiySlZUlgC14y2jaAwIDAQAB" )  ; ----- DKIM key mail2048 for ryuar.in
    ```

1. DNS 설정

    | Name                           | Type | Data                                |
    |-------------------------------:|:----:|-------------------------------------|
    |    `_adsp._domainkey.ryuar.in` | TXT  | `dkim=all`                          |
    | `mail2048._domainkey.ryuar.in` | TXT  | `v=DKIM1; k=rsa; s=email;p=.......` |

1. 권한 설정

    ```sh
    chown opendkim:postfix -R /etc/opendkim
    chmod g+rx /etc/opendkim
    chmod g+r /etc/opendkim/mail2048.private
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
