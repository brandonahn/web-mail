
1. apt-get 명령어를 이용하여 postfix를 설치한다.

  $ apt-get install postfix


2. postfix를 설정한다.

  $ dpkg-reconfigure postfix

  # 인터넷 사이트
  # 서버는 도메인으로
  # Postfix Configuration : server.com mail.server.com localhost.server.com localhost
  # 동기 업데이트 설정은 원하는 값으로
  # 나머지 그냥 엔터 연타


3. postfix 세부설정

  # 메일 폴더 설정
  $ postconf -e 'home_mailbox = Maildir/'
 
  # procmail 미사용 설정
  $ postconf -e "mailbox_command = "
 
  # SASL을 이용해 SMTP 인증을 사용하기 위하여 설정할 것들
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


4. 인증서 생성이 되었으면 관련 TLS인크립션 사용을 위해 설정을 해주어야 한다.
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


5. SMTP 인증에 관한 설정

  $ vim /etc/postfix/sasl/smtpd.conf
 
  pwcheck_method: saslauthd
 
  mech_list: plain login


6. postfix 데몬 재시작

  $ service postfix reload


7. 다음 작업으로 sasl2를 설치한다.

  $ apt-get install libsasl2-2 libsasl2-modules sasl2-bin


8. saslauthd를 수정한다.

  $ vim /etc/default/saslauthd
 
  # START를 yes로 수정하고 PWDIR, PARAMS, PIDFILE를 추가
  START=yes
  PWDIR="/var/spool/postfix/var/run/saslauthd"
  PARAMS="-m ${PWDIR}"
  PIDFILE="${PWDIR}/saslauthd.pid"
 
  # OPTION 수정
  OPTIONS="-c -m /var/spool/postfix/var/run/saslauthd"


9. saslauthd 업데이트 그리고 실행

  $ dpkg-statoverride --force --update --add root sasl 755 /var/spool/postfix/var/run/saslauthd
 
  $ service saslauthd start


10. IMAP과 POP3를 설치한다.

  $ apt-get install courier-pop courier-imap


11. Mail 디렉토리 생성
  $ mkdir /etc/skel
  $ mkdir /etc/skel/Maildir
  $ maildirmake /etc/skel/Maildir/.Drafts
  $ maildirmake /etc/skel/Maildir/.Sent
  $ maildirmake /etc/skel/Maildir/.Trash
  $ maildirmake /etc/skel/Maildir/.Templates


12. 사용자에 대해 메일 폴더 생성(user는 메일 사용자이며 우분투 계정이다)

  $ cp -r /etc/skel/Maildir /home/myuser/
 
  $ chown -R myuser:usergroup /home/myuser/Maildir
 
  $ chmod -R 700 /home/myuser/Maildir


13. 메일 전송 테스트

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


14. 메일을 로컬 유저에게 발송 했을 경우 확인해본다.
  $ cd /home/user/Maildir/new
  $ ls













