#Let's Encrypt Pragmatic

The [Let's Encrypt](https://letsencrypt.org/) initiative is a fantastic program
that is going to offer **free** https certificates! This is an alternative to
their command program to get a free certificate.

You generate your private key and certificate signing request (CSR) like
normal, then run `sign_csr.py` with your CSR to get it signed. The script goes
through the [ACME protocol](https://github.com/letsencrypt/acme-spec) with the
Let's Encrypt certificate authority and outputs the signed certificate to
stdout.

Different from the original letsencrypt-nosudo it

* it is meant to run on a different machine (probably your pc) than the server serving the domain
* does not require sudo to start a python home grown webserver because:
 * you probably already have a https running
 * the script does /not/ run on the server
* it does not require you to issue commands manuall (instead it can fire these commands itself)

#How to

##Do Once

* install openssl and python2 if not available
* create an user account key if not already done
```sh
openssl genrsa 4096 > user.key
openssl rsa -in user.key -pubout > user.pub
```
* create the domain key
```sh
openssl genrsa 4096 > example.com.key
```
* a certificate request if not already done
```sh
openssl req -new -sha256 -key example.com.key -subj "/CN=example.com" > example.com.csr
# or for multiple domains:
openssl req -new -sha256 -key example.com.key -subj "/" -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:example.com,DNS:www.example.com")) > example.com.csr
```
* get the intermediate cert for the chain
```sh
wget https://letsencrypt.org/certs/lets-encrypt-x1-cross-signed.pem
```
* make sure .well-known/acme-challenge is served by the httpd, the script puts a file there which is retreived by letsencrypt

##Loop

* you run the script using python and passing in the path to your user account public key and the domain CSR. The paths can be relative or absolute.
```sh
python sign_csr.py \
	--email=webmaster@example.com \
	--private-key user.key \
	--ssh-host sample.com \
	--docroot /var/www/sample.com/ \
	--public-key user.pub \
	sample.com.csr \
	> sample.com.crt
```
* chain the cert
```sh
cat sample.com.crt lets-encrypt-x1-cross-signed.pem > sample.com.chained.pem
```
* copy the cert to the server
* restart httpd

##Insides

The functionality of the original script was mostly kept intact. instead
additional parameters were added which, if given trigger the automatism to use
the private key or copy to the server via ssh. If you do not provide these
informations the script still spits out command to issue to progress to the
next step.

##Donate

If this script is useful to you, please donate to the EFF. I don't work there,
but they do fantastic work.

[https://eff.org/donate/](https://eff.org/donate/)

