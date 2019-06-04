External Authentication Keys
============================

Leihs supports (at this time exclusively) elliptic curve digital signature
referred as `ES256` on the [JWT](https://jwt.io/) website. 

Corresponding keys can be created with [OpenSSL](https://www.openssl.org/) as in the following example.

Generate a private/public key pair:

    openssl ecparam -name prime256v1 -genkey -noout -out key-pair.pem


Extract the public key: 

    openssl ec -in key-pair.pem -pubout -out public-key.pem
