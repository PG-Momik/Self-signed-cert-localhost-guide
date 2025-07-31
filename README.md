# Local HTTPS Setup with Self-Signed Certificate and Stunnel
> [!NOTE]
> Step by step guide of what steps I take to enable HTTPS on my Laravel local development server using self-signed SSL certificate and stunnel. This guide is written for Ubuntu, but the commands should work similarly on macOS (you can figure out the Mac equivalents).



# Prerequisites

- OpenSSL installed on your system
    - Ubuntu: sudo apt-get install openssl
    - macOS: Usually pre-installed, or brew install openssl
- Stunnel installed
    - Ubuntu: sudo apt-get install stunnel4
    - macOS: brew install stunnel
- Laravel project running locally
- Basic terminal/command line knowledge

---

## Steps

### 1. CD into Laravel project root
```bash
cd /path/to/your/project
```

### 2. Create an SSL folder
```bash
mkdir ssl
cd ssl
```

### 3. Generate Self-Signed SSL Certificate

> **What's happening here?** We're creating a digital certificate that proves our server's identity. In production, you'd get this from a Certificate Authority (CA), but for local development, we create our own.

#### a. Generate a private key
```bash
openssl genrsa -out localhost.key 2048
```
*This creates a 2048-bit RSA private key - think of it as a secret password that only your server knows.*

#### b. Create a Certificate Signing Request (CSR)
```bash
openssl req -new -key localhost.key -out localhost.csr
```
*A CSR contains information about your server and is normally sent to a CA for signing. We'll self-sign it instead.*

> [!NOTE]
> During this step, when asked for "Common Name", enter: `localhost`
> *The Common Name must match the domain you'll access in your browser.*

#### c. Create the self-signed certificate
```bash
openssl x509 -req -days 365 -in localhost.csr -signkey localhost.key -out localhost.crt
```
*This creates the actual certificate file, valid for 365 days, signed with our own private key.*

### 4. Return to project root
```bash
cd ..
```

### 5. Create the stunnel.conf file
```bash
touch stunnel.conf
```

Paste the following into stunnel.conf:
```ini
[https]
accept = 8000
connect = 8001
cert = ssl/localhost.crt
key = ssl/localhost.key
```

> **What stunnel does:** Stunnel acts as a proxy that adds SSL/TLS encryption. It listens on port 8000 (HTTPS) and forwards decrypted traffic to port 8001 where your Laravel app runs (HTTP).

### 6. Serve the Laravel application

#### Terminal 1:
```bash
php artisan serve --port=8001
```
*Starts Laravel on HTTP port 8001*

#### Terminal 2:
```bash
stunnel stunnel.conf
```
*Starts stunnel to handle HTTPS on port 8000*

### 7. Open in Browser
- Visit: https://localhost:8000
- You may see a browser warning due to the self-signed certificate. You can safely bypass this warning for local development.

## .gitignore

> [!ERROR]
> Do NOT commit your local SSL files. Add the following to .gitignore:

### SSL certificates for local development
Inside `.gitignore` file of your project:
```gitignore
ssl/localhost.key
ssl/localhost.csr
ssl/localhost.crt
stunnel.conf
```

Optionally, you can ignore the entire folder:
```gitignore
ssl/*
!ssl/.gitkeep # If you want to track the folder itself
```

---

## References

### Understanding HTTPS and SSL/TLS
- **[How HTTPS Works - Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/Security/Transport_Layer_Security)**
  - Comprehensive guide to understanding SSL/TLS and HTTPS
- **[What is SSL/TLS? - Cloudflare Learning Center](https://www.cloudflare.com/learning/ssl/what-is-ssl/)**
  - Beginner-friendly explanation of SSL certificates and encryption

### OpenSSL Documentation
- **[OpenSSL Command Line Howto](https://www.openssl.org/docs/man1.1.1/man1/)**
  - Official documentation for OpenSSL commands
- **[Self-Signed Certificates Explained](https://www.ssl.com/faqs/what-is-a-self-signed-certificate/)**
  - When and why to use self-signed certificates

### Stunnel Resources
- **[Stunnel Official Documentation](https://www.stunnel.org/docs.html)**
  - Complete stunnel configuration guide
- **[Stunnel Tutorial - DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-ssl-tunnel-using-stunnel-on-ubuntu)**
  - Step-by-step stunnel setup tutorial

### Laravel HTTPS Development
- **[Laravel Documentation - HTTPS](https://laravel.com/docs/10.x/requests#determining-if-a-request-is-secure)**
  - Laravel's built-in HTTPS handling
- **[Laravel Valet](https://laravel.com/docs/10.x/valet)**
  - Alternative tool that provides automatic HTTPS for local Laravel development

### Certificate Authority and PKI Concepts
- **[Public Key Infrastructure (PKI) Explained](https://www.keyfactor.com/blog/what-is-pki/)**
  - Understanding the broader context of digital certificates
- **[Certificate Signing Request (CSR) Guide](https://www.ssl.com/how-to/manually-generate-a-certificate-signing-request-csr-using-openssl/)**
  - Detailed explanation of CSR generation and fields

### Security Considerations
- **[Why Self-Signed Certificates are Dangerous in Production](https://www.thesslstore.com/blog/self-signed-certificate-risks/)**
  - Important security considerations
- **[Let's Encrypt](https://letsencrypt.org/)**
  - Free SSL certificates for production use
