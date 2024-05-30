# Practice Flashcards

# Kubernetes

## Networking 
### How does kube-proxy work?
Kube-proxy is a network proxy running on each node in a Kubernetes cluster. It manages network rules to allow communication between services and pods. It operates in different modes:

1. **Userspace Mode**: Proxies traffic through the kube-proxy process. It's less efficient and mostly deprecated.
2. **iptables Mode**: Uses iptables rules to handle traffic routing at the kernel level, intercepting and redirecting traffic to service IPs to the appropriate pod IPs.
3. **IPVS Mode**: Uses IP Virtual Server for more efficient load balancing with multiple algorithms, offering better performance for larger clusters.

Kube-proxy watches the Kubernetes API for updates to Service and Endpoint objects and configures the necessary rules to ensure traffic is correctly routed to the service's backend pods.
### What drawbacks occur from having clients connect to NodePorts to access services in a cluster?
Using NodePorts for clients to access services in a Kubernetes cluster has several drawbacks:

1. **Security Risks**: Exposing NodePorts opens specific ports on all nodes, increasing the attack surface of your cluster.
2. **Limited Port Range**: NodePorts are limited to a specific port range (default 30000-32767), which can be restrictive and conflict with other applications.
3. **Load Balancing**: NodePorts lack advanced load balancing features and do not automatically distribute traffic evenly among nodes, potentially leading to uneven load distribution.
4. **Scalability**: Scaling can be challenging as you need to manage port assignments and ensure that all nodes can handle traffic for the services.
5. **Complexity**: Managing NodePorts requires additional configuration and oversight, especially in larger clusters with many services.

These drawbacks can make NodePorts less suitable for production environments compared to other service types like LoadBalancer or Ingress.

### Why do NodePort services not distribute traffic evenly across nodes? 
NodePort services do not distribute traffic evenly across nodes because the client decides which node to connect to, rather than the Kubernetes scheduler. This often leads to an uneven distribution of traffic, as clients may not select nodes uniformly​.

NodePort services not distributing traffic evenly across nodes is related to how clients select nodes to connect to, not to how kube-proxy routes traffic to pods. Once a node receives traffic, kube-proxy on that node can distribute it evenly among the pods backing the service using internal mechanisms like iptables or IPVS.

### If there a service exposed via a nodeport, how would a client be able to connect to it via DNS? 
To connect to a service exposed via a NodePort using DNS, clients can use the internal DNS name provided by Kubernetes. The format for this DNS name is typically:

```
<service-name>.<namespace>.svc.cluster.local
```

When a client queries this DNS name, it resolves to the cluster IP of the service. From there, the service’s NodePort can be used to access the service from outside the cluster. Here’s an example:

1. **Service Name and Namespace**: Suppose your service is named `my-service` in the `default` namespace.
2. **DNS Name**: The DNS name would be `my-service.default.svc.cluster.local`.
3. **NodePort**: Suppose the NodePort assigned is `32000`.

A client outside the cluster would connect to the NodePort on any node's IP address like this:

```
<node-ip>:32000
```

Inside the cluster, the client can connect using:

```
my-service.default.svc.cluster.local:32000
```

This approach leverages Kubernetes’ built-in DNS service to resolve the service name to the appropriate cluster IP, and then uses the NodePort to reach the service.

### How is virtual hosting used to host multiple HTTP sites served via NodePort on a single IP address?
Virtual hosting allows multiple HTTP sites to be hosted on a single IP address by using a load balancer or reverse proxy. This setup directs incoming traffic based on the Host header and URL path in the HTTP requests. The load balancer or reverse proxy accepts connections on common HTTP (80) and HTTPS (443) ports, decodes the HTTP request, and forwards it to the appropriate upstream server - such as in the 'Location' block in nginx. 

### What does minikube tunnel do? 
`minikube tunnel` creates a network route on your host machine to access Kubernetes services of type LoadBalancer. This command enables external IPs for LoadBalancer services, allowing you to reach them directly from your local environment, just as you would in a full Kubernetes cluster​.

