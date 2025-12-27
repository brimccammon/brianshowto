---
layout: post
title: Create a CA
---

I was tired of having all these invalid or untrusted certs on my network applications and devices so I decided to load up a CentOS box with OpenSSL and make my own Certificate Authority (CA).  See the steps after the jump.I was tired of having all these invalid or untrusted certs on my network applications and devices so I decided to load up a CentOS box with OpenSSL and make my own Certificate Authority (CA).  Here is how I did it.

Steps:

1. Change directory to /etc/pki/tls
2. Edit the openssl.cnf and make the following changes.
    1. Change HOME to  from . to /etc/pki/tls
    2. Under [CA_Defautls] change dir from ../../CA to ../CA
    3. Under [ policy_match ] change all the ones that are set to match to supplied
    4. Under [ req_distinguished_name ] change all entries to the correct ones for the location on the CA
3. Create the index.txt file
````
touch /etc/pki/CA/index.txt
````
4. Create the serial file
````
echo ’01’ > /etc/pki/CA/serial
````
5. Generate the CA Cert and Key
````
openssl req -new -x509 -keyout private/cakey.pem -out cacert.pem -days 3650
````
6. Generate the crl
````
openssl ca -gencrl -out crl.pem
````

Once that is done the server is ready to create certs from requests.  I do this by placing the cert request in a directory accessible by both the CA and the requester.  Once the request has been generated and is accessable by the CA run the following command to generate the cert.

````
openssl ca -in /mnt/cert-request.txt  -out /mnt/server.cer -days 365
````

This will create the cert server.cer (cer is used for IIS, if a different extension is needed then replace cer with what is desired) and it will be good for 1 year.  Once the cert is generated it is ready to be applied to the requester.

NOTE:  To have CRL work correctly the ca.crl file needs to be published to the web.  Usually it is at server.domain.com/crl.pem.  Otherwise depending on the browser if the ca.crl file is not accessible an error will show with the cert.
