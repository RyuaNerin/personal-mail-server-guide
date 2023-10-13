# Fail2ban

- `10분` 동안 `10번` 이상 로그인 실패 시 `1시간 밴`

1. `/etc/fail2ban/fail2ban.conf` 파일 수정

    ```conf
    (...)

    #dbpurgeage = 1d
    dbpurgeage = 7d

    (...)
    ```

1. `/etc/fail2ban/jail.conf` 파일 수정

    ```conf
    (...)

    #ignoreip = 127.0.0.1/8 ::1
    ignoreip = 127.0.0.1/8 ::1 10.0.0.0/30 10.10.0.0/24

    (...)

    #bantime  = 10m
    bantime = 1h

    (...)

    #findtime  = 10m
    findtime  = 10m

    (...)

    #maxretry = 5
    maxretry = 10

    (...)

    # sshd 섹션에 추가 또는 수정
    [sshd]
    mode    = aggressive
    port    = ssh
    logpath = %(sshd_log)s
    backend = %(sshd_backend)s
    enabled = true

    (...)

    # postfix 섹션에 추가 또는 수정
    mode    = more
    port    = smtp,465,submission,2525
    logpath = %(postfix_log)s
    backend = %(postfix_backend)s
    enabled = true

    # postfix 섹션 아래에 추가
    [postfix-ddos]
    filter   = postfix[mode=ddos]
    port     = smtp,465,submission,2525
    logpath  = %(postfix_log)s
    backend  = %(postfix_backend)s
    enabled  = true

    # # 1분동안 10번 이상 요청 = 1시간 밴
    findtime = 1m
    bantime  = 1h
    maxretry = 10

    [postfix-auth]
    filter   = postfix[mode=auth]
    port     = smtp,465,submission,2525
    logpath  = %(postfix_log)s
    backend  = %(postfix_backend)s
    enabled  = true

    (...)

    # dovecot 섹션에 추가 또는 수정
    [dovecot]
    port    = pop3,pop3s,imap,imaps,submission,465,sieve
    logpath = %(dovecot_log)s
    backend = %(dovecot_backend)s
    enabled = true

    # dovecot 섹션 밑에 추가
    [dovecot-aggressive]

    filter = dovecot[mode=aggressive]
    port    = pop3,pop3s,imap,imaps,submission,465,sieve
    logpath = %(dovecot_log)s
    backend = %(dovecot_backend)s
    enabled = true

    (...)
    ```

1. 데몬 활성화 및 실행

    ```sh
    systemctl daemon-reload
    systemctl enable fail2ban
    systemctl start fail2ban
    ```
