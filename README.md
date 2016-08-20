# sophos-utm-letsencrypt

## Get all Files

SSH to you Sophos UTM first

```
sudo su -
cd /root
wget https://raw.githubusercontent.com/rklomp/getssl/master/getssl --no-check-certificate
wget https://raw.githubusercontent.com/rklomp/sophos-utm-letsencrypt/master/update-cert --no-check-certificate
chmod +x getssl
chmod +x update-cert
```

get a default openssl config

for example:
`wget http://web.mit.edu/crypto/openssl.cnf`

## Find the certificate reference

Get certificate Reference

```
#cc
127.0.0.1 MAIN > OBJS
Switched to OBJS mode.
127.0.0.1 OBJS > ca
127.0.0.1 OBJS ca > host_key_cert
127.0.0.1 OBJS ca host_key_cert >
```

pres tab twice

from the list find the certificate you want to write the Let's Encrypt certificate in.
Copy everything upto the [ sign. We will use this reference in the next step

For example
REF_CaHosLetsEncryp[Let's Encrypt,ca,host_key_cert] 

The reference to use is: REF_CaHosLetsEncryp

## Create config

Create default config files
`./getssl -c yourdomain.com`

Edit the config files

`vi ~/.getssl/getssl.cfg`

set
```
ACCOUNT_EMAIL=<your email>
SSLCONF="/root/openssl.cnf"
```

`vi ~/.getssl/yourdomain.com/getssl.cfg`

set ACL; where to copy acme challenge to
`ACL=('ssh:<user>@<server>:/var/www/.well-known/acme-challenge')`

set RELOAD_CMD; use your domain and the reference you looked up earlier
`RELOAD_CMD="/root/update-cert yourdomain.com REF_CaHosLetsEncryp"`

# Test it!
Test 
`./getssl -f yourdomain.com`

If everything works correct and your website now uses the new certificate you can continue. If not.. solve it ;)

# Finish your work

`vi ~/.getssl/yourdomain.com/getssl.cfg`

Uncomment:
`CA="https://acme-v01.api.letsencrypt.org"`

Test again:
`./getssl -f yourdomain.com`

Make cronjob:

`crontab -e`

add a line to run daily:

`0 0 * * * /root/getssl yourdomain.com`

