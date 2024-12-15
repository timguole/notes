# EASAY-RSA Basics

## Installation

```shell
sudo apt install easy-rsa
```

## Usage

### Create a CA

```shell
mkdir mypki-ca
cd mypki-ca
cp -rp /usr/share/easy-rsa/* ./
./easy-rsa init-pki
mv openssl-easyrsa.cnf pki/
./easy-rsa build-ca # this will create ca.crt in ./pki/ and key in ./pki/private/
```

### Create a server cert signing request

```shell
mkdir mypki-server
cd mypki-server
cp -rp /usr/share/easy-rsa/* ./
./easy-rsa init-pki
mv openssl-easyrsa.cnf pki/
./easy-rsa gen-req myserver # this will create request in ./pki/reqs and key in ./pki/private/
```

### Sign the server cert (switch back to pki-ca dir and do the following steps)

```shell
cd ../pki-ca
./easy-rsa import-req myserver ../pki-server/pki/reqs/myserver.req
./easy-rsa sign-req myserver # this will create the signed server cert in ./pki/issued/
```

At this point, we have four files:

- CA certificate
- CA key
- self-signed server certificate
- server key

## What's next?

Now, we can configure server programs (nginx, httpd, openvpn) to use the server certificate and server key. Optionally, we can install the CA certificate on server OS and client OS.
