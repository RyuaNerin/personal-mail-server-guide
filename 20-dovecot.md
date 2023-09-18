# Dovecot

1. 기본 설정 복사
    - [Archlinux Wiki](https://wiki.archlinux.org/title/dovecot#Dovecot_configuration)

    ```sh
    mkdir /etc/dovecot
    cp /usr/share/doc/dovecot/example-config/dovecot.conf /etc/dovecot/dovecot.conf
    cp -r /usr/share/doc/dovecot/example-config/conf.d /etc/dovecot
    ```

1. dh.pem 파일 생성 (오래걸려요)

    ```sh
    openssl dhparam -out /etc/dovecot/dh.pem 4096
    ```

1. 스팸 메일함 생성을 위한 sieve 설정

    1. `/etc/dovecot/dovecot.sieve` 파일 생성

        ```txt
        # /etc/dovecot/dovecot.sieve
        require ["fileinto", "mailbox" ];

        if header :contains "X-Spam-Flag" "YES" {
            fileinto :create "Junk";
            stop;
        }
        ```

    1. sieve 파일 빌드

        ```sh
        cd /etc/dovecot
        sievec dovecot.sieve
        mkdir sieve
        ```

1. `/etc/users.dovecot` 생성

    ```txt
    # /etc/users.dovecot
    mail
    ```

    - 권한 설정

        ```sh
        chmod 644 /etc/users.dovecot
        ```

1. `/etc/pam.d/dovecot-auth` 생성

    ```txt
    # /etc/pam.d/dovecot-auth
    auth      required        pam_listfile.so  item=user sense=allow file=/etc/users.dovecot onerr=fail

    # same with /etc/pam.d/dovecot
    auth include system-auth
    account include system-auth
    session include system-auth
    password include system-auth
    ```

1. `/etc/dovecot/dovecot.conf` 수정

    ```conf
    (...)

    #protocols = imap pop3 lmtp submission
    protocols = imap

    (...)

    #base_dir = /var/run/dovecot/
    base_dir = /var/run/dovecot/

    (...)
    ```

1. `/etc/dovecot/conf.d/10-auth.conf` 수정

    ```conf
    (...)

    #disable_plaintext_auth = yes
    disable_plaintext_auth = no

    (...)

    #auth_mechanisms = plain
    auth_mechanisms = plain login

    (...)
    ```

1. `/etc/dovecot/conf.d/10-mail.conf` 수정

    - `mail` 계정의 uid가 8이라 설정 변경

    ```conf
    (...)

    #mail_location =
    mail_location = maildir:~/Maildir

    namespace inbox {
        (...)

        #type = private
        type = private

        (...)

        #list = yes
        list = yes

        (...)
    }

    (...)

    #first_valid_uid = 500
    first_valid_uid = 5

    (...)
    ```

1. `/etc/dovecot/conf.d/10-master.conf` 수정

    ```conf
    service imap-login {
        (...)

        inet_listener imap {
            #port = 143
            port = 143
        }
        inet_listener imaps {
            #port = 993
            port = 993
            #ssl = yes
            ssl = yes
        }

        (...)
    }

    service auth {
        (...)

        # Postfix smtp-auth
        #unix_listener /var/spool/postfix/private/auth {
        #  mode = 0666
        #}
        unix_listener /var/spool/postfix/private/auth {
            mode = 0666
            user = postfix
            group = postfix
        }
        (...)
    }
    ```

1. `/etc/dovecot/conf.d/10-ssl.conf` 수정

    ```conf
    (...)

    #ssl = yes
    ssl = required

    (...)

    #ssl_cert = </etc/ssl/certs/dovecot.pem
    #ssl_key = </etc/ssl/private/dovecot.pem
    ssl_cert     = </etc/acme.sh/mail.ryuar.in_ecc/fullchain.cer
    ssl_key      = </etc/acme.sh/mail.ryuar.in_ecc/mail.ryuar.in.key
    ssl_alt_cert = </etc/acme.sh/mail.ryuar.in/fullchain.cer
    ssl_alt_key  = </etc/acme.sh/mail.ryuar.in/mail.ryuar.in.key

    (...)

    #ssl_ca =
    ssl_ca = </etc/acme.sh/mail.ryuar.in/ca.cer

    #ssl_dh = </etc/dovecot/dh.pem
    ssl_dh = </etc/dovecot/dh.pem

    (...)

    #ssl_min_protocol = TLSv1.2
    ssl_min_protocol = TLSv1.2

    (...)

    #ssl_cipher_list = ALL:!DH:!kRSA:!SRP:!kDHd:!DSS:!aNULL:!eNULL:!EXPORT:!DES:!3DES:!MD5:!PSK:!RC4:!ADH:!LOW@STRENGTH
    # TLSv1.3 all + Recommended + OpenSSL https://ciphersuite.info/cs/?software=openssl&tls=tls13&singlepage=true&security=recommended
    ssl_cipher_list= ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-PSK-CHACHA20-POLY1305:TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384

    (...)

    #ssl_curve_list =
    ssl_curve_list = P-521:P-384:P-256

    (...)

    #ssl_prefer_server_ciphers = no
    ssl_prefer_server_ciphers = yes

    (...)
    ```

1. `/etc/dovecot/conf.d/15-lda.conf` 수정

    ```conf
    (...)
    protocol lda {
      # Space separated list of plugins to load (default is global mail_plugins).
      #mail_plugins = $mail_plugins
      mail_plugins = $mail_plugins sieve
    }

    (...)
    ```

1. `/etc/dovecot/conf.d/15-mailboxes.conf` 수정

    ```conf
    (...)

    namespace inbox {
      (...)

      # 아래 하단에 추가....
      mailbox Sent {
        special_use = \Sent
      }
      mailbox "Sent Messages" {
        special_use = \Sent
      }
      # 추가할 내용
      mailbox "보낸 메일함" {
        special_use = \Sent
      }

      #mailbox virtual/All {
      #  special_use = \All
      #  comment = All my messages
      #}
      mailbox virtual/All {
        special_use = \All
        comment = All my messages
      }

      (...)
    }

    (...)
    ```

1. `/etc/dovecot/conf.d/20-managesieve.conf` 수정

    ```conf
    (...)

    #protocols = $protocols sieve
    protocols = $protocols sieve

    (...)

    service managesieve-login {
        inet_listener sieve {
            address = localhost
            port = 4190
        }
        (...)
    }

    (...)

    protocol sieve {
        (...)

        #managesieve_max_line_length = 65536
        managesieve_max_line_length = 65536

        (...)
    }

    (...)
    ```

1. `/etc/dovecot/conf.d/90-sieve-extprograms.conf` 수정

    ```conf
    (...)

    # TODO
    #sieve_execute_bin_dir = /usr/lib/dovecot/sieve-execute
    ##################sieve_execute_bin_dir = /usr/lib/dovecot/sieve-execute


    (...)
    ```

1. `/etc/dovecot/conf.d/90-sieve.conf` 수정

    ```conf
    (...)

    plugin {
        (...)

        #sieve_global_extensions =
        sieve_global_extensions = +vnd.dovecot.pipe +vnd.dovecot.execute

        #sieve_plugins =
        sieve_plugins = sieve_extprograms

        (...)

        # 추가
        sieve_global_path   = /etc/dovecot/dovecot.sieve
        sieve_global_dir    = /etc/dovecot/sieve/
    }
    ```

1. `/etc/dovecot/conf.d/auth-system.conf.ext` 수정

    ```conf
    (...)

   passdb {
        driver = pam
        # [session=yes] [setcred=yes] [failure_show_msg=yes] [max_requests=<n>]
        # [cache_key=<key>] [<service name>]
        #args = dovecot
        args = session=yes failure_show_msg=yes max_requests=1 dovecot-auth
    }

    (...)
    ```

1. 데몬 활성화 및 시작

    ```sh
    systemctl daemon-reload
    systemctl enable dovecot
    systemctl start dovecot
    ```

1. dovecot IMAP 접속해보기

    ```sh
    openssl s_client -crlf -connect mail.ryuar.in:993
    01 LOGIN mail "password"
    02 LIST "" *
    03 SELECT INBOX
    04 STATUS INBOX (MESSAGES)
    05 FETCH 1 ALL
    06 LOGOUT
    ```
