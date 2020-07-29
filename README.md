# Easy RSA OCI-compliant Container
## Overview
[Easy RSA](https://github.com/OpenVPN/easy-rsa) is a CLI utility to build and manage a PKI CA. In other words, Easy RSA simplifies your effort to create a root certificate authority, and request and sign certificates, including intermediate CAs and certificate revocation lists (CRL).

The goal of this project is to containerize Easy RSA and make your effort even simpler by providing helper scripts to:

* Setup a CA and Subordinate CA (as you would in a production configuration)
* Generate a server certificate bundle from the Subordinate CA

## Getting started
### Construct var file
For testing purposes, construct a file named `vars` in `test/.easyrsa/config`.

*NOTE: A working example config exists in this path for your convenience. Review and modify it for your needs. You can also spin up the container and fetch the `vars.example` file contents. This file exists in the container path `/easy-rsa/`.*

### Run an Easy-RSA container in interactive TTY mode
`cd` to the project root path.

```
sudo podman run -it --rm --name devtest-example-net-rootca -v $(pwd)/test/.easyrsa:/.easyrsa:Z easyrsa /bin/sh
```


In container:

``./easyrsa --vars="/.easyrsa/config/vars" init-pki``

*expected stdout*
```
### Offline RootCA for devtest environments ###

/easy-rsa # ./easyrsa --vars="/.easyrsa/config/vars" init-pki

Note: using Easy-RSA configuration from: /.easyrsa/config/vars

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /.easyrsa/pki
```

### Create a CA

In container:

``dd if=/dev/urandom of=/.easyrsa/pki/.rnd bs=256 count=1``

``./easyrsa --vars="/.easyrsa/config/vars" build-ca nopass``

*WARNING: This is a very very bad idea! Never ever ever build a CA without a passphrase for Production use. This CA will ONLY be used for evaluation and devtest purposes!*

### Verify the root certificate
In container:

``openssl x509 -noout -text -in /.easyrsa/pki/ca.cert``

### Use EasyRSA Helper scripts
#### Setup your RootCA and SubCA
```
sudo podman run -it --rm --name devtest-example-net-rootca -v $(pwd)/test/.easyrsa:/.easyrsa:Z easyrsa setup-ca
```
*RootCA*
You may accept all defaults except for `Common Name`. Change `Common Name` to something more meaniful that describes the use. For example, `EXAMPLE-DOT-COM-ROOTCA` or `My Test1 RootCA`

*SubCA*
You may accept all defaults except for `Common Name`. Change `Common Name` to something more meaniful that describes the use. For example, `EXAMPLE-DOT-COM-SUBCA` or `My Test1 SubCA`

#### Generate server certificate bundle
```
sudo podman run -it --rm --name devtest-example-net-rootca -v $(pwd)/test/.easyrsa:/.easyrsa:Z easyrsa generate-cert-bundle server minio001.devtest.example.com
```

### Add Trusted CA on your host
copy cert bundle to `/etc/pki/ca-trust/source/anchors`

``update-ca-trust``

*NOTE: If you need to remove trust; (1) cd to `/etc/pki/ca-trust/source/anchors`. (2) rm target *.crt files. (3) `update-ca-trust -f`*

## External References
* https://github.com/OpenVPN/easy-rsa

* https://easy-rsa.readthedocs.io/en/latest/

* [Can't load /usr/share/easy-rsa_github/easy-rsa-3.0.5/easyrsa3/pki/.rnd into RNG
140199901626816:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:88:Filename=/usr/share/easy-rsa_github/easy-rsa-3.0.5/easyrsa3/pki/.rnd](https://github.com/OpenVPN/easy-rsa/issues/261)

* https://jamielinux.com/docs/openssl-certificate-authority/sign-server-and-client-certificates.html