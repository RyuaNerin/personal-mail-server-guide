# SpamAssassin

1. 권한 설정

    ```sh
    mkdir -p /etc/mail/spamassassin/sa-update-keys /etc/mail/sa-update-keys
    chown -R spamd:spamd /etc/mail/spamassassin /etc/mail/sa-update-keys
    chmod 755 /etc/mail/spamassassin
    chmod 700 /etc/mail/spamassassin/sa-update-keys
    ```

1. `/etc/mail/spamassassin/local.cf` 파일 수정

    ```conf
    (...)

    #rewrite_header Subject *****SPAM*****
    rewrite_header Subject [SPAM]

    (...)

    # report_safe 1
    report_safe 0
    report_charset utf-8


    (...)
    ```

1. `/etc/mail/spamassassin/spamc.conf` 파일 생성

    ```conf
    # spamc global configuration file

    # max message size for scanning = 10 MB
    -s 10000000
    ```

1. 룰 업데이트

    ```sh
    sudo -u spamd /usr/bin/vendor_perl/sa-update && sudo -u spamd /usr/bin/vendor_perl/sa-compile
    ```

1. 자동 업데이트를 위한 데몬 파일 생성

    ```txt
    # /etc/systemd/system/spamassassin-update.service
    [Unit]
    Description=spamassassin housekeeping stuff
    After=network.target

    [Service]
    User=spamd
    Group=spamd
    Type=oneshot

    ExecStart=/usr/bin/vendor_perl/sa-update
    SuccessExitStatus=1
    ExecStart=/usr/bin/vendor_perl/sa-compile --quiet
    ExecStart=!/usr/bin/systemctl -q --no-block try-restart spamassassin.service
    ```

    ```txt
    # /etc/systemd/system/spamassassin-update.timer
    [Unit]
    Description=spamassassin house keeping

    [Timer]
    OnCalendar=daily
    Persistent=true

    [Install]
    WantedBy=timers.target
    ```

1. 데몬 활성화 및 실행

    ```sh
    systemctl daemon-reload

    systemctl enable spamassassin.service
    systemctl start spamassassin.service

    systemctl enable spamassassin-update.timer
    systemctl start spamassassin-update.timer
    ```
