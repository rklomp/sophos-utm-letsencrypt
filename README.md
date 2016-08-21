# sophos-utm-letsencrypt

## Get all Files

SSH to you Sophos UTM first and then download everything needed:

```
sudo su -
cd /root
wget https://raw.githubusercontent.com/rklomp/getssl/master/getssl --no-check-certificate
wget https://raw.githubusercontent.com/rklomp/sophos-utm-letsencrypt/master/update-cert --no-check-certificate
wget http://web.mit.edu/crypto/openssl.cnf
chmod +x getssl
chmod +x update-cert
```

## Find the certificate reference

Get certificate Reference using cc:

```
#cc
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

`vi ~/.getssl/yourdomain.com/getssl.cfg`

Set ACL; The directory where to copy acme challenge file to. This should be the server that is serving the yourdomain.com webpages. Also create the folder on the server and test if http://yourdomain.com/.well-known/acme-challenge/ is reachable.

`ACL=('ssh:<user>@<server>:/var/www/.well-known/acme-challenge')`

Tip: If you cannot copy the file to the server serving this domain you can copy it to another server and use Site Path Routing for path /.well-known/acme-challenge/ to this other server.

Tip2: If you use SSH you need to create a ssh-key using `ssh-keygen` and copy it to your server `ssh-copy-id <user>@<server>`

Tip3: Using FTP is also possible, see the example in the config file.

Set RELOAD_CMD; use your domain and the reference you looked up earlier

`RELOAD_CMD="/root/update-cert yourdomain.com REF_CaHosLetsEncryp"`

Finally comment out or edit the SANS parameter, it could contain some additional (unwanted) domains. All domains should be resolvable from the outside and have a line in ACL. So for example if the SANS in the yourdomain.com config is set to `SANS=sub.yourdomain.com` the ACL shoud contain two lines, one for the server serving yourdomain.com and one for the server serving sub.yourdomain.com.

```
ACL=('ssh:<user>@<server1>:/var/www/.well-known/acme-challenge'
'ssh:<user>@<server2>:/var/www/.well-known/acme-challenge')
```


## Test it!
Testing time...

`./getssl -f yourdomain.com`

If everything works correct and your website now uses the new certificate you can continue. If not.. solve it ;)

Note: This is a test certificate that is not a valid signed certificate.

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

`0 0 * * * /root/getssl yourdomain.com`

