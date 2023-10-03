# Amavisd

1. `/etc/amavisd/amavisd.conf` 설정 변경

    ```conf
    (...)

    #@bypass_spam_checks_maps = (1);  # controls running of anti-spam code
    @bypass_spam_checks_maps = (0);

    (...)

    #$max_servers = 2;            # num of pre-forked children (2..30 is common), -m
    #$daemon_group = 'amavis';     # (no default;  customary: vscan or amavis), -g
    $max_servers = 1;
    $daemon_group = 'postfix';

    (...)

    #$mydomain = 'example.com';
    $mydomain = 'ryuar.in';

    (...)

    #@local_domains_maps = ( [".$mydomain"] );  # list of all local domains
    @local_domains_maps = ( [".$mydomain", "$mydomain", "ryuanerin.kr", ".ryuanerin.kr"] );
    dkim_key('ryuanerin.kr', 'mail2048', '/etc/opendkim/mail2048.private');
    dkim_key(  'ryuaner.in', 'mail2048', '/etc/opendkim/mail2048.private');
    dkim_key(    'ryuar.in', 'mail2048', '/etc/opendkim/mail2048.private');
    @dkim_signature_options_bysender_maps = (
        { 'ryuanerin.kr' => { ttl => 21*24*3600, c => 'relaxed/simple' } },
        {   'ryuaner.in' => { ttl => 21*24*3600, c => 'relaxed/simple' } },
        {     'ryuar.in' => { ttl => 21*24*3600, c => 'relaxed/simple' } },
    );

    (...)

    #$sa_tag_level_deflt  = 2.0;
    $sa_tag_level_deflt  = -999;

    (...)

    #$sa_mail_body_size_limit = 400*1024; # don't waste time on SA if mail is larger
    $sa_mail_body_size_limit = 10*1024*1024;

    (...)

    #$sa_spam_subject_tag = '***Spam*** ';
    $sa_spam_subject_tag = '[Spam] ';
    $allowed_added_header_fields {lc ('X-Spam-Report')} = 1;

    (...)

    # $myhostname = 'host.example.com';  # must be a fully-qualified domain name!
    $myhostname = 'ryuar.in';

    (...)

    # $final_virus_destiny      = D_DISCARD;
    # $final_banned_destiny     = D_DISCARD;
    # $final_spam_destiny       = D_PASS;  #!!!  D_DISCARD / D_REJECT
    # $final_bad_header_destiny = D_PASS;
    $final_virus_destiny      = D_REJECT;
    $final_banned_destiny     = D_PASS;
    $final_spam_destiny       = D_PASS;  #!!!  D_DISCARD / D_REJECT
    $final_bad_header_destiny = D_PASS;

    (...)
    ```

1. 데몬 설정 변경

    ```sh
    # systemctl edit amavisd
    [Service]
    Group=
    Group=postfix
    ```

1. 데몬 활성화 및 실행

    ```sh
    systemctl daemon-reload
    systemctl enable amavisd
    systemctl start amavisd
    ```

1. 테스트

    1. telnet 으로 amavis에 연결

        ```sh
        $ telnet 127.0.0.1 10024
        Trying 127.0.0.1...
        Connected to 127.0.0.1.
        Escape character is '^]'.
        220 [127.0.0.1] ESMTP amavisd-new service ready
        ```

    1. `ehlo 127.0.0.1` 입력

        ```txt
        ehlo 127.0.0.1
        250-[127.0.0.1]
        250-VRFY
        250-PIPELINING
        250-SIZE
        250-ENHANCEDSTATUSCODES
        250-8BITMIME
        250-SMTPUTF8
        250-DSN
        250 XFORWARD NAME ADDR PORT PROTO HELO IDENT SOURCE
        ```

    1. `quit` 입력
