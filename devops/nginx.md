# NGINX Questions

### How do you set an owner for NGINX worker processes? 
<details>
To set an owner for NGINX worker processes, use the `user` directive in the NGINX configuration file (`nginx.conf`):

```nginx
user username groupname;
```

- **`username`** specifies the user that the worker processes will run as.
- **`groupname`** specifies the group that the worker processes will run under.

For example:

```nginx
user www-data www-data;
```

This configuration should be placed in the `main` context, typically at the beginning of the `nginx.conf` file.
</details>

### What headers are required in order to tell a browser to stash static content? 
<details>

To instruct a browser to cache static content, you should set the following HTTP headers:

1. **`Cache-Control:`** Specifies caching directives.
   ```http
   Cache-Control: max-age=31536000, public
   ```
   - `max-age=31536000` indicates the content should be cached for one year.
   - `public` makes the response cacheable by any cache.

2. **`Expires:`** Provides an expiration date and time.
   ```http
   Expires: Tue, 01 Jun 2025 00:00:00 GMT
   ```
   - This header specifies a fixed date/time after which the response is considered stale.

3. **`ETag:`** Provides a unique identifier for the version of the resource.
   ```http
   ETag: "unique-identifier"
   ```
   - Helps in validating if the cached content matches the current content on the server.

4. **`Last-Modified:`** Indicates the last time the resource was modified.
   ```http
   Last-Modified: Tue, 01 Jun 2024 00:00:00 GMT
   ```
   - Useful for conditional requests and cache validation.

#### Example NGINX Configuration
```nginx
location /static/ {
    expires 1y;
    add_header Cache-Control "public";
    add_header Last-Modified $date_gmt;
    add_header ETag "unique-identifier";
}
```

These headers collectively help manage and control the caching behavior of static content in the browser.
</details>

### How do virtual hosts operate in nginx? 
<details>
In NGINX, virtual hosts operate through the use of `server` blocks within the configuration file. Each `server` block defines a separate virtual host, handling different domain names or IP addresses.

#### Key Elements:
- **`server_name`**: Specifies the domain names that the server block will respond to.
- **`listen`**: Defines the port and optionally the IP address for the server block.
- **`location`**: Defines how requests should be processed and where to find resources.

#### Example Configuration:
```nginx
http {
    server {
        listen 80;
        server_name example.com www.example.com;
        
        location / {
            root /var/www/example;
            index index.html;
        }
    }

    server {
        listen 80;
        server_name another-example.com www.another-example.com;
        
        location / {
            root /var/www/another-example;
            index index.html;
        }
    }
}
```

#### How it Works:
- Requests are routed to the appropriate `server` block based on the `Host` header in the HTTP request.
- Each virtual host can have its own root directory, SSL certificates, and configuration settings.
</details>



### What does "upstream" do in NGINX? 
<details>

In NGINX, the `upstream` directive defines a group of backend servers that can be used to handle client requests. It is commonly used for load balancing, distributing the incoming traffic across multiple servers.

#### Example Configuration:
```nginx
http {
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
    }

    server {
        listen 80;
        server_name example.com;

        location / {
            proxy_pass http://backend;
        }
    }
}
```

#### #How it Works:
- **`upstream` block:** Defines a named group of servers (`backend` in this case).
- **`server` directive within `upstream`:** Specifies each backend server.
- **`proxy_pass` directive:** Forwards client requests to the defined upstream group.

#### Benefits:
- **Load Balancing:** Distributes load across multiple servers.
- **Failover:** Provides redundancy in case one or more backend servers fail.
- **Scalability:** Easily scale the application by adding more backend servers.

The `upstream` directive is essential for creating a scalable and resilient architecture in NGINX.
</details>

### What does "curl resolve" do? 
<details>

The `curl --resolve` option allows you to manually specify the IP address for a given hostname and port. This is useful for testing and debugging DNS resolution or directing traffic to a specific server without changing the DNS configuration.

#### Syntax:
```sh
curl --resolve example.com:80:192.168.1.1 http://example.com
```

#### Explanation:
- **`example.com:80:192.168.1.1`**: Maps `example.com` on port `80` to the IP address `192.168.1.1`.
- **`http://example.com`**: The URL to make the request to.

This overrides DNS resolution for the specified host and port, directing the request to the given IP address.
</details>

### What is the difference between `sites-avaliable` and `sites-enabled`?
<details>
In NGINX, `sites-available` and `sites-enabled` are directories used for managing virtual host configurations.

#### `sites-available`
- **Purpose:** Stores all available virtual host configuration files.
- **Location:** `/etc/nginx/sites-available/` (on Debian-based systems).
- **Usage:** Contains configurations that define how different domains should be handled. These files are not active until they are linked to `sites-enabled`.

#### `sites-enabled`
- **Purpose:** Contains symbolic links to the configuration files in `sites-available` that are currently enabled.
- **Location:** `/etc/nginx/sites-enabled/` (on Debian-based systems).
- **Usage:** NGINX reads configurations from this directory to determine which virtual hosts are active.

#### How It Works
1. **Create a Configuration:** Add a virtual host file in `sites-available`.
   ```sh
   sudo nano /etc/nginx/sites-available/example.com
   ```

2. **Enable the Site:** Create a symbolic link in `sites-enabled`.
   ```sh
   sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
   ```

3. **Reload NGINX:** Apply the changes.
   ```sh
   sudo systemctl reload nginx
   ```

#### Summary
- **`sites-available`:** Holds all potential virtual host configurations.
- **`sites-enabled`:** Holds symbolic links to active virtual host configurations.

This structure helps in managing and organizing virtual hosts efficiently.
</details>

### What is the difference between a Tomcat server and a nginx server?
<details>

#### Tomcat Server:
- **Type:** Application Server.
- **Purpose:** Designed to run Java Servlets and JavaServer Pages (JSP).
- **Features:** Provides a Java runtime environment, supports Java EE specifications, and handles dynamic web content.
- **Usage:** Often used for deploying Java web applications.

#### NGINX Server:
- **Type:** Web Server and Reverse Proxy.
- **Purpose:** Serves static content, acts as a load balancer, reverse proxy, and HTTP cache.
- **Features:** High performance, low resource consumption, supports multiple protocols (HTTP, HTTPS, SMTP, etc.), efficient load balancing.
- **Usage:** Commonly used for serving static files, reverse proxying, and load balancing web traffic.

#### Summary:
- **Tomcat:** Focuses on running Java-based web applications.
- **NGINX:** Focuses on serving static content, reverse proxying, and load balancing.
</details>
