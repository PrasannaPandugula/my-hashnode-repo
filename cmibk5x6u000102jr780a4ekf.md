---
title: "Day 9: TLS Certificates"
seoTitle: "Transport Layer Security, Symmetric and Asymmetric, Real World Exm"
datePublished: Sun Nov 23 2025 10:11:28 GMT+0000 (Coordinated Universal Time)
cuid: cmibk5x6u000102jr780a4ekf
slug: day-9-tls-certificates
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1763877386766/11ad37fe-cf8a-4c2b-af63-c9f4f0f4f184.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1763892655698/04eafe5b-4913-47c6-81c3-d3705b505f34.png
tags: k8s-series

---

## TLS Basics

### What is a Certificate?

* Certificates enable a secure handshake between a user and a web server by verifying identities and encrypting communication.
    

## Understanding Encryption with Real-World Scenario

**Scenario**: User tries to access the online bank application

1. Without secure connectivity, users were to access online back application credentials they typed in and send as ‘plain text’ format to the web server, if a hacker sniffing n/w traffic could easily retrieve the credentials.
    
    ### **Symmetric Encryption:**
    
2. Encrypt data using keys and send the request to web server, suppose hacker sniffing the network and has a copy of user data, the server can’t decrypt data without key we must pass the key.
    
    \-imagine ‘key‘ passed to webserver through the same network, even the key exposed to hackers.
    
    ```plaintext
    username = sdghy
    password = nmhyklospyg
    ```
    
3. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1763885432624/e2feb669-ee2b-4d7d-8b80-39ffa752b940.png align="center")
    

**Note:** Here we are using the same key to encrypt and decrypt data.

### Asymmetric Encryption

\- It uses public and private key to encrypt and decrypt data, be conscious about using keys.

Generate Keys:

```bash
$ ssh-keygen
id_rse #private key
id_rse.pub   # public key
```

How to use these keys in our scenario:

1. Admins Lock **web server** with a public key (usually done by adding entry into the server), you see lock is public; anyone can break through if they have the private key, so protect your private key.
    

```bash
# on web server do ssh to keys file
$ cat ~/.ssh/autorized_keys 
```

2. Using the same public key you can lock as many servers as you want and with same private key we can SSH into all the servers securely.
    
3. How other users can get access to servers, they generate their own public and private keys and actual owner can copy the newly created public keys into server file so others can access using there private keys.
    

### Lets use Asymmetric Encryption in our scenario:

1. Generate keys at server side.
    
    ```bash
    $ openssl genrsa -out my-bank.key 2040 # my-bank.key
    $ openssl rsa -in my-bank.key -pubout > mybank.pem # mybank.key mybank.pem
    ```
    
2. when use access the web server using https, he gets the public key from server, lets assume hacker get public key too.
    
3. user (browser) encrypts the symmetric key provided by the public key from the server, symmetric now secured.
    
4. User/browser sends the key to the server, hacker also get a copy.
    
5. server use private key to decrypt the symmetric key and use it, how ever hacker does not have private key to decrypt the message.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1763890300783/b8c4b7e3-77f8-4a20-a32e-4aa5d00e2ae1.png align="center")
    
6. A hacker could only get the credentials if the user typed them into a fake login page the attacker controls, created by mimicking the real app and redirecting the user to it.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1763890531635/f1305473-e678-4742-9369-9526a96ebcd4.png align="center")

7.When the server sends public key, it also sends the certificate with a signature, if hacker generates fake certificate browser identifies and warns you.

8.How do we get authority sign on certificate ? That’s where CA(Certificate Authority) comes in.

9.Popular CA’s \[Symantec, digicert, GlobalSign\].

10.How to request CSR (Certificate Signing Request)

```bash
$ openssl req -new -key my-bank -out my-bank.csr -subj "/C=US/ST=CA/o=MyOrg, Inc./CN=my-bank.com"
```

11. CA verify and sign the cert and send to you.
    
12. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1763891417234/24d09b19-0c51-4266-a786-2670f6e89fbd.png align="center")
    
    How CA’s process work?, Certificate Authorities will have there own public and private keys, they use private key to sign cert and send public keys to browser for validation, this process for public apps which we are using day to day life.
    
13. Application in our organization and private in this case we host private CA’s (most of teh companies office private services). install public key on employee browsers to access app securely.
    
14. How server validate client ? (client=browser), server can request cert from client, so client must generate cert and send to CA and sever for validation this process is TLS certificate implemented under the hood, if you notice when we access any app from browser we won’t be generating any certs.
    

Note: Public and private keys are paired, we can only use any one for encryption and other for decryption.

15. Identify public and private keys with extensions.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1763892468043/a59307e2-25d1-4f19-896f-c6a7251ad5f4.png align="left")