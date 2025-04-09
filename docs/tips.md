# Tips & Tricks

## Linux command lines

``` bash 

# Get external ip
curl -s icanhazip.com

# Get private ip
hostname -I

# Get hostname
hostname


``` 


## Open SSL

Make sure you have OpenSSL installed on your system. You can check by running:

`openssl version`

Generate a Private Key

Run the following command to generate a private key:

```bash 
openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048
```

This command creates a 2048-bit RSA private key and saves it to private_key.pem.

Create a Certificate Signing Request (CSR)


```bash
openssl req -new -key private_key.pem -out request.csr
```

You will be prompted to enter information such as your country, state, organization name, etc.

Generate a Self-Signed Certificate


```bash
openssl x509 -req -days 365 -in request.csr -signkey private_key.pem -out certificate.crt
```

This command creates a self-signed certificate valid for 365 days and saves it to certificate.crt.

Summary of Files Created
`private_key.pem`: Your private key.
`request.csr`: Your Certificate Signing Request.
`certificate.crt`: Your self-signed certificate.


### Verify the Certificate

You can verify the contents of your certificate with:

```bash
openssl x509 -in certificate.crt -text -noout
```
