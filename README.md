This repo is obsolete since Let's Encrypt is now a supported feature

https://community.sophos.com/kb/en-us/132940


# sophos-utm-letsencrypt

## Backup!
Before you start make a backup of your configuration in case something goes wrong or the wrong certificate is overwritten.

## Install Verification CA Certificate

Download the let's encrypt intermediate certificate
https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem

Install it as Verification CA via webadmin (Webserver Protection -> Certificate Management -> Certificate Authority -> New CA...)

I have named it "Let’s Encrypt Authority X3", but you can give it any name you want.

This certificate is then served by the Web Application Firewall when a Let's Encrypt certificate is used to complete the certificate chain and provide better acceptance of the Let's Encrypt certificate.


## Get all Files

SSH to you Sophos UTM and then download everything needed:

```
sudo su -
cd /root
wget https://raw.githubusercontent.com/srvrco/getssl/master/getssl --no-check-certificate
wget https://raw.githubusercontent.com/rklomp/sophos-utm-letsencrypt/master/update-cert --no-check-certificate
wget http://web.mit.edu/crypto/openssl.cnf
chmod +x getssl
chmod +x update-cert
```

## Find the certificate reference

Use an existing certificate entry to overwrite, or generate a new one in the webgui (Webserver Protection -> Certificate Management -> Add Certificate...). Using the VPN ID type "Hostname". Assign this certificate to your virtual webserver.

Get certificate Reference using confd-client:

```
#/usr/local/bin/confd-client.plx
127.0.0.1 MAIN > OBJS
Switched to OBJS mode.
127.0.0.1 OBJS > ca
127.0.0.1 OBJS ca > host_key_cert
127.0.0.1 OBJS ca host_key_cert >
```

pres tab twice to show the list of all certificates

from the list find the certificate you want to write the Let's Encrypt certificate in.
Copy everything upto the [ sign. We will use this reference in the next step

For example
REF_CaHosLetsEncryp[Let's Encrypt,ca,host_key_cert] 

The reference to use is: REF_CaHosLetsEncryp

## Create config

Create default config files. If you get a curl error, make sure your firewall is not blocking outbound (IPv6) traffic.

`./getssl -c yourdomain.com`

Edit the config files

`vi ~/.getssl/getssl.cfg`

Set ACCOUNT_EMAIL and SSLCONF
```
ACCOUNT_EMAIL=<your email>
SSLCONF="/root/openssl.cnf"
```

Edit the domain specific config file

`vi ~/.getssl/yourdomain.com/getssl.cfg`

Set RELOAD_CMD; use your domain and the reference you looked up earlier

`RELOAD_CMD="/root/update-cert yourdomain.com REF_CaHosLetsEncryp"`


Set ACL; The directory where to copy acme challenge file to. This should be the server that is serving the yourdomain.com webpages. Also create the folder on the server and test if http://yourdomain.com/.well-known/acme-challenge/ is reachable and if you can ssh from the UTM to the server. Maybe you need to add a firewall rulle to allow traffic.

`ACL=('ssh:<user>@<server>:/var/www/.well-known/acme-challenge')`


Note: If you use SSH you need to create a ssh-key using `ssh-keygen` and copy it to your server `ssh-copy-id <user>@<server>`

Tip: If you cannot copy the file to the server serving this domain you can copy it to another server and use Site Path Routing for path /.well-known/acme-challenge/ to this other server.

Tip2: Using FTP is also possible, see the example in the config file, but an FTP excutable is not available by default on the Sophos UTM.

Finally comment out or edit the SANS parameter, it could contain some additional (unwanted) domains. All domains should be resolvable from the outside and have a line in ACL. So for example if the SANS in the yourdomain.com config is set to `SANS=sub.yourdomain.com` the ACL shoud contain two lines, one for the server serving yourdomain.com and one for the server serving sub.yourdomain.com.

```
ACL=('ssh:<user>@<server1>:/var/www/.well-known/acme-challenge'
'ssh:<user>@<server2>:/var/www/.well-known/acme-challenge')
```


## Test it!
Testing time...

`./getssl -f yourdomain.com`

If everything works correct and your website now uses the new certificate you can continue. If not.. solve it ;)

Note: **This is a test certificate that is not a valid signed certificate. The certificate is issued by "fake LE intermediate x1". The next step will make sure you will get a valid signed certificate.**

Tip: getssl supports the -d parameter to show debug output

## Finish your work

`vi ~/.getssl/yourdomain.com/getssl.cfg`

Uncomment:

`CA="https://acme-v01.api.letsencrypt.org"`

Test again:

`./getssl -f yourdomain.com`

Now you should have a valid certificate!

Make cronjob:

`crontab -e`

add a line to run daily:

`33 0 * * * /root/getssl yourdomain.com`

(please use random minute instead of 33)
