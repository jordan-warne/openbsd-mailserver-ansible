# openbsd-mailserver

Setup mail server using ansible, based on the wonderful guide by TuM'Fatig (Joel Carnat) https://www.tumfatig.net/2023/self-hosted-email-services-on-openbsd/

## Requirements

- Ansible >= 2.7
- Server with OpenBSD (tested working as of OpenBSD 7.4)
- SSH key to login to the server
- Public domain resolving to the server (MX, AAAA and A)

## 

## Configuration

- Dovecot
- OpenSMTPD
- RspamD
- acme-client
- httpd

### Setup your DNS!

```
A       @       123.123.123.123
```
```
AAAA    @       0000:0000:0000:000:00
```
```
A       mail    123.123.123.123
```
```
AAAA    mail       0000:0000:0000:000:00
```
```
A       mta-sts    123.123.123.123
```
```
AAAA    mta-sts       0000:0000:0000:000:00
```
```
MX      @       mail.example.com
```

#### SPF domain rules

```
TXT     @       v=spf1 mx:example.com -all ~all
```

#### DMARC domain policy

```
TXT     _dmarc       v=DMARC1; p=reject; adkim=s; aspf=s;
```

#### MTA-STS and TLSRPT policies

You can use the current date as the mta-sts ID

```
TXT     _mta-sts        v=STSv1; id=20190811231231
```

Make sure to edit the reporting address to a address you control

```
TXT     _smtp._tls      v=TLSRPTv1; rua=mailto:tls-reports@example.com
```

### Inventory

Set the hostname in the inventory file [hosts](hosts)

### Variables

All variables are stored in the [vars](group_vars/all/vars.yml) file. Set the values according to your server.

## Run the playbook

```
ansible-playbook install.yml
```

## Set your DKIM DNS 

The playbook will generate the DKIM DNS, check this file for what to enter in your DNS

```
/etc/mail/dkim-dns-example.txt
```

Don't skip this step unless you want to go to the spam folder.

## Setup DANE email security

Make sure to enable DNSSEC at your domain provider!

First you need to create a TLSA record, you can use this tool to generate one:

https://www.huque.com/bin/gen_tlsa

Use the public key from your domain to generate the TLSA record.
Usage field: DANE-EE
Selector field: SPKI
Matching type field: SHA-256: SHA-256 hash
Port: 25
Transport protocol: tcp
Domain name: (your mail server fqdn)

## Check rspamd reporting

You can use the rspamd web GUI to view spam:

```
# ssh -L 11334:localhost:11334 <mail server>
```

Browse to http://localhost:11334/

## Contributions

Contributions welcome

## Credits

Check out these articles and tutorials that I used to create this Ansible-Playbook

- https://www.tumfatig.net/2023/self-hosted-email-services-on-openbsd/
- https://github.com/mubn/ansible-mail-server
- https://nicolascarpi.github.io/openbsd/2019/04/03/openbsd-mail-server.html
- https://man.openbsd.org/smtpd.conf 
