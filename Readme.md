# TLS showcase

This repository contains the basic principles behind the TLS protocol along with the steps to create the needed certificates for the parties to communicate securely.

The Transport Layer Security (TLS) protocol, is meant to provide a secure, authenticated, private and integrous communication between 2 parties, through the use of various cryptographic algorithms. 

To understand more on how this works please visit this blogpost: TBA

The following instructions are for Osx
## Certificate generation - mkcert

Follow the commands bellow line by line. The comments should guide you through creating a Certificate Authority(CA) and generate signed certificate using the mkcert tool.

```bash
# Install mkcert and verify it was installed correctly
brew install mkcert
mkcert --version

# Install the openssl utilities
brew install openssl

#  This will generate a local CA
# You should have locally a pem file representing the rootCA and its private key
# Please note that this command will also ensure the CA is trusted by your system
mkcert -install

# Prints the parent folder of the root CA.
mkcert -CAROOT

# Generate signed certificates for a localhost server
mkcert 127.0.0.1 localhost ::1

# Process the generated key with the rsa command. 
# The initially generated format is not right for our tokio_rustls server
openssl rsa -in 127.0.0.1+2-key.pem   -out 127.0.0.1+2-rsa-key.pem
rm 127.0.0.1+2-key.pem
```

## Certificate generation using openssl only
If you are trying to go a bit deeper, you can generate and sign the certificates yourself using openssl only. Follow along with the commands below.

```bash
# generate the root CA that will be used to sign the SSL certificates
openssl req -x509 \
            -sha256 -days 356 \
            -nodes \
            -newkey rsa:2048 \
            -subj "/CN=127.0.0.1/C=US/L=San Fransisco" \
            -keyout rootCA.key -out rootCA.crt

# create the server private key
openssl genrsa -out server.key 2048

# Create a file called csr.conf with the contents below representing
# the config for the csr (certificate signing request)

# begining of csr file 
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = US
ST = California
L = San Fransisco
O = MLopsHub
OU = MlopsHub Dev
CN = localhost

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = localhost
DNS.2 = localhost
IP.1 = 192.168.1.5
IP.2 = 192.168.1.6

# End of file

# Generate the actual CSR based on the config above
openssl req -new -key server.key -out server.csr -config csr.conf

# Similar to the csr we will now create the config file for the server certificate.
# Name it cert.conf

# begining of the cert.conf file

authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost

# End of cert.conf file

# Generate the server certificate signed by the CA, based on the csr and the certificate config.
# Before running this last command please make sure to add the CA.crt file to the osxkeychain
# (drag and drop works) and then make sure to trust it (double click on it and set the trust options).
openssl x509 -req \
    -in server.csr \
    -CA rootCA.crt -CAkey rootCA.key \
    -CAcreateserial -out server.crt \
    -days 365 \
    -sha256 -extfile cert.conf
```



## Starting the server and client

```bash
cargo run --bin server 127.0.0.1:8000 --cert 127.0.0.1+2.pem   --key 127.0.0.1+2-rsa-key.pem

# Please not that this assumes the $CA_DIR variable is already set. If you don't set it,
# please make sure to pass the correct filepath on your system. 
# You can retrieve it with the mkcert command explained above.
cargo run --bin client -- 127.0.0.1 -p 8000 -d localhost -c $CA_DIR/rootCA.pem
```