### How do you configure DNS entires for a Kubernetes Ingress controller? Can you do it for instances when you have a hostname for you load balancer and when you have just an IP address?  

To configure DNS entries for a Kubernetes Ingress controller, follow these steps:

1. **Identify the External Address**:
   - Determine the external IP address or hostname for your load balancer. This is where the traffic will be directed.

2. **Create DNS Records**:
   - **A Records**: If you have an IP address, create A records.
     - For example, if your domain is `example.com`, create A records for `alpaca.example.com` and `bandicoot.example.com` pointing to the external IP address.
     ```
     alpaca.example.com. IN A <external-ip-address>
     bandicoot.example.com. IN A <external-ip-address>
     ```
   - **CNAME Records**: If you have a hostname, create CNAME records.
     - Map the subdomains to the load balancer's hostname.
     ```
     alpaca.example.com. IN CNAME <load-balancer-hostname>
     bandicoot.example.com. IN CNAME <load-balancer-hostname>
     ```

3. **Update DNS Provider**:
   - Log in to your DNS provider's management console.
   - Add the appropriate A or CNAME records as specified above.

This setup ensures that when a request is made to `alpaca.example.com` or `bandicoot.example.com`, it is directed to your load balancer, which then routes it to the appropriate Kubernetes service based on the hostname.

By configuring these DNS entries correctly, the Ingress controller can effectively manage and route traffic to the desired upstream services based on the incoming request's hostname【8†source】【9†source】.


### How do you map your local environment to the Ingress load balancer when using minikube?
First, run minikube tunnel. This will assign an IP address to your load balancer. Then, update your `/etc/hosts` file, adding an entry `<ip-address> <DNS Entry 1> <DNS Entry 2>`. This will allow you to access services proxied to using those entries via the Kubernetes Ingress Load Balancer.

### Explain the behavior of Ingress and Namespaces.

### What does nginx.ingress.kubernetes.io/rewrite-target: / do ? 
The annotation nginx.ingress.kubernetes.io/rewrite-target: / in an Ingress resource specifies that the path of the incoming request should be rewritten to /. This is commonly used to strip the path prefix from the URL before forwarding the request to the backend service. For example, if the Ingress rule matches /app, a request to /app/page would be rewritten to /page before being sent to the backend service​.

This is similar to the URI in the proxy_pass directive not including a trailing slash - which would include the base path in the redirection to the proxied server.


# SSL, TLS
## Questions on HTTPS / SSL / Certs

1. **What is HTTPS, and how does it differ from HTTP?**

    HTTPS differs from HTTP because HTTPS requires SSL or TLS to encrypt the data exchange, assuring that third parties cannot intercept the data being passed between the client and server, and that the data cannot be tampered with.

    Additionally, HTTPS uses port 443 for communication, while HTTP uses port 80. Also, for a website to use HTTPS, it must receive a security certificate from a Certificate Authority (CA).
    

2. **Describe the role of SSL in securing internet communications.**
3. **What is a Certificate Authority (CA), and what role does it play in the SSL ecosystem?**

    A Certificate Authority issues digital certificates to website owners, that establish a trusted credential for a website, and that a website is who it says it is. CAs have limited to rigorous checks that it makes before issuing a certificate. First, the CA makes sure that the person requesting the certificate actually controls the website. Otherwise, an attacker could send a user trying to access a website the certificate from the CA and public key, and the user would respond with its symmetric key, that the false person can decode with a private key. The false person would then be able to read all encrypted messages. 

    CAs are also responsible for revoking a certificate and its public key if the private key gets leaked. They are considered trust anchors. 

    Certificates sign the SSL/TLS certifactes in a way that enables browsers to confirm their identity. The CAs have both a public key and a private key, with the browsers having the public keys installed. The owner of the website submits a Certificate Signing Request (CSR), when the owner adds the public key, the website identifying information, etc. The CA will then "sign" the certificate with its private key, which the CA keeps hidden. This is hierarchal, but basically, the browser has a public key for the CA, which it can use to confirm that that the Certificate Authority actually signed the SSL certificate with its private key. When it does this, it is able to confirm that the SSL certificate sent by the server is the cert that was verified and signed by the CA, and that the server is who it says it is. 

