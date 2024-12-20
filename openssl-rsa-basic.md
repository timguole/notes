# OpenSSL rsa basic

Generate an RSA private key file in PEM format:

```shell
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out rsa.key
```

Show content of a private key:

```shell
openssl rsa -in rsa.key -text -noout
```

Generate an RSA public key file in PEM format:

```shell
openssl rsa -in rsa.key -pubout -out rsa.pub
```

Encrypt a short message using public key:

```shell
openssl rsautl -in text.txt -inkey rsa.pub -pubin -encrypt -out text.en
```

Decrypt a message using private key:

```shell
openssl rsautl -in text.en -inkey rsa.key -decrypt -out text.de
```

Sign a message using private key:

```shell
openssl rsautl -in text.txt -sign -inkey rsa.key -out text.sig
```

Verify a signature using public key:

```shell
openssl rsautl -in text.sig -inkey rsa.pub -pubin -out text.unsig
```

