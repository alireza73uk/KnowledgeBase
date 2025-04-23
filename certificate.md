# Create CSR

create config file `myserver.cnf`:

```conf

[ req ]
default_bits = 2048
default_md = sha256
prompt = no
encrypt_key = no
distinguished_name = dn
req_extensions = req_ext

[ dn ]
C = CA
O = My Org
CN = example.com

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = *.example.com
DNS.2 = ipv6.example.com
DNS.3 = ipv4.example.com
DNS.4 = test.example.com
DNS.5 = party.example.com

```

```sh
openssl req -new -config myserver.cnf -keyout myserver.key -out myserver.csr
```