4. **Explain the process of SSL handshake.**

    An SSL handshake works as follows:
    1. The browser establishes a connection with the server. The browser make a request for the server's SSL certificate. 
    2. The browser assures that the certificate matches the domain name of the site, that it is not expired, and that it has been signed by a certificate authority, verifying that it is accurate, and that the information has not been tampered with.
    3. The browser takes the public key from the SSL server response, chooses and algorithm, and encrypts a generated symmetric key on its own side with public key sent by the server.
    4. The client sends the encrypted symmetric key to the server. The server then decrypts the symmetric key using its private key, which only it has access to. You can only use the public key to encrypt data, not decrypt, so only the browser can decrypt this.
    5. Both the browser and the server switch to using the symmetric key, now that they securely agreed on this key using the public / private key. They begin to transfer data between one another by encrypting the data with the symmetric key. This is much quicker than using public key / private key the whole time. 
    6. At the end of the session, the symmetric key is discarded. Another key will be generated in a given secure session.  

5. **How does SSL encryption ensure data privacy and integrity?**
7. **What is a self-signed certificate, and why is it less trusted than those issued by a CA? And does a self signed certificate contain a signature that can be verified by the browser, and made with a private key?**
    A self-signed certificate is still able to be verified by the browser. The big difference is that the public key used by the browser to confirm that the signature is valid is provided directly in the certificate. 

    The server uses its own private key to put a signature on the SSL certificate. It adds the public key to the SSL cert, allowing anyone to confirm that the server does have the private key for the public key / private key combination. However, while the certificate is valid, and fine for being used in SSL, the valid certificate could have been generated by anyone using its own public / private key combination. There is no trusted authority that confirms that the person who generated the certificate is the owner of the website. 

    While SSL communication is allowed, the lack of external validation makes them more susceptible to man-in-the-middle attacks.
8. **What are the differences between a single-domain, multi-domain, and wildcard SSL certificate?**
    - **Single Domain**: Secure a single qualified domain name or subtomain.
    - **Multi-Domain**: Certificate for securing multiple domain names.
    - **Wildcard**: Would allow for mail.google.com, shop.google.com. 
9. **How can a website owner obtain an SSL certificate?**
10. **Explain the concept of public key infrastructure (PKI) and its relevance to SSL/TLS.**
11. **What is the difference between SSL and TLS?**
12. **How do browsers verify the authenticity of an SSL certificate?**
13. **What steps should an organization take to secure their website with HTTPS?**
14. **Discuss the impact of not securing a website with SSL on SEO and user trust.**
16. **What is the role of cipher suites in SSL/TLS?**
    Cipher suites are the chosen algoithms using in an SSL/TLS handshake for each step, proposed by the client by priority, and chosen by the server. The different steps where a protocol is chosen are:
    - Secret Key Exchange - The algorithm that will be used to encrypt the symmetric key that is excrypted using the public key, and decrypted using the private key. This includes RSA, ECDHE, Diffie-Hellman.
    - Authentication - Verifies the identity of the server. Uses RSA
    - Encryption - How data is encrypted and transmitted between server and client. Algos like AES, ChaCha20 and others are used.
    - MAC - Ensures that the data has not been altered. HMAC is widely used. 
