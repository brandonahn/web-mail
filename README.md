Postfix
=======

아직도 이메일은 여전히 중요한 의사 소통 수단 및 협업 도구로 사용되고 있지만 이 외에도 이메일 서버의 용도는
이슈 관리, 지속적인 통합, 버전 관리 시스템등에서 이벤트 발생이나 통지 사항을 사용자에게 알리는 도구로도 사용하고 있다.

 Postfix 소개
 ------------

sendmail 같은 SMTP(Simple Mail Transfer Protocl) 를 구현한 소프트웨어를 MTA(Mail Transfer Agent) 라고 부르며
MS의 아웃룩이나 모질라의 썬더버드, 콘솔에서 구동되는 mutt 등의 사용자 프로그램은 MUA(Mail User Agent) 라고 분류한다. 

sendmail 은 전통적으로 많이 사용되던 MTA 였고 RHEL 5 까지는 기본 메일 서버 데몬이었으나 RHEL 6 부터는 postfix 로 교체되었다.
물론 원하는 사용자는 sendmail 을 MTA 로 사용하는 것도 가능하다.

 

특징
----

postfix 는 IBM의 보안 전문가가 만든 제품으로 sendmail 과 비교해서 다음과 같은 장점이 있다.

## 손쉬운 설정
 sendmail 은 m4 라는 매크로 언어를 사용하여 설정 파일을 생성하는데 예전부터 sendmail 의 설정은 어렵고 가독성이 떨어지기로 악명이 높았다. postfix 는 사용자 친화적이고 손쉬운 설정 방식을 제공한다. 
## 견고한 보안
 sendmail 은 보안이 큰 문제가 되지 않던 시절에 개발되어서 많은 보안 취약점을 갖고 있다. postfix 는 대신 RHEL 버전 6 부터 기본 MTA(Message Transfer Agent) 로 채택된 메일 전송 서버이다.
## 빠른 처리 속도
 빠른 메일 송수신을 염두에 두고 설계/개발되어 sendmail 에 비해 빠른 속도를 자랑한다.


Postfix 설치
------------

#### 1. apt-get 명령어를 이용하여 postfix를 설치한다.

  $ apt-get install postfix


#### 2. postfix를 설정한다.

  $ dpkg-reconfigure postfix

  

#### 3. postfix 세부설정

  #메일 폴더 설정
  $ postconf -e 'home_mailbox = Maildir/'
 
  #procmail 미사용 설정
  $ postconf -e "mailbox_command = "
 
  #SASL을 이용해 SMTP 인증을 사용하기 위하여 설정할 것들
  $ postconf -e 'smtpd_sasl_local_domain ='
 
  $ postconf -e 'smtpd_sasl_auth_enable = yes'
 
  $ postconf -e 'smtpd_sasl_security_options = noanonymous'
 
  $ postconf -e 'broken_sasl_auth_clients = yes'
 
  $ postconf -e 'smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination'
 
  $ postconf -e 'inet_interfaces = all'

  $ openssl genrsa -des3 -rand /etc/hosts -out smtpd.key 1024
 
  $ chmod 600 smtpd.key
 
  $ openssl req -new -key smtpd.key -out smtpd.csr
 
  $ openssl x509 -req -days 3650 -in smtpd.csr -signkey smtpd.key -out smtpd.crt
 
  $ openssl rsa -in smtpd.key -out smtpd.key.unencrypted
 
  $ mv -f smtpd.key.unencrypted smtpd.key
 
  $ openssl req -new -x509 -extensions v3_ca -keyout cakey.pem -out cacert.pem -days 3650
 
  $ mv smtpd.key /etc/ssl/private/
 
  $ mv smtpd.crt /etc/ssl/certs/
 
  $ mv cakey.pem /etc/ssl/private/
 
  $ mv cacert.pem /etc/ssl/certs/


