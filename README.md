# OpenSSL "startdate" and "enddate" patch for self-signed certificates

Do you need to use OpenSSL to create self-signed server certificates
with specific start and end dates? If so, you've probably discovered
that you need to create a full CA certificate to accomplish this.
While this isn't prohibitively difficult, it does require several
steps and some contortions to avoid creating the framework for a CA
or interfering with an existing CA.

The most painless way I've found to do this is to use Bash with
process substitution to read the OpenSSL configurations. This avoids
any dependency on any existing OpenSSL configuration file (usually
named `openssl.cnf`).

The following procedure uses the bare minimum commands and configuration
file to create a self-signed certificate with a start date of
January 1, 2021 and an end date of December 31, 2021:

```
openssl genrsa -out key.pem 4096

openssl req -new -key key.pem -out csr.pem \
    -subj "/C=US/ST=CA/L=San Francisco/O=Example Corp./CN=example.com"

touch index.txt

openssl ca -batch -selfsign -md sha256 -rand_serial \
    -in csr.pem -out cert.pem -keyfile key.pem \
    -startdate 20210101000000Z -enddate 20211231235959Z \
    -config <(echo '
[ ca ]
default_ca = CA_default

[ CA_default ]
database = ./index.txt
new_certs_dir = .
rand_serial = yes
policy = policy_any

[ policy_any ]
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = optional
emailAddress            = optional
')
```

`openssl ca` requires that the values `default_ca`, `database`,
`new_certs_dir`, `rand_serial`, and `policy` exist. If this
looks tedious and unsightly, it is!

With this OpenSSL patch, you can generate self-signed certificates
with start and end dates using this considerably less onerous method:
```
openssl req -x509 -newkey rsa:4096 -sha256 \
    -keyout key.pem -out cert.pem \
    -startdate 20210101000000Z -enddate 20211231235959Z \
    -subj "/C=US/ST=CA/L=San Francisco/O=Example Corp./CN=example.com" \
    -config <(echo '
[ req ]
distinguished_name = req
')
```
Note that even though there's a dependency on the config file, it's
only one value (`distinguished_name`), and also no extraneous files
are created that require cleanup.
