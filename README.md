# LearnPKI
Summarized learning experience of X.509 and Public Key Infrastructure (PKI) 
This is not intended for any production standards, but to help with learning the architecture of how PKI is applied in the process. 

Zero-trust architecture has become the accelerated standard for protecting infrastructure and everything around it. PKI is the centralized component playing a significant role in trust for how reuquest are made and how that information is encrypted > decrypted > encrypted, to provide secure connections and detections of common attacks like malware trying to infiltrate applications. 


## What is PKI ?  

PKI uses a combination of public key cryptography and digital certificates to ensure the confidentiality, integrity, and authenticity of electronic communications and transactions.


## What is X.509 ?

A certificate in X.509 format contains information about the subject of the certificate (e.g., the name of an individual or organization), the public key, and the digital signature of a trusted certificate authority (CA) attesting to the validity of the information.


The X.509 standard defines the format of public key certificates, including the syntax for representing names and public keys, the format of digital signatures, and the procedures for certificate issuance and management. X.509 certificates are used to securely bind a public key to a subject’s identity, and they are widely used for secure communication over the internet and for secure file transfer protocols such as Secure File Transfer Protocol (SFTP) and Secure Shell (SSH).

X.509 defines a standard format for digital certificates, which are used to authenticate the identity of a device or individual and to establish trust in secure communication.

## **Common use cases for how X.509 certificates are used in production for secure communication.**


SSL/TLS encryption X.509 certificates are used to establish secure connections between web browsers and servers using the SSL (Secure Sockets Layer) or TLS (Transport Layer Security) protocols. When a browser connects to a secure website, it verifies the server’s identity using its X.509 certificate.

**Email encryption** X.509 certificates can be used to encrypt and sign email messages, providing a secure way to exchange confidential information.

**Virtual Private Networks** (VPNs) use X.509 certificates to establish secure connections between remote users and a corporate network. The certificates are used to authenticate the identities of the remote users and to ensure the confidentiality and integrity of the data being transmitted.

**Code signing** Software developers can use X.509 certificates to sign their code, providing a way for users to verify the authenticity of the code and to establish trust in the software.

**For a Zero-trust example:** we can refer to an architecture reference from [Microsoft Azure Zero-trust network for web applications with Azure Firewall and Application Gateway - Azure Architecture Center | Microsoft Learn](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/gateway/application-gateway-before-azure-firewall)

In the Zero-trust reference, the Azure application gateway with Azure Web Application Firewall utilizes TLS to encrypt traffic and ensure multiple steps before passing information to the intended applications on the backend. 

**How does it work**

The SSL/TLS layer 7 load balancer operates by decrypting incoming SSL/TLS traffic, inspecting the contents of the decrypted packets to determine the destination of the traffic, and then forwarding the traffic to the appropriate backend server.

The load balancer acts as a reverse proxy and terminates the SSL/TLS connection from the client. It then establishes a new, separate SSL/TLS connection with the backend server, forwarding the decrypted traffic to the backend. This allows the load balancer to make load balancing decisions based on the contents of the decrypted packets, such as the HTTP request headers and the URL path.

The load balancer can use various algorithms, such as round-robin, least connections, and IP hash, to determine the destination backend server for each incoming request. It can also perform health checks on the backend servers to ensure they are available to handle traffic before forwarding requests to them.

By using an SSL/TLS layer 7 load balancer, organizations can offload SSL/TLS processing from their backend servers and improve the performance, reliability, and security of their web applications.

However, layer 4 load balancing has some limitations, such as the inability to inspect the contents of the payload to make more informed load balancing decisions, and the inability to support advanced features such as sticky sessions, health checks, and SSL/TLS offloading. This is the benefit of payloads being inspected by solutions like azure Application Gateway, which handles the inspection of the contents before securely handing off the payload to the intended application which processes the request, that request usually reaches a Firewall that process the information securely before it reaches the backend services. 

### **Getting started with a summary on OpenSSL** 

## OpenSSL is an open-source implementation of the Secure Sockets Layer (SSL) and Transport Layer Security (TLS) protocols


**Encryption** -  supports various encryption algorithms, such as AES, DES, RSA, and DSA, and allows for the creation of secure SSL/TLS connections.

**Digital Certificates** - provides tools for generating and managing digital certificates, which are used to authenticate the identity of a party in an SSL/TLS connection.

**Key Management** - provides tools for managing encryption keys, including key generation, key exchange, and key storage.

