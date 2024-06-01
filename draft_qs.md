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





