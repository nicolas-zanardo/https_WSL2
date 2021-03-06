# https_WSL2

```bash
$ mkdir certs
$ cd certs
$ openssl genrsa -des3 -out myCA.key 2048
$ openssl req -x509 -new -nodes -key myCA.key -sha256 -days 1825 -out myCA.pem
$ mkdir /mnt/c/Users/YOUR_USER/.localhost-ssl/
$ cp myCA.pem /mnt/c/Users/YOUR_USER/.localhost-ssl/myCA.pem
```

Go to Windows 10: "Manager computer certificates"

Select: "Trusted Root Certification Authorities"

import your myCA.pem

```bash
$ cd /mnt/c/Users/YOUR_USER/.localhost-ssl/
vi ssl.sh
```

Copy and Past the code:

```bash
#!/bin/bash

if [ "$#" -ne 1 ]
then
echo "You must supply a domain... And windows user name"
exit 1
fi

DOMAIN=$1
YOUR_USER=$2

cd ~/certs
openssl genrsa -out /mnt/c/Users/$YOUR_USER/.localhost-ssl/$DOMAIN.key 2048
openssl req -new -key /mnt/c/Users/$YOUR_USER/.localhost-ssl/$DOMAIN.key -out /mnt/c/Users/$YOUR_USER/.localhost-ssl/$DOMAIN.csr

cat > /mnt/c/Users/$YOUR_USER/.localhost-ssl/$DOMAIN.ext << EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = $DOMAIN
EOF

openssl x509 -req -in /mnt/c/Users/$YOUR_USER/.localhost-ssl/$DOMAIN.csr -CA myCA.pem -CAkey myCA.key -CAcreateserial \
-out /mnt/c/Users/$YOUR_USER/.localhost-ssl/$DOMAIN.crt -days 1825 -sha256 -extfile /mnt/c/Users/$YOUR_USER/.localhost-ssl/$DOMAIN.ext

rm /mnt/c/Users/$YOUR_USER/.localhost-ssl/$DOMAIN.csr
rm /mnt/c/Users/$YOUR_USER/.localhost-ssl/$DOMAIN.ext
```

```bash
chmod u+x ssl.sh 
sudo mv ssl.sh /usr/local/bin/ssl.sh

ssl.sh YOUR_DOMAINE YOUR_USER
``` 

Config Nginx:

```bash
# ADD in your Virtualhost

ssl_certificate /mnt/c/User/YOUR_USER/.localhost-ssl/YOUR_DOMAINE.crt;
ssl_certificate_key /mnt/c/User/YOUR_USER/.localhost-ssl/YOUR_DOMAINE.key;

```