One book I highly recommend is [“Bullet Proof TLS and PKI” by Ivan Ristić](https://www.feistyduck.com/books/bulletproof-tls-and-pki/)



Getting started with OpenSSL X.509 commands,

here’s some OpenSSL X.509 commands to understand how it works. 

```openssl req -new -key private.key -out csr.pem```

creates a new X.509 certificate signing request (CSR) using the private key file “private.key”. The resulting CSR is stored in the file “csr.pem”.

```openssl verify -CAfile root.pem cert.pem```

This command verifies the authenticity of an X.509 certificate using the root certificate file “root.pem”. The certificate file to be verified is specified using the “cert.pem” argument.

```openssl x509 -req -in csr.pem -signkey private.key -out cert.pem```

signs a CSR file using the private key file “private.key”. The resulting X.509 certificate is stored in the file “cert.pem”.


**This script generates a root CA private key, a root CA certificate, a server private key, and a server certificate. The server certificate is signed by the root CA, which acts as a trusted third-party that verifies the identity of the server**


```
#!/bin/bash

# Define CA and server names
CA_NAME=myrootca
SERVER_NAME=myserver

# Create a root CA
openssl genpkey -algorithm RSA -out $CA_NAME.key
openssl req -key $CA_NAME.key -new -x509 -out $CA_NAME.crt -subj "/CN=$CA_NAME"

# Create a server key and certificate request
openssl genpkey -algorithm RSA -out $SERVER_NAME.key
openssl req -key $SERVER_NAME.key -new -out $SERVER_NAME.csr -subj "/CN=$SERVER_NAME"

# Sign the server certificate with the root CA
openssl x509 -req -in $SERVER_NAME.csr -CA $CA_NAME.crt -CAkey $CA_NAME.key -CAcreateserial -out $SERVER_NAME.crt
```


Generating a  key pair

```openssl genpkey [options]```

**-algorithm** - specifies the algorithm to use for key generation

**-out** - Specifies the file name to there the generated key pair will be written 

**-Aes256** Specifies that the generated key should be encrypted using AES-256 encryption. 

How to generate a 2048-bit RSA key pair

```openssl genpkey -algorithm RSA -out private_key.pem -aes256```

Once you have generated the key pair, you can use the private key to sign messages and the public key to verify signatures.

To extract the public key from the private key, you can use the following command

```openssl rsa -in private_key.pem -pubout -out public_key.pem```

This will write the public key to the file public_key.pem.

**AES** is a symmetric encryption algorithm, meaning that the same key is used for both encryption and decryption. AES is widely used for encrypting sensitive data, such as financial transactions and government communications, due to its high security and speed.

**RSA** is an asymmetric encryption algorithm, meaning that it uses two keys, a public key for encryption and a private key for decryption. RSA is commonly used for secure data transmission and digital signatures.

Bulletproof TLS Newsletter provides resourceful information on the latest developments on SSL/TLS Bulletproof TLS Newsletter | Feisty Duck  get started with your PKI journey with one of the solutions below from multiple cloud providers


**PKI solutions across cloud providers** 

**Azure** - Azure Key Vault is a cloud-based key management service that enables you to store, manage, and control access to cryptographic keys, certificates, and secrets. You can use Azure Key Vault to create and manage digital certificates, including X.509 certificates, that are used for authentication, secure communication, and secure storage.

**Azure Certificate Services**  - is a cloud-based PKI service that enables you to issue, manage, and renew X.509 certificates. Azure Certificate Services provides a highly available and scalable certificate authority (CA) infrastructure, eliminating the need to manage and maintain a CA infrastructure on-premises.

**AWS Certificate Manager** (ACM) is a service that enables you to easily provision, manage, and deploy secure SSL/TLS certificates for use with AWS services. ACM provides a user-friendly interface for creating, importing, and managing X.509 digital certificates, making it easy to secure your website and applications.

**AWS Key Management Service** (KMS) is a service that enables you to create and manage encryption keys that are used to encrypt your data. KMS integrates with other AWS services, such as Amazon S3 and Amazon RDS, to provide a secure key management solution for your applications.

**Google Cloud Key Management Service** (KMS) is a service that enables you to create and manage encryption keys that are used to encrypt your data. KMS integrates with other Google Cloud services, such as Google Cloud Storage and Google Cloud SQL, to provide a secure key management solution for your applications.
