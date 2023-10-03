# 테스트

1. 메일박스 확인을 위한 mutt 설치

    ```sh
    pacman -Sy mutt
    ```

1. root에서 mail 유저의 Mailbox 쉽게 열기 위한 설정

    ```sh
    ln -s /var/spool/mail/Maildir /var/spool/root
    ln -s /var/spool/mail/Maildir ~/Mail
    ```

1. Gmail, 네이버에서 `admin@ryuar.in` 으로 메일 전송하고 수신 확인

    ```sh
    mutt
    # exit with q
    ```

1. SMTP 설정

    |Name|Value|
    |:-|:-:|
    |Host|`mail.ryuar.in`|
    |Port|`992`|
    |Protocol|SSL/TLS|
    |Login type|Plain Password|
    |ID|`mail`|
    |Password|`...`|

1. 다양한 메일 테스트 서비스를 이용해보세요

    - [mail-tester](https://www.mail-tester.com/)
    - [m@ilgenius](https://www.mailgenius.com/)
