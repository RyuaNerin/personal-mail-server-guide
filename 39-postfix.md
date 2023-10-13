# Postfix 설정

1. aliases 수정

    ```txt
    # /etc/postfix/aliases
    # 주석처리되지 않은 기존 내용을 삭제
    *: mail

    # ALIASES(5)                                                          ALIASES(5)
    #
    # NAME
    #        aliases - Postfix local alias database format
    (...)
    ```

    ```sh
    postalias /etc/postfix/aliases
    ```

1. virtual 설정

    ```txt
    # /etc/postfix/virtual
    @ryuar.in       mail
    @ryuaner.in     mail
    @ryuanerin.kr   mail

    # VIRTUAL(5)                                                          VIRTUAL(5)
    #
    # NAME
    #        virtual - Postfix virtual alias table format
    (...)
    ```

    ```sh
    postmap /etc/postfix/virtual
    ```

1. whitelist 설정

    - 애플 이메일을 못 받는 문제가 있어서 whitelist에 수동으로 추가하여 활용하였음.
    - 비슷한 문제 발생 시 여기에 추가

    ```txt
    # /etc/postfix/whitelist.pcre
    /^.+\.apple.com$/ OK
    ```

1. 로그인 계정을 지정하기 위해 `sender_login.pcre` 파일 생성

    ```txt
    # /etc/postfix/sender_login.pcre
    /^(.+)@ryuar\.in$/          mail
    /^(.+)@ryuaner\.in$/        mail
    /^(.+)@ryuanerin\.kr$/      mail
    ```

1. `matesr.cf` 파일 수정

    ```conf
    smtp      inet  n       -       n       -       -       smtpd
    # 위 라인 하단에 아래 내용 추가
    2525      inet  n       -       n       -       -       smtpd
    465       inet  n       -       n       -       -       smtpd
      -o syslog_name=postfix/smtps
      -o smtpd_tls_wrappermode=yes

    # 주석 해제
    submission inet n       -       n       -       -       smtpd
      -o syslog_name=postfix/submission
      -o smtpd_tls_security_level=encrypt

    (...)

    # 맨 밑에 추가
    policy-spf unix -       n       n       -       -       spawn
      user=nobody argv=/usr/lib/postfix/postfix-policyd-spf-perl

    amavisfeed unix  -    -       y       -       1       smtp
    -o syslog_name=postfix/amavisfeed
    -o smtp_data_done_timeout=1200
    -o disable_dns_lookups=yes
    -o smtp_send_xforward_command=yes

    127.0.0.1:10025 inet n  -       y       -       -       smtpd
    -o syslog_name=postfix/amavis-re
    -o content_filter=
    -o smtpd_helo_restrictions=
    -o smtpd_sender_restrictions=
    -o smtpd_client_connection_count_limit=0
    -o smtpd_client_connection_rate_limit=0
    -o smtpd_end_of_data_restrictions=
    -o smtpd_recipient_restrictions=permit_mynetworks,reject
    -o smtpd_data_restrictions=reject_unauth_pipelining
    -o smtpd_delay_reject=no
    -o mynetworks=127.0.0.0/8
    -o smtpd_error_sleep_time=0
    -o smtpd_soft_error_limit=1001
    -o smtpd_hard_error_limit=1000
    -o receive_override_options=no_header_body_checks,no_unknown_recipient_checks,no_milters
    -o smtpd_helo_required=no
    -o smtpd_client_restrictions=permit_mynetworks,reject
    -o smtpd_restriction_classes=
    -o disable_vrfy_command=no
    -o local_header_rewrite_clients=

    ```