18. **What is RSA?**
    RSA (Rivest-Shamir-Adleman) is one of the first public-key cryptosystems and is widely used for secure data transmission. Invented in 1977 by Ron Rivest, Adi Shamir, and Leonard Adleman, RSA remains significant in the field of cryptography. Here’s a breakdown of what RSA is and how it works:

    1. **Public-Key Cryptography**: Unlike symmetric-key algorithms, which use the same key for both encryption and decryption, RSA uses a pair of keys: a public key for encryption and a private key for decryption. This allows anyone to encrypt data using the public key, but only the holder of the private key can decrypt it.

    2. **Key Generation**:
    - RSA keys are generated through a process that starts with the selection of two large prime numbers. These numbers are then used to produce the public and private keys, which are mathematically linked.
    - The public key consists of the modulus (a large number obtained by multiplying the two chosen primes) and a public exponent. The private key consists of the same modulus and a private exponent.

    3. **Encryption and Decryption**:
    - **Encryption**: To encrypt a message, RSA converts the message into a number smaller than the modulus (usually through a process known as padding) and then raises it to the power of the public exponent modulo the modulus.
    - **Decryption**: The ciphertext is decrypted by raising it to the power of the private exponent modulo the modulus. Due to the mathematical properties of the keys, this reverses the encryption process and yields the original message.

    4. **Digital Signatures**: RSA can also be used for digital signatures. A message is signed by encrypting it (or a hash of it) with a private key. Anyone can verify the signature using the corresponding public key, ensuring the message integrity and origin authenticity.

    5. **Security**: The security of RSA is based on the difficulty of factoring large numbers into their prime components, a problem that currently has no efficient solution for large enough primes. However, the strength of RSA encryption directly depends on the key size, with larger keys providing stronger security.

    6. **Usage**: RSA is used in a variety of applications, including SSL/TLS for securing web traffic, digital signatures, secure remote access, and more. Despite the emergence of new algorithms, especially those based on elliptical curve cryptography (ECC), RSA remains prevalent due to its simplicity and widespread support.

    RSA’s ability to secure confidential data, authenticate the sender, and ensure data integrity through digital signatures makes it a cornerstone of modern cryptographic practices.
17. **What are X.509 Certificates?**
    X.509 certificates serve as a form of digital identity for entities such as websites, individuals, or organizations. They are issued by trusted entities called Certificate Authorities (CAs), which verify the certificate holder's identity before issuing the certificate.

    Components of an X.509 Certificate: An X.509 certificate includes several key pieces of information, such as:
    - Subject: The entity the certificate represents, typically including a name and other identifying information.
    - Issuer: The Certificate Authority that issued the certificate.
    - Serial Number: A unique identifier for the certificate.
    - Validity Period: The date range during which the certificate is considered valid.
    - Public Key: The public key of the entity to which the certificate is issued.
    - Signature: The digital signature of the issuer, verifying the certificate's authenticity.
    - Public Key Infrastructure (PKI): X.509 certificates are a cornerstone of PKI, a system for managing public-key encryption. In a PKI, the CA acts as a trusted third party that issues, revokes, and manages certificates, thereby facilitating secure communication and transaction authentication over networks.
