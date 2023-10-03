# 개인 메일 서버 구축

- 이 문서는 개인용 메일 서버 구축 과정을 기록한 문서입니다.

- 구글, 네이버 등 기업 메일과 송수신이 가능한 것을 목적으로 합니다.

    - 이 외의 추가적으로 스팸 필터, Brute-Force 밴 등을 추가 세팅합니다.

- 2014년부터 사용되던 설정을 일부 변경한 것으로, 최적의 설정이 아닙니다.

## 서버 환경

- Vultr VPS

    - Cloud Compute : AMD High Performance
    - 1 Core CPU
    - 1 GB RAM
    - 1 TB Bandwidth
    - 25GB NVMe

- OS : ArchLinux

    ```sh
    # uname -a
    Linux mail-proxy 6.1.53-1-lts #1 SMP PREEMPT_DYNAMIC Wed, 13 Sep 2023 09:32:00 +0000 x86_64 GNU/Linux
    ```

- 연결할 도메인 : `ryuanerin.kr` `ryuaner.in` `ryuar.in`

- 메일 서버 주소 : `mail.ryuar.in`

- 메일 서버에서 모든 메일을 한 유저로 받도록 설정
    - username: `mail`

- 내부 네트워크 IP : `10.10.0.0/24`

## 메일 서버 구축 과정

1. [환경 설정](00-prepair.md)

1. [SPF 설정](01-spf.md)

1. [acme.sh 이용하여 인증서 발급](10-acme.sh.md)

1. [Dovecot 설정](20-dovecot.md)

1. postfix 서버 설정

    1. [OpenDMARC](30-opendmarc.md)
    1. [OpenDKIM](31-opendkim.md)
    1. [SpamAssassin : 스팸 필터 설정](32-spamassassin.md)
    1. [Amavis : 스팸 필터 설정](38-amavisd.md)
    1. [Postfix : 메일 서버 설정](39-postfix.md)

1. [Fail2Ban : 봇 차단](40-fail2ban.md)

1. [테스트](50-test.md)

1. 변경 사항

    1. [2023-10-04](90-2023-10-04.md)