1. `main.cf`

    ```conf
    (...)

    #myhostname = virtual.domain.tld
    myhostname = mail.ryuar.in

    (...)

    #mydomain = domain.tld
    mydomain = ryuar.in

    (...)

    #myorigin = $myhostname
    myorigin = $myhostname

    (...)

    #inet_interfaces = all
    inet_interfaces = all

    (...)

    #mydestination = $myhostname, localhost.$mydomain, localhost
    mydestination =
        localhost,
        ryuar.in,
        ryuaner.in,
        ryuanerin.kr,
        mail.ryuar.in

    (...)

    #mynetworks = 168.100.3.0/28, 127.0.0.0/8
    mynetworks =
        127.0.0.0/8,
        [::1]/128,
        [fe80::]/64,
        10.10.0.0/24

    (...)

    #inet_protocols = ipv4
    inet_protocols = all

    (...)

    #alias_maps = dbm:/etc/aliases
    alias_maps = hash:/etc/postfix/aliases

    (...)

    #alias_database = dbm:/etc/aliases
    alias_database = $alias_maps

    (...)

    #home_mailbox = Maildir/
    home_mailbox = Maildir/


    (...)

    #smtpd_banner = $myhostname ESMTP $mail_name
    smtpd_banner = $myhostname ESMTP $mail_name

    (...)

    ##################################################
    # 이하 제일 하단에 추가

    mailbox_size_limit = 0

    # https://www.postfix.org/postconf.5.html#biff
    biff = no

    mailbox_command = /usr/lib/dovecot/dovecot-lda

    policy-spf_time_limit = 3600s

    smtputf8_enable = yes

    message_size_limit = 0
    mailbox_size_limit = 0

    append_dot_mydomain = no

    content_filter      = amavisfeed:[127.0.0.1]:10024
    non_smtpd_milters   = unix:/run/opendkim/opendkim.sock, unix:/run/opendmarc/opendmarc.sock
    #, unix:/run/clamav/clamav-milter.socket
    smtpd_milters       = unix:/run/opendkim/opendkim.sock, unix:/run/opendmarc/opendmarc.sock
    #, unix:/run/clamav/clamav-milter.socket

    # SASL
    smtpd_sasl_local_domain         =
    smtpd_sasl_auth_enable          = yes
    smtpd_sasl_security_options     = noanonymous
    smtpd_sasl_type                 = dovecot
    smtp_sasl_type                  = dovecot
    smtpd_sasl_path                 = private/auth
    smtp_sasl_path                  = private/auth
    smtpd_sasl_authenticated_header = yes

    broken_sasl_auth_clients        = no

    ###################################################################
    # TLS parameters
    smtp_tls_loglevel       = 2
    smtpd_tls_loglevel      = 2

    smtp_use_tls            = yes
    smtpd_use_tls           = $smtp_use_tls

    smtpd_tls_received_header = yes

    smtpd_tls_auth_only     = yes

    smtp_tls_note_starttls_offer    = yes

    smtp_tls_security_level         = may
    smtpd_tls_security_level        = may

    tls_preempt_cipherlist = yes

    smtpd_tls_protocols             = >=TLSv1
    smtp_tls_protocols              = $smtpd_tls_protocols

    smtpd_tls_ciphers               = high
    smtp_tls_ciphers                = $smtpd_tls_ciphers

    smtpd_tls_exclude_ciphers = aNULL, eNULL, EXPORT, DES, RC4, MD5, PSK, aECDH, EDH-DSS-DES-CBC3-SHA, EDH-RSA-DES-CBC3-SHA, KRB5-DES, CBC3-SHA, CAMELLIA, CHACHA20, AESCCM, SEED, IDEA
    smtp_tls_exclude_ciphers = $smtpd_tls_exclude_ciphers

    smtpd_tls_mandatory_protocols   = >=TLSv1
    smtp_tls_mandatory_protocols    = $smtpd_tls_mandatory_protocols

    smtpd_tls_mandatory_ciphers     = high
    smtp_tls_mandatory_ciphers      = $smtpd_tls_mandatory_ciphers

    smtpd_tls_mandatory_exclude_ciphers = aNULL, eNULL, EXPORT, DES, RC4, MD5, PSK, aECDH, EDH-DSS-DES-CBC3-SHA, EDH-RSA-DES-CBC3-SHA, KRB5-DES, CBC3-SHA, CAMELLIA, CHACHA20, AESCCM, SEED, IDEA
    smtp_tls_mandatory_exclude_ciphers = $smtpd_tls_mandatory_exclude_ciphers

    smtpd_tls_cert_file     = /etc/acme.sh/mail.ryuar.in/fullchain.cer
    smtpd_tls_key_file      = /etc/acme.sh/mail.ryuar.in/mail.ryuar.in.key
    smtpd_tls_eccert_file   = /etc/acme.sh/mail.ryuar.in_ecc/fullchain.cer
    smtpd_tls_eckey_file    = /etc/acme.sh/mail.ryuar.in_ecc/mail.ryuar.in.key
    smtpd_tls_CAfile        = /etc/acme.sh/mail.ryuar.in/ca.cer

    smtp_tls_cert_file      = $smtpd_tls_cert_file
    smtp_tls_key_file       = $smtpd_tls_key_file
    smtp_tls_eccert_file    = $smtpd_tls_eccert_file
    smtp_tls_eckey_file     = $smtpd_tls_eckey_file
    smtp_tls_CAfile         = $smtpd_tls_CAfile

    smtpd_tls_session_cache_database        = btree:${data_directory}/smtpd_scache
    smtp_tls_session_cache_database         = btree:${data_directory}/smtp_scache

    ###################################################################
    # Virtual Alias
    virtual_alias_maps      = hash:/etc/postfix/virtual

    disable_vrfy_command    = yes

    smtpd_delay_reject      = yes
    smtpd_helo_required     = yes

    # https://www.postfix.org/postconf.5.html#smtpd_helo_restrictions
    # Optional restrictions that the Postfix SMTP server applies in the context of a client HELO command.
    # 보내는 서버 주소 검사
    smtpd_helo_restrictions =
        # 로컬 허용
        permit_mynetworks,
        # 인증된 클라이언트 허용
        permit_sasl_authenticated,
        # HELO or EHLO hostname이 잘못되면 거부
        reject_invalid_helo_hostname,
        # HELO or EHLO hostname에 A or MX 레코드가 없으면 거부
        reject_unknown_helo_hostname,
        # HELO or EHLO hostname가 올바른 형식이 아니면 거부
        reject_non_fqdn_helo_hostname,
        # 스팸 필터링
        reject_rbl_client zen.spamhaus.org,
        reject_rbl_client spamlist.or.kr,
        # SPF 확인
        check_policy_service unix:private/policy-spf,
        permit

    # https://www.postfix.org/postconf.5.html#smtpd_sender_restrictions
    # Optional restrictions that the Postfix SMTP server applies in the context of a client MAIL FROM command.
    # 발신자 유효성 검사
    smtpd_sender_restrictions =
        permit_mynetworks,
        reject_unauthenticated_sender_login_mismatch,
        reject_authenticated_sender_login_mismatch,
        permit_sasl_authenticated,
        check_client_access pcre:/etc/postfix/whitelist.pcre,
        reject_unauth_destination,
        # MAIL FROM : 올바른 형식이 아니면 거부
        reject_non_fqdn_sender,
        # MAIL FROM : 1) MX 또는 A 레코드가 없거나 2) 잘못된 형식의 MX 레코드 거부
        reject_unknown_sender_domain,
        # MAIL FROM : 주소가 bounce로 알려졌거나, 주소에 도달하지 못하면 거부
        reject_unverified_sender,
        # 스팸 필터링
        reject_rbl_client zen.spamhaus.org,
        reject_rbl_client spamlist.or.kr,
        # SPF 확인
        check_policy_service unix:private/policy-spf,
        permit
    smtpd_sender_login_maps = pcre:/etc/postfix/sender_login.pcre

    # https://www.postfix.org/postconf.5.html#smtpd_recipient_restrictions
    # Optional restrictions that the Postfix SMTP server applies in the context of a client RCPT TO command, after smtpd_relay_restrictions.
    # 수신자 유효성 검사
    smtpd_recipient_restrictions =
        permit_mynetworks,
        permit_sasl_authenticated,
        # whitelist 허용 (일부 도메인으로부터 메일이 오지 않는 경우가 있음...)
        check_client_access pcre:/etc/postfix/whitelist.pcre,
        # HELO or EHLO hostname이 잘못되면 거부
        reject_invalid_helo_hostname,
        # RCPT TO : 올바른 형식이 아니면 거부
        reject_non_fqdn_recipient,
        # MAIL FROM : 올바른 형식이 아니면 거부
        reject_non_fqdn_sender,
        # 보낼 때 : RCPT TO in relay_domains 확인 / 받을 때 : RCPT TO in $mydestination, $inet_interfaces, $proxy_interfaces, $virtual_alias_domains, or $virtual_mailbox_domains
        reject_unauth_destination,
        # RCPT TO   : 1) MX 또는 A 레코드가 없거나 2) 잘못된 형식의 MX 레코드 거부
        reject_unknown_recipient_domain,
        # MAIL FROM : 1) MX 또는 A 레코드가 없거나 2) 잘못된 형식의 MX 레코드 거부
        reject_unknown_sender_domain,
        # RCPT TO   : 주소가 bounce로 알려졌거나, 주소에 도달하지 못하면 거부
        reject_unverified_recipient,
        # MAIL FROM : 주소가 bounce로 알려졌거나, 주소에 도달하지 못하면 거부
        reject_unverified_sender,
        # 스팸 필터링
        reject_rbl_client zen.spamhaus.org,
        reject_rbl_client spamlist.or.kr,
        # SPF 확인
        check_policy_service unix:private/policy-spf,
        reject_unauth_pipelining,
        permit

    # https://www.postfix.org/postconf.5.html#smtpd_client_restrictions
    # Optional restrictions that the Postfix SMTP server applies in the context of a client connection request.
    smtpd_client_restrictions =
        permit_mynetworks,
        permit_sasl_authenticated,
        reject_rbl_client zen.spamhaus.org,
        reject_rbl_client spamlist.or.kr,
        check_policy_service unix:private/policy-spf,
        defer_unauth_destination,
    #    reject

    # https://www.postfix.org/postconf.5.html#smtpd_data_restrictions
    # Optional access restrictions that the Postfix SMTP server applies in the context of the SMTP DATA command.
    smtpd_data_restrictions =
        reject_unauth_pipelining

    # https://www.postfix.org/postconf.5.html#smtpd_relay_restrictions
    # Access restrictions for mail relay control that the Postfix SMTP server applies in the context of the RCPT TO command, before smtpd_recipient_restrictions.
    smtpd_relay_restrictions =
        permit_mynetworks,
        permit_sasl_authenticated,
        permit_auth_destination,
        reject_unauth_destination

    allow_untrusted_routing = no
    ```

1. 데몬 활성화 및 실행

    ```sh
    systemctl daemon-reload
    systemctl enable postfix
    systemctl start postfix
    ```