17. **Describe the potential vulnerabilities of SSL/TLS and how they can be mitigated.**
18. **Explain the process of revoking an SSL certificate.**
20. **What future advancements do you foresee in the field of secure web communications?**
21. **If I install kubernetes locally, and you are saying that Kuberentes leverages X.509 certificates and SSL for letting nodes communicate with the master, etc, where are they getting these certs from, and who signs them ? If I install kubernetes and just run it, the master node isn't getting a cert from the CA instantly, so how is this working?**

    When you install Kubernetes locally, such as with Minikube, Kind, or a manual setup, the necessary certificates for secure communication within the cluster are generated automatically as part of the cluster setup process. Here’s how this works and what’s happening behind the scenes regarding X.509 certificates and SSL/TLS encryption:

    1. **Automatic Certificate Generation**: Tools like `kubeadm` (used for bootstrapping Kubernetes clusters) automatically generate the required certificates at the time of cluster initialization. This includes certificates for the Kubernetes API server, the etcd server (Kubernetes' datastore), controller manager, scheduler, and kubelet certificates for each node that will be part of the cluster.

    2. **Self-Signed Certificates**: In most local setups, these automatically generated certificates are self-signed. This means that the Certificate Authority (CA) that signs these certificates is not an external or publicly trusted CA (like Let's Encrypt or VeriSign) but a CA created specifically for your Kubernetes cluster during its setup. This CA then signs all the certificates used within the cluster.

    3. **Certificate Authority (CA) Creation**: The first step in the process involves creating a root CA certificate for your Kubernetes cluster. This CA certificate is then used to sign other certificates required for the various components of the cluster, establishing a chain of trust. The root CA's public key is distributed to all parts of the cluster that need to verify the identity of other components.

    4. **Role of Certificates in the Cluster**: These certificates are used to secure communications between the cluster components. For example, the kubelet on each node uses its certificate to securely communicate with the Kubernetes API server, and the API server presents its certificate to kubelets, kubectl, and other clients, proving its identity.

    5. **Managing and Rotating Certificates**: While tools like `kubeadm` handle the initial generation of certificates, managing the lifecycle of these certificates—such as renewing them before they expire—is an important aspect of cluster administration. Kubernetes provides mechanisms and tools to rotate these certificates automatically or with minimal manual intervention.

    6. **Security Considerations**: Although self-signed certificates can secure communication, they lack the third-party validation provided by certificates signed by publicly trusted CAs. However, for local development environments, testing, and learning purposes, self-signed certificates provide a practical balance between ease of setup and security. For production environments, especially those exposed to the internet, you might consider using certificates from a publicly trusted CA or setting up your own internal CA that follows strict security practices.

    In summary, when you install Kubernetes locally, it creates its own Certificate Authority to generate and sign the necessary certificates for secure internal communication. This process is mostly transparent to the user, ensuring that the cluster is secure by default without requiring manual certificate management for initial setup.
22. **What is a Certificate Signing Request?**

    A CSR (Certificate Signing Request) is a block of encoded text that an applicant submits to a Certificate Authority (CA) to apply for a digital certificate. It contains the public key that will be included in the certificate and information about the applicant, such as the organization name, common name (domain name), locality, and country. The CSR is used by the CA to create and issue a digital certificate that can be used for secure communications.
23. **What commands can be used to create a secret?**
    ```
    openssl genrsa -out mydomain.key 2048

23. **What commands need to be ran on a Mac to add a self-signed certificate to my Mac's keychain?**
    ```
    sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain flashcard.app.crt 
    security find-certificate -c "flashcard.app" (Which is the CN of the cert)
    ```
    To delete:
    ```
    sudo security delete-certificate -c flashcard.app -t /Library/Keychains/System.keychain 
    ```
24. **What is a Subject Alternate Name (SAN), why is it needed, and how to you include this when generating certificates?**
    ```
    openssl req -new -key mydomain.key -out mydomain.csr -addext "subjectAltName = DNS:flashcard.app, DNS:www.flashcard.app"
    openssl req -new -key flashcard.app.key -addext "subjectAltName = DNS:flashcard:app, DNS:www.flashcard.app" -out flashcard.app.csr
    openssl x509 -req -in flashcard.app.csr -signkey flashcard.app.key -out flashcard.app.crt -days 365 -extfile openssl.cnf -extensions v3_req
    ```

## Authentication
1. **What does the provider configuration endpoint do in OIDC?**


<!-- ### 1. ddfdf 
<details>
    <summary>Answer</Summary>
    <p>
        
    </p>
</details> -->
## Webpack
1. **What does npx webpack do?**
2. **What is the difference between npm and npx?**
3. **How can, in webpack, can one dynamically bundle CSS dependencies into a webpack build?**
4. **How do Webpack's built-in Asset Modules work to handle image files, and what effects do they have on how images are referenced and processed in JavaScript, CSS, and HTML files?**
5. **How do you setup a DEV testing server using webpack-dev-middleware?**
6. **What does webpack-dev-middleware do?**
    webpack-dev-middleware is a middleware for a Node.js server that serves Webpack bundles directly from memory, which speeds up the development process by avoiding disk writes. It intercepts HTTP requests and dynamically serves the latest compiled assets, providing real-time updates when source files change. This middleware is typically used with development servers like Express.js, enabling efficient and seamless integration with Webpack, and supports features like Hot Module Replacement (HMR) for instant module updates without full page reloads.
7. **What are the downside of splitting modules using different entries only?**
8. **What strategies can be employed to prevent duplication?**
9. **What does a config with optimization.splitChunks.cacheGroups.vendor do?**
10. **What does "cache_groups" indicate?** 
11. **What is HtmlWebpackPlugin?**