#### 4. 인증서 생성이 되었으면 관련 TLS인크립션 사용을 위해 설정을 해주어야 한다.
  $ postconf -e 'smtpd_tls_auth_only = no'
 
  $ postconf -e 'smtp_use_tls = yes'
 
  $ postconf -e 'smtpd_use_tls = yes'
 
  $ postconf -e 'smtp_tls_note_starttls_offer = yes'
 
  #하단 경로는 자신에게 맞춰 수정
  $ postconf -e 'smtpd_tls_key_file = /etc/ssl/private/smtpd.key'
 
  $ postconf -e 'smtpd_tls_cert_file = /etc/ssl/certs/smtpd.crt'
 
  $ postconf -e 'smtpd_tls_CAfile = /etc/ssl/certs/cacert.pem'
 
  $ postconf -e 'smtpd_tls_loglevel = 1'
 
  $ postconf -e 'smtpd_tls_received_header = yes'
 
  $ postconf -e 'smtpd_tls_session_cache_timeout = 3600s'
 
  $ postconf -e 'tls_random_source = dev:/dev/urandom'
 
  #이 아래 주소는 알아서 수정
  $ postconf -e 'myhostname = server1.example.com'


#### 5. SMTP 인증에 관한 설정

  $ vim /etc/postfix/sasl/smtpd.conf
 
  pwcheck_method: saslauthd
 
  mech_list: plain login


#### 6. postfix 데몬 재시작

  $ service postfix reload


#### 7. 다음 작업으로 sasl2를 설치한다.

  $ apt-get install libsasl2-2 libsasl2-modules sasl2-bin


#### 8. saslauthd를 수정한다.

  $ vim /etc/default/saslauthd
 
  #START를 yes로 수정하고 PWDIR, PARAMS, PIDFILE를 추가
  START=yes
  PWDIR="/var/spool/postfix/var/run/saslauthd"
  PARAMS="-m ${PWDIR}"
  PIDFILE="${PWDIR}/saslauthd.pid"
 
  #OPTION 수정
  OPTIONS="-c -m /var/spool/postfix/var/run/saslauthd"


#### 9. saslauthd 업데이트 그리고 실행

  $ dpkg-statoverride --force --update --add root sasl 755 /var/spool/postfix/var/run/saslauthd
 
  $ service saslauthd start


#### 10. IMAP과 POP3를 설치한다.

  $ apt-get install courier-pop courier-imap


#### 11. Mail 디렉토리 생성
  $ mkdir /etc/skel
  $ mkdir /etc/skel/Maildir
  $ maildirmake /etc/skel/Maildir/.Drafts
  $ maildirmake /etc/skel/Maildir/.Sent
  $ maildirmake /etc/skel/Maildir/.Trash
  $ maildirmake /etc/skel/Maildir/.Templates


#### 12. 사용자에 대해 메일 폴더 생성(user는 메일 사용자이며 우분투 계정이다)

  $ cp -r /etc/skel/Maildir /home/myuser/
 
  $ chown -R myuser:usergroup /home/myuser/Maildir
 
  $ chmod -R 700 /home/myuser/Maildir


#### 13. 메일 전송 테스트

  $ telnet localhost 25
 
    Trying 127.0.0.1...
    Connected to localhost.
    Escape character is '^]'.
    220 mail.comingmedia.com ESMTP Postfix (Ubuntu)
 
    ehlo yourdomain.com					# 타이핑
 
    250-mail.yourdomain.com
    250-PIPELINING
    250-SIZE 10240000
    250-VRFY
    250-ETRN
    250-STARTTLS
    250-AUTH PLAIN LOGIN
    250-AUTH=PLAIN LOGIN
    250-ENHANCEDSTATUSCODES
    250-8BITMIME
    250 DSN
 
    mail from: root@yourdomain.com				# 타이핑
 
    250 2.1.0 Ok
 
    rcpt to: jhanglim@yourdomain.com			# 타이핑
 
    250 2.1.5 Ok
 
    data							# 타이핑
 
    354 End data with .
 
    Subject: My first mail					# 타이핑
 
    Hi,							# 타이핑
    .   (and Enter In a new Line)				# 타이핑
 
    250 2.0.0 Ok: queued as C515B863FC
 
    quit							# 타이핑
    221 2.0.0 Bye
    Connection closed by foreign host.


#### 14. 메일을 로컬 유저에게 발송 했을 경우 확인해본다.
  $ cd /home/user/Maildir/new
  $ ls













