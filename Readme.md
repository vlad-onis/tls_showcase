# TLS showcase

This repository contains the basic principles behind the TLS protocol along with the steps to create the needed certificates for the parties to communicate securely.

The Transport Layer Security (TLS) protocol, is meant to provide a secure, authenticated, private and integrous communication between 2 parties, through the use of various cryptographic algorithms. 

To understand more on how this works please visit this blogpost: TBA

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

## Starting the server and client

```bash
cargo run --bin server 127.0.0.1:8000 --cert 127.0.0.1+2.pem   --key 127.0.0.1+2-rsa-key.pem

# Please not that this assumes the $CA_DIR variable is already set. If you don't set it,
# please make sure to pass the correct filepath on your system. 
# You can retrieve it with the mkcert command explained above.
cargo run --bin client -- 127.0.0.1 -p 8000 -d localhost -c $CA_DIR/rootCA.pem
```





