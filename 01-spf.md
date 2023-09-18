# SPF

- 스푸핑 및 스팸 방지를 위해서 SPF(Sender Policy Framework) 설정

- SPF에 대한 자세한 내용은 [여기](https://support.google.com/a/answer/33786) 참조.

- DNS 설정

    | Name            | Type | Data                           |
    |----------------:|:----:|--------------------------------|
    |      `ryuar.in` | TXT  | `v=spf1 a mx -all`             |
    |    `ryuaner.in` | TXT  | `v=spf1 include:ryuar.in -all` |
    | `rryuanerin.kr` | TXT  | `v=spf1 include:ryuar.in -all` |

1. `postfix-policyd-spf-perl` 설치

    ```sh
    cd /tmp
    git clone https://aur.archlinux.org/postfix-policyd-spf-perl.git
    chown nobody:nobody -R /tmp/postfix-policyd-spf-perl
    cd /tmp/postfix-policyd-spf-perl
    sudo -u nobody makepkg --skippgpcheck
    pacman --noconfirm -U postfix-policyd-spf-perl-2.011-1-x86_64.pkg.tar.zst
    rm -rf /tmp/postfix-policyd-spf-perl/
    ```
