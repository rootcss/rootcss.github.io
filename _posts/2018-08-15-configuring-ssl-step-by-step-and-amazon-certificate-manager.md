---
layout: post
title: Configuring SSL step by step (and Amazon Certificate Manager)
tags:
- SSL
- AWS
---

Configuring SSL for your domains is still not as simple as it can be. Everytime I do that, I need to refer to my previous notes. Recently, I was using <a href="https://aws.amazon.com/certificate-manager/" target="_blank">`AWS Certificate Manager`</a> to setup a PositiveSSL Wildcard certificate, so I thought of putting up my notes on the blog.

Note: This post focuses on configuring SSL, and very less on details about what & why.


<b>Step 1: Generate Certificate Signing Request (CSR) and Private key</b>


```bash
openssl req -new -newkey rsa:2048 -nodes -keyout mydomain.key -out mydomain.csr
```

Enter the details like country, state etc when asked. After this step you will have two files:

- mydomain.csr (the CSR file)
- mydomain.key (the private key)

Content of these files look like following:

mydomain.csr
```
-----BEGIN CERTIFICATE REQUEST-----
:
-----END CERTIFICATE REQUEST-----
```

mydomain.key
```
-----BEGIN PRIVATE KEY-----
:
-----END PRIVATE KEY-----
```

<br>
<b>Step 2: Buying the certificate from providers</b>

When requesting for your SSL certificate on the providers like namecheap, godaddy etc, you'll be asked to enter your CSR content. Once you complete all the steps, you'll receive following files from the provider (I'm taking the example of files from Namecheap where I purchased the  certificate)

1. AddTrustExternalCARoot.crt [Root CA Certificate]
2. COMODORSAAddTrustCA.crt [Intermediate CA Certificate]
3. COMODORSADomainValidationSecureServerCA.crt [Intermediate CA Certificate]
4. STAR_mydomain.crt [Your PositiveSSL Certificate]

<br>
<b>Step 3: Creating SSL Bundle</b>

Using the files mentioned in Step 2, we'll be creating a SSL bundle, which is very simple. Just concatenate the content of first three files in right order as mentioned in the command:

```bash
cat COMODORSADomainValidationSecureServerCA.crt COMODORSAAddTrustCA.crt AddTrustExternalCARoot.crt > ssl-bundle.crt
```

If you want to configure it for NGINX, then you need to concatenate your PositiveSSL certificate as well.
```bash
cat STAR_mydomain.crt COMODORSADomainValidationSecureServerCA.crt COMODORSAAddTrustCA.crt AddTrustExternalCARoot.crt > ssl-bundle.crt
```

Content of the bundle file will look something like this: (for ACM, only three entries will be there)

```text
-----BEGIN CERTIFICATE-----
: - STAR_mydomain.crt
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
: - COMODORSADomainValidationSecureServerCA.crt
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
: - COMODORSAAddTrustCA.crt
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
: - AddTrustExternalCARoot.crt
-----END CERTIFICATE-----
```

Note: 
- The order of above files is very important.
- There shouldn't be empty lines or line break between the certificates.

<br>
<b>Step 4: Configuring on NGINX</b>

If you wanted to configure on NGINX, you just need these two files:
- ssl-bundle.crt
- mydomain.key

In you NGINX configuration, point these parameters to right path, restart NGINX and you're good to go:
```
ssl_certificate /etc/nginx/ssl/mydomain/ssl-bundle.crt;
ssl_certificate_key /etc/nginx/ssl/mydomain/mydomain.key;
```

<br>
<b>Step 5: Configuring on Amazon Certificate Manager (ACM)</b>

To configure it with ACM, you need to go through couple of more steps, as they required the certificates to be in certain format.
First convert your private key to .pem format:
```bash
openssl rsa -in mydomain.key -text > mydomain-private-key.pem
```

Content:
```
-----BEGIN RSA PRIVATE KEY-----
:
-----END RSA PRIVATE KEY-----
```

At last, to upload these certificates on ACM, use the following command:
```bash
aws acm import-certificate \
	--certificate file:///Users/rootcss/Downloads/ssl/STAR_mydomain.crt \
	--private-key file:///Users/rootcss/Downloads/ssl/mydomain-private-key.pem \
	--certificate-chain file:///Users/rootcss/Downloads/ssl/ssl-bundle.crt
```

<br>
<b>Other notes:</b>

1. If your private key is of format .pfx, you can convert it to .pem format directly using the command:
```bash
openssl pkcs12 -in mydomain-private-key.pfx -out mydomain-private-key.pem -nodes
```
(Enter the created password when asked)

2. If you need to convert your CSR file into PEM format, you can use the command:
```bash
openssl x509 -inform PEM -in ssl-bundle.crt > ssl-bundle.crt
```

<br>
<b>References:</b>
- https://aws.amazon.com/certificate-manager/
- http://nginx.org/en/docs/http/configuring_https_servers.html
- https://stackoverflow.com/questions/991758/how-to-get-pem-file-from-key-and-crt-files
