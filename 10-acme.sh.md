# 인증서 발급

- `acme.sh` 를 활용하여 인증서 발급

- 메일 서버 주소 : `mail.ryuar.in`

1. DNS CAA(DNS Certification Authority Authorization) 설정

    | Name       | Type | Data                              |
    |-----------:|:----:|-----------------------------------|
    | `ryuar.in` | CAA  | `0 iodef "mailto:admin@ryuar.in"` |
    | `ryuar.in` | CAA  | `0 issue "letsencrypt.org"`       |

1. 인증서 자동 발급 및 연장을 위해 `acme` 유저 생성

    ```sh
    useradd acme --home-dir /etc/acme --create-home --shell /usr/bin/nologin
    ```

1. 인증서 발급

    - 필자는 vultr를 쓰고있기 때문에 `vultr_dns` 옵션으로 인증서를 발급하였음.
    - 각자 자기 환경에 맞는 방식으로 인증서를 발급하여 사용.

    ```sh
    sudo -u acme bash -c 'VULTR_API_KEY="........." acme.sh --issue --keylength ec-256 --domain mail.ryuar.in --server letsencrypt --dns dns_vultr'
    sudo -u acme bash -c 'VULTR_API_KEY="........." acme.sh --issue --keylength 2048   --domain mail.ryuar.in --server letsencrypt --dns dns_vultr'
    chmod 640 /etc/acme.sh/mail.ryuar.in/mail.ryuar.in.key
    chmod 640 /etc/acme.sh/mail.ryuar.in_ecc/mail.ryuar.in.key
    ```

1. 자동 연장을 위해서 systemd-timers에 등록 및 활성화

    ```ini
    # /etc/systemd/system/acme.service
    [Unit]
    Description = acme.sh renewal
    After = network-online.target

    [Service]
    Type=oneshot
    ExecStart=/usr/bin/acme.sh --renew-all --home /etc/acme.sh
    ExecStart=!/usr/bin/systemctl -q --no-block reload postfix.service
    ExecStart=!/usr/bin/systemctl -q --no-block reload dovecot.service
    User=acme
    Group=acme
    ```

    ```ini
    # /etc/systemd/system/acme.timer
    [Unit]
    Description = acme.sh renewal

    [Timer]
    OnCalendar=weekly
    RandomizedDelaySec=6h
    Persistent=true

    [Install]
    WantedBy=timers.target
    ```

1. 데몬 활성화 및 시작

    ```sh
    systemctl daemon-reload
    systemctl enable acme.timer
    systemctl start acme.timer
    ```

1. 다른 데몬에서 인증서를 읽게 하기 위한 그룹 권한 설정

    ```sh
    chmod g+rx /home/acme
    gpasswd -M postfix,dovecot,dovenull acme
    ```
