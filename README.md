
1. apt-get ��ɾ �̿��Ͽ� postfix�� ��ġ�Ѵ�.

  $ apt-get install postfix


2. postfix�� �����Ѵ�.

  $ dpkg-reconfigure postfix

  # ���ͳ� ����Ʈ
  # ������ ����������
  # Postfix Configuration : server.com mail.server.com localhost.server.com localhost
  # ���� ������Ʈ ������ ���ϴ� ������
  # ������ �׳� ���� ��Ÿ


3. postfix ���μ���

  # ���� ���� ����
  $ postconf -e 'home_mailbox = Maildir/'
 
  # procmail �̻�� ����
  $ postconf -e "mailbox_command = "
 
  # SASL�� �̿��� SMTP ������ ����ϱ� ���Ͽ� ������ �͵�
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


4. ������ ������ �Ǿ����� ���� TLS��ũ���� ����� ���� ������ ���־�� �Ѵ�.
  $ postconf -e 'smtpd_tls_auth_only = no'
 
  $ postconf -e 'smtp_use_tls = yes'
 
  $ postconf -e 'smtpd_use_tls = yes'
 
  $ postconf -e 'smtp_tls_note_starttls_offer = yes'
 
  #�ϴ� ��δ� �ڽſ��� ���� ����
  $ postconf -e 'smtpd_tls_key_file = /etc/ssl/private/smtpd.key'
 
  $ postconf -e 'smtpd_tls_cert_file = /etc/ssl/certs/smtpd.crt'
 
  $ postconf -e 'smtpd_tls_CAfile = /etc/ssl/certs/cacert.pem'
 
  $ postconf -e 'smtpd_tls_loglevel = 1'
 
  $ postconf -e 'smtpd_tls_received_header = yes'
 
  $ postconf -e 'smtpd_tls_session_cache_timeout = 3600s'
 
  $ postconf -e 'tls_random_source = dev:/dev/urandom'
 
  #�� �Ʒ� �ּҴ� �˾Ƽ� ����
  $ postconf -e 'myhostname = server1.example.com'


5. SMTP ������ ���� ����

  $ vim /etc/postfix/sasl/smtpd.conf
 
  pwcheck_method: saslauthd
 
  mech_list: plain login


6. postfix ���� �����

  $ service postfix reload


7. ���� �۾����� sasl2�� ��ġ�Ѵ�.

  $ apt-get install libsasl2-2 libsasl2-modules sasl2-bin


8. saslauthd�� �����Ѵ�.

  $ vim /etc/default/saslauthd
 
  # START�� yes�� �����ϰ� PWDIR, PARAMS, PIDFILE�� �߰�
  START=yes
  PWDIR="/var/spool/postfix/var/run/saslauthd"
  PARAMS="-m ${PWDIR}"
  PIDFILE="${PWDIR}/saslauthd.pid"
 
  # OPTION ����
  OPTIONS="-c -m /var/spool/postfix/var/run/saslauthd"


9. saslauthd ������Ʈ �׸��� ����

  $ dpkg-statoverride --force --update --add root sasl 755 /var/spool/postfix/var/run/saslauthd
 
  $ service saslauthd start


10. IMAP�� POP3�� ��ġ�Ѵ�.

  $ apt-get install courier-pop courier-imap


11. Mail ���丮 ����
  $ mkdir /etc/skel
  $ mkdir /etc/skel/Maildir
  $ maildirmake /etc/skel/Maildir/.Drafts
  $ maildirmake /etc/skel/Maildir/.Sent
  $ maildirmake /etc/skel/Maildir/.Trash
  $ maildirmake /etc/skel/Maildir/.Templates


12. ����ڿ� ���� ���� ���� ����(user�� ���� ������̸� ����� �����̴�)

  $ cp -r /etc/skel/Maildir /home/myuser/
 
  $ chown -R myuser:usergroup /home/myuser/Maildir
 
  $ chmod -R 700 /home/myuser/Maildir


13. ���� ���� �׽�Ʈ

  $ telnet localhost 25
 
    Trying 127.0.0.1...
    Connected to localhost.
    Escape character is '^]'.
    220 mail.comingmedia.com ESMTP Postfix (Ubuntu)
 
    ehlo yourdomain.com					# Ÿ����
 
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
 
    mail from: root@yourdomain.com				# Ÿ����
 
    250 2.1.0 Ok
 
    rcpt to: jhanglim@yourdomain.com			# Ÿ����
 
    250 2.1.5 Ok
 
    data							# Ÿ����
 
    354 End data with .
 
    Subject: My first mail					# Ÿ����
 
    Hi,							# Ÿ����
    .   (and Enter In a new Line)				# Ÿ����
 
    250 2.0.0 Ok: queued as C515B863FC
 
    quit							# Ÿ����
    221 2.0.0 Bye
    Connection closed by foreign host.


14. ������ ���� �������� �߼� ���� ��� Ȯ���غ���.
  $ cd /home/user/Maildir/new
  $ ls













