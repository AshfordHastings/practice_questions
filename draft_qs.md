## Drafting

# Practice Flashcards


### Explain the behavior of Ingress and Namespaces.

### What does nginx.ingress.kubernetes.io/rewrite-target: / do ? 
The annotation nginx.ingress.kubernetes.io/rewrite-target: / in an Ingress resource specifies that the path of the incoming request should be rewritten to /. This is commonly used to strip the path prefix from the URL before forwarding the request to the backend service. For example, if the Ingress rule matches /app, a request to /app/page would be rewritten to /page before being sent to the backend service​.

This is similar to the URI in the proxy_pass directive not including a trailing slash - which would include the base path in the redirection to the proxied server.



## Authentication
1. **What does the provider configuration endpoint do in OIDC?**


NGINX
1. How do error.log and access.log work in NGINX? 



OTHER
- What is a PID file? 
- What does setting external=True on a Docker network do? 
- What are WebSockets?
- What is the process of a WebSocket connection handshake? 
- What is FastCGI? 
- What is Ray?
- What is the difference between Ray and Spark?
- How does Shopify use Ray within its Merlin project? 
- What is DBT? 
- What is the difference between Apache Fink and Spark?
- what is the nginx annotation mergeable ingress type minion? 
- what is nginx vs nginx plus? 
- What are imagePullSecrets? 
- What is the difference between labels and annotations? 
- What is a service account in Kubernetes? 
- Questions about PathLib.å
- Curl cheatsheet - curl questions.
- Relearn pandas
- What is CORS?
- NFS / SMB
- Learn Jinja templating
- What is Lambda architecture? 
- How Do YAML anchors operate? How can these be referenced?
## Priority Resources
- https://linuxjourney.com/
[nginx-ingress-rewrite-target-problem](https://stackoverflow.com/questions/61185530/another-nginx-ingress-rewrite-target-problem)






