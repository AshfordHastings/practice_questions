## Networking 
### How does kube-proxy work?
<details>
Kube-proxy is a network proxy running on each node in a Kubernetes cluster. It manages network rules to allow communication between services and pods. It operates in different modes:

1. **Userspace Mode**: Proxies traffic through the kube-proxy process. It's less efficient and mostly deprecated.
2. **iptables Mode**: Uses iptables rules to handle traffic routing at the kernel level, intercepting and redirecting traffic to service IPs to the appropriate pod IPs.
3. **IPVS Mode**: Uses IP Virtual Server for more efficient load balancing with multiple algorithms, offering better performance for larger clusters.

Kube-proxy watches the Kubernetes API for updates to Service and Endpoint objects and configures the necessary rules to ensure traffic is correctly routed to the service's backend pods.
</details>

### What drawbacks occur from having clients connect to NodePorts to access services in a cluster?

<details>
Using NodePorts for clients to access services in a Kubernetes cluster has several drawbacks:

1. **Security Risks**: Exposing NodePorts opens specific ports on all nodes, increasing the attack surface of your cluster.
2. **Limited Port Range**: NodePorts are limited to a specific port range (default 30000-32767), which can be restrictive and conflict with other applications.
3. **Load Balancing**: NodePorts lack advanced load balancing features and do not automatically distribute traffic evenly among nodes, potentially leading to uneven load distribution.
4. **Scalability**: Scaling can be challenging as you need to manage port assignments and ensure that all nodes can handle traffic for the services.
5. **Complexity**: Managing NodePorts requires additional configuration and oversight, especially in larger clusters with many services.

These drawbacks can make NodePorts less suitable for production environments compared to other service types like LoadBalancer or Ingress.
</details>

### Why do NodePort services not distribute traffic evenly across nodes? 
<details>
NodePort services do not distribute traffic evenly across nodes because the client decides which node to connect to, rather than the Kubernetes scheduler. This often leads to an uneven distribution of traffic, as clients may not select nodes uniformly​.

NodePort services not distributing traffic evenly across nodes is related to how clients select nodes to connect to, not to how kube-proxy routes traffic to pods. Once a node receives traffic, kube-proxy on that node can distribute it evenly among the pods backing the service using internal mechanisms like iptables or IPVS.
</details>

### If there a service exposed via a nodeport, how would a client be able to connect to it via DNS? 
<details>
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
</details>

### How is virtual hosting used to host multiple HTTP sites served via NodePort on a single IP address?
<details>
Virtual hosting allows multiple HTTP sites to be hosted on a single IP address by using a load balancer or reverse proxy. This setup directs incoming traffic based on the Host header and URL path in the HTTP requests. The load balancer or reverse proxy accepts connections on common HTTP (80) and HTTPS (443) ports, decodes the HTTP request, and forwards it to the appropriate upstream server - such as in the 'Location' block in nginx. 
</details>

### What does minikube tunnel do? 
<details>
`minikube tunnel` creates a network route on your host machine to access Kubernetes services of type LoadBalancer. This command enables external IPs for LoadBalancer services, allowing you to reach them directly from your local environment, just as you would in a full Kubernetes cluster​.
</details>

### How do you configure DNS entires for a Kubernetes Ingress controller? Can you do it for instances when you have a hostname for you load balancer and when you have just an IP address?  

<details>

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

By configuring these DNS entries correctly, the Ingress controller can effectively manage and route traffic to the desired upstream services based on the incoming request's hostname.
</details>

### How do you map your local environment to the Ingress load balancer when using minikube?

<details>

First, run minikube tunnel. This will assign an IP address to your load balancer. Then, update your `/etc/hosts` file, adding an entry `<ip-address> <DNS Entry 1> <DNS Entry 2>`. This will allow you to access services proxied to using those entries via the Kubernetes Ingress Load Balancer.

</details>