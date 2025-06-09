# NGINX: A Comprehensive Guide

*Last updated: June 7, 2025*

## Table of Contents
- [Introduction to NGINX](#introduction-to-nginx)
- [Installation](#installation)
- [File Structure](#file-structure)
- [Configuration Basics](#configuration-basics)
  - [Configuration Syntax](#configuration-syntax)
  - [Context Hierarchy](#context-hierarchy)
  - [Directives](#directives)
- [Server Blocks (Virtual Hosts)](#server-blocks-virtual-hosts)
- [Location Blocks](#location-blocks)
- [Reverse Proxy](#reverse-proxy)
- [Load Balancing](#load-balancing)
  - [Load Balancing Methods](#load-balancing-methods)
  - [Health Checks](#health-checks)
  - [Session Persistence](#session-persistence)
- [SSL/TLS Configuration](#ssltls-configuration)
  - [SSL Certificate Setup](#ssl-certificate-setup)
  - [SSL Optimization](#ssl-optimization)
- [HTTP/2 and HTTP/3](#http2-and-http3)
- [Security Best Practices](#security-best-practices)
  - [Headers Security](#headers-security)
  - [Rate Limiting](#rate-limiting)
  - [Access Control](#access-control)
  - [Content Security Policy](#content-security-policy)
  - [WAF Integration](#waf-integration)
- [Caching](#caching)
- [Logging and Monitoring](#logging-and-monitoring)
- [Optimization Techniques](#optimization-techniques)
- [Common Use Cases with Sample Configurations](#common-use-cases-with-sample-configurations)
- [Troubleshooting](#troubleshooting)

## Introduction to NGINX

NGINX (pronounced "engine-x") is a high-performance web server, reverse proxy, load balancer, and HTTP cache. Originally created to solve the C10K problem (handling 10,000+ concurrent connections), NGINX uses an asynchronous, event-driven architecture that enables it to handle high loads with minimal resource consumption.

**Key features:**

- **Web Server**: Serves static content efficiently and can be configured to serve dynamic content via FastCGI, WSGI, etc.
- **Reverse Proxy**: Acts as an intermediary between clients and backend servers
- **Load Balancer**: Distributes traffic across multiple servers
- **HTTP Cache**: Caches responses to reduce load on backend servers
- **SSL Termination**: Handles SSL/TLS encryption
- **Media Streaming**: Supports streaming of media files
- **Microservices Gateway**: Routes requests to appropriate microservices

**NGINX vs. NGINX Plus:**

| Feature | NGINX Open Source | NGINX Plus |
|---------|------------------|------------|
| Core functionality | ✅ | ✅ |
| Advanced load balancing | Basic | Advanced with session persistence |
| Health checks | Passive | Active and passive |
| Authentication | Basic | Advanced, JWT, etc. |
| API Dashboard | ❌ | ✅ |
| Commercial support | ❌ | ✅ |
| Dynamic reconfiguration | ❌ | ✅ |

## Installation

### On Debian/Ubuntu:

```bash
# Update package lists
sudo apt update

# Install NGINX
sudo apt install nginx

# Start NGINX
sudo systemctl start nginx

# Enable NGINX to start on boot
sudo systemctl enable nginx

# Check status
sudo systemctl status nginx
```

### On RHEL/CentOS:

```bash
# Add NGINX repository
sudo tee /etc/yum.repos.d/nginx.repo <<EOF
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/rhel/\$releasever/\$basearch/
gpgcheck=0
enabled=1
EOF

# Install NGINX
sudo yum install nginx

# Start and enable NGINX
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Docker:

```bash
# Pull the NGINX image
docker pull nginx

# Run NGINX container
docker run -d --name nginx-server -p 80:80 -p 443:443 nginx
```

## File Structure

NGINX follows a logical file organization structure:

```
/etc/nginx/
├── nginx.conf                # Main configuration file
├── mime.types                # Maps file extensions to MIME types
├── conf.d/                   # Directory for additional configs
│   └── default.conf          # Default server block
├── sites-available/          # Available site configurations (Debian/Ubuntu)
│   └── default
├── sites-enabled/            # Enabled site configurations (Debian/Ubuntu)
│   └── default -> ../sites-available/default
├── modules-available/        # Available modules
├── modules-enabled/          # Enabled modules
└── snippets/                 # Reusable configuration snippets
    ├── fastcgi-php.conf
    └── ssl-params.conf
```

**Important paths:**

- **Configuration files**: `/etc/nginx/`
- **Log files**: `/var/log/nginx/` (access.log, error.log)
- **Default web root**: `/var/www/html/` or `/usr/share/nginx/html/`
- **Executable**: `/usr/sbin/nginx`
- **PID file**: `/var/run/nginx.pid`

## Configuration Basics

### Configuration Syntax

NGINX configuration uses a simple, declarative syntax with the following elements:

1. **Directives**: Simple directives consist of a name and parameters separated by spaces and ending with a semicolon (`;`)
2. **Blocks**: Complex directives have `{` and `}` and contain additional directives
3. **Comments**: Start with `#`

Example:
```nginx
# This is a comment
server {                # Block start
    listen 80;          # Directive
    server_name example.com;
    
    location / {        # Nested block
        root /var/www/html;
        index index.html;
    }
}                      # Block end
```

### Context Hierarchy

NGINX configuration has a hierarchical structure with nested contexts:

1. **Main Context**: Global configuration
2. **Events Context**: Connection processing settings
3. **HTTP Context**: HTTP server settings
4. **Server Context**: Virtual host settings
5. **Location Context**: URI-specific settings
6. **Upstream Context**: Defines server groups for load balancing

```nginx
# Main context
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Events context
events {
    worker_connections 1024;
}

# HTTP context
http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Server context
    server {
        listen 80;
        server_name example.com;
        
        # Location context
        location / {
            root /var/www/html;
            index index.html;
        }
    }
    
    # Upstream context
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
    }
}
```

### Directives

Directives control NGINX's behavior. Here are some important ones with detailed explanations:

**Core directives:**
- `worker_processes`: Sets the number of worker processes NGINX should spawn. Value can be set to a specific number (e.g., `worker_processes 4;`) or `auto` to match CPU core count. More workers increase concurrency on multi-core systems.

- `worker_connections`: Defines the maximum number of simultaneous connections that each worker process can handle. The total max connections = worker_processes × worker_connections. Default is 512 or 1024.

- `include`: Imports configuration from other files, enabling modular configuration. Example: `include /etc/nginx/conf.d/*.conf;` includes all .conf files in that directory.

- `error_log`: Specifies the file where NGINX error logs are written. Syntax: `error_log /path/to/file [level];` where level can be debug, info, notice, warn, error, crit, alert, or emerg.

- `pid`: Defines the file that will store the process ID of the main NGINX process. Example: `pid /var/run/nginx.pid;`

**HTTP directives:**
- `server_name`: Specifies which domain name(s) this server block handles. Multiple names can be specified (e.g., `server_name example.com www.example.com;`). Supports exact names, wildcards, and regular expressions.

- `listen`: Configures the IP address and port the server listens on. Can include options like `default_server`, `ssl`, `http2`, etc. Example: `listen 443 ssl http2;`

- `root`: Defines the document root directory where NGINX looks for files to serve. Should be an absolute path. Example: `root /var/www/html;`

- `index`: Specifies default filenames to use when a directory is requested. NGINX tries each file in order. Example: `index index.html index.htm index.php;`

- `error_page`: Maps HTTP error codes to custom error page URLs. Example: `error_page 404 /404.html;`

- `try_files`: Checks for the existence of files in a specified order and uses the first one that exists. Example: `try_files $uri $uri/ /index.php?$args;` (common for PHP applications)

- `rewrite`: Modifies requested URLs based on regular expression patterns. Example: `rewrite ^/old-page$ /new-page permanent;` creates a 301 redirect.

- `return`: Issues a redirect or specific status code. Example: `return 301 https://$host$request_uri;` for HTTP to HTTPS redirection.

- `proxy_pass`: Specifies the address of a proxied server. Used in reverse proxy configurations. Example: `proxy_pass http://backend;` where backend can be a server address or upstream block.

## Server Blocks (Virtual Hosts)

Server blocks are similar to Apache's Virtual Hosts, allowing multiple domains on a single server:

```nginx
# First domain
server {
    listen 80;
    server_name example.com www.example.com;
    
    root /var/www/example.com;
    index index.html;
    
    # ... more configuration ...
}

# Second domain
server {
    listen 80;
    server_name another.example.org;
    
    root /var/www/another.example.org;
    index index.html;
    
    # ... more configuration ...
}
```

**Server Block Selection:**

When NGINX receives a request, it selects the server block based on:
1. The `listen` directive that matches the IP and port
2. The `server_name` directive that matches the Host header

Server name matching follows this order:
1. Exact name match (`example.com`)
2. Wildcard at the beginning (`*.example.com`)
3. Wildcard at the end (`example.*`)
4. Regular expression match (`~^www\d+\.example\.com$`)
5. Default server block (first one or marked with `default_server`)

## Location Blocks

Location blocks define how NGINX processes requests for specific URIs:

```nginx
server {
    listen 80;
    server_name example.com;
    
    # Exact match
    location = /exact {
        # ...
    }
    
    # Prefix match (with priority)
    location ^~ /images/ {
        # ...
    }
    
    # Regular expression (case-sensitive)
    location ~ \.php$ {
        # ...
    }
    
    # Regular expression (case-insensitive)
    location ~* \.(jpg|jpeg|png|gif)$ {
        # ...
    }
    
    # Standard prefix match
    location /docs/ {
        # ...
    }
    
    # Default catch-all
    location / {
        # ...
    }
}
```

**Location matching order:**
1. Exact matches (`=`) - highest priority
2. Preferential prefix matches (`^~`)
3. Regular expression matches (`~` or `~*`) - first match wins
4. Standard prefix matches (no modifier) - lowest priority

**Location block modifiers explained in detail:**
- `=` (Exact match): Matches the URI exactly as specified. Provides fastest performance as NGINX stops searching when it finds an exact match. Example: `location = /favicon.ico { ... }`

- `^~` (Preferential prefix): If the longest matching prefix location has this modifier, NGINX stops searching and uses this location. Useful for prioritizing static files. Example: `location ^~ /assets/ { ... }`

- `~` (Case-sensitive regex): Matches the URI using a case-sensitive regular expression. Example: `location ~ \.php$ { ... }`

- `~*` (Case-insensitive regex): Matches the URI using a case-insensitive regular expression. Example: `location ~* \.(jpg|jpeg|png)$ { ... }`

- No modifier (Prefix match): Matches the URI by prefix. NGINX continues searching for better matches. Example: `location /api/ { ... }`

**Common location directives explained:**
- `root`: Specifies the root directory for requests. The full path is constructed by appending the URI to the root value. Example: `root /var/www/html;` with URI `/images/logo.png` looks for `/var/www/html/images/logo.png`

- `alias`: Creates an alternative path mapping. The matched part of the URI is replaced with the alias value. Example: `location /images/ { alias /var/www/static/pictures/; }` maps `/images/logo.png` to `/var/www/static/pictures/logo.png`

- `try_files`: Checks for the existence of files in specified order and uses the first one found. If none found, processes the final parameter. Example: `try_files $uri $uri/ /index.php?$args;` first checks for the file, then the directory, then forwards to PHP

- `index`: Defines default files to serve when a directory is requested. Example: `index index.html index.php;`

- `autoindex`: When set to `on`, generates a directory listing when no index file is present. Should generally be `off` in production

- `deny`/`allow`: Controls access based on client IP address. Example: `deny 192.168.1.2; allow 10.0.0.0/8; deny all;` blocks one specific IP, allows an entire subnet, and denies everyone else

## Reverse Proxy

NGINX excels as a reverse proxy, forwarding requests to backend servers:

```nginx
server {
    listen 80;
    server_name example.com;
    
    # Basic reverse proxy
    location / {
        proxy_pass http://backend_server:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 5s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

**Advanced Reverse Proxy Configuration:**

```nginx
server {
    listen 80;
    server_name example.com;
    
    # API requests
    location /api/ {
        # Remove /api/ prefix when forwarding
        proxy_pass http://api-backend:8000/;
        
        # Buffer settings
        proxy_buffering on;
        proxy_buffer_size 8k;
        proxy_buffers 8 8k;
        
        # Upgrade support (WebSockets)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    # Static assets
    location /static/ {
        proxy_pass http://static-backend:9000/;
        proxy_cache my_cache_zone;
        proxy_cache_valid 200 302 60m;
        proxy_cache_valid 404 1m;
        expires 7d;
    }
    
    # Default application
    location / {
        proxy_pass http://web-backend:3000;
    }
}
```

**Proxy Headers Explained:**
- `Host`: Sends original Host header to backend server. This ensures the backend knows which virtual host was requested. Example: `proxy_set_header Host $host;`

- `X-Real-IP`: Passes the client's actual IP address to the backend. Without this, backend applications would only see NGINX's IP address as the client. Example: `proxy_set_header X-Real-IP $remote_addr;`

- `X-Forwarded-For`: Contains the complete chain of client IP addresses (useful when requests pass through multiple proxies). Format: client_ip, proxy1_ip, proxy2_ip. Example: `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;`

- `X-Forwarded-Proto`: Informs the backend about the protocol (http/https) used by the client to connect to NGINX. Critical for applications that generate absolute URLs. Example: `proxy_set_header X-Forwarded-Proto $scheme;`

- `X-Forwarded-Host`: Passes the original host requested by the client. Useful when the backend needs to know the exact hostname in the original request. Example: `proxy_set_header X-Forwarded-Host $host;`

- `X-Forwarded-Port`: Indicates the original port requested by the client. Useful for applications that need to generate URLs with the correct port. Example: `proxy_set_header X-Forwarded-Port $server_port;`

## Load Balancing

NGINX can distribute traffic across multiple servers:

```nginx
# Define upstream server group
upstream backend {
    server backend1.example.com weight=3;
    server backend2.example.com;
    server backend3.example.com backup;
    server backend4.example.com max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Load Balancing Methods

NGINX supports several load balancing algorithms:

```nginx
upstream backend {
    # Round Robin (default) - requests distributed equally
    server backend1.example.com;
    server backend2.example.com;
    
    # Weighted - proportional to weight value
    # server backend1.example.com weight=3;
    # server backend2.example.com weight=1;
    
    # Least Connections - server with fewest active connections
    # least_conn;
    
    # IP Hash - same client to same server (simple session persistence)
    # ip_hash;
    
    # Generic Hash - based on a defined key
    # hash $request_uri consistent;
    
    # Least Time (NGINX Plus only)
    # least_time header;
}
```

### Health Checks

**Passive Health Checks (Open Source):**

```nginx
upstream backend {
    server backend1.example.com max_fails=3 fail_timeout=30s;
    server backend2.example.com max_fails=3 fail_timeout=30s;
}
```

The passive health check parameters explained:
- `max_fails`: Maximum number of consecutive failed connection attempts before marking a server as temporarily unavailable. Default is 1.
- `fail_timeout`: Serves two purposes: 1) How long a server is considered unavailable after reaching max_fails, and 2) The timeframe in which max_fails is counted. Default is 10 seconds.
- These are called "passive" because NGINX doesn't actively test the backends but detects failures during actual client requests.

**Active Health Checks (NGINX Plus):**

```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    
    # Active health check
    health_check interval=5s jitter=3s fails=3 passes=2;
}

# Custom health check
location /health {
    health_check uri=/status match=status;
    proxy_pass http://backend;
}

# Define match
match status {
    status 200;
    body ~ "status: ok";
}
```

The active health check parameters explained:
- `health_check`: Enables proactive testing of backend servers by NGINX Plus at regular intervals
- `interval`: How frequently to check the health of upstream servers (5 seconds in this example)
- `jitter`: Random delay added to the interval to prevent all health checks happening simultaneously (3 seconds max in this example)
- `fails`: Number of consecutive failed checks before marking a server as unhealthy (3 in this example)
- `passes`: Number of consecutive successful checks required to mark a server as healthy again (2 in this example)
- `uri`: Path to probe on the backend server (defaults to /)
- `match`: References a named match block that defines success criteria for the health check

## Session Persistence

**IP Hash (Open Source):**

```nginx
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
}
```

**Sticky Cookies (NGINX Plus):**

```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    
    sticky cookie srv_id expires=1h domain=.example.com path=/;
}
```

## SSL/TLS Configuration

### SSL Certificate Setup

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    
    # Redirect all HTTP requests to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    
    # SSL certificate files
    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;
    
    # SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;  # Only use secure TLS versions (disable older vulnerable ones)
    # Modern cipher suite focusing on security, following Mozilla recommendations
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;  # Let client choose cipher (modern approach) vs on (server chooses)
    
    # OCSP Stapling - Improves performance by including certificate validation info
    # so clients don't need to check certificate revocation separately
    ssl_stapling on;  # Enable OCSP stapling
    ssl_stapling_verify on;  # Verify OCSP responses
    resolver 1.1.1.1 1.0.0.1 valid=300s;  # DNS servers to use for OCSP validation
    resolver_timeout 5s;  # DNS lookup timeout
    
    # SSL session parameters - Improve performance for returning visitors
    ssl_session_cache shared:SSL:10m;  # 10MB cache shared between worker processes
    ssl_session_timeout 1d;  # Keep SSL sessions in cache for 1 day
    ssl_session_tickets off;  # Disable TLS session tickets (security improvement)
    
    # HSTS (Strict Transport Security) - Forces clients to always use HTTPS
    # max-age=63072000: Remember for 2 years (in seconds)
    # includeSubDomains: Apply to all subdomains 
    # preload: Allow inclusion in browser preload lists
    # always: Add header regardless of response code
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    
    location / {
        root /var/www/html;
        index index.html;
    }
}
```

### SSL Optimization

**Let's Encrypt with Certbot:**

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain and install certificate
sudo certbot --nginx -d example.com -d www.example.com
```

**Set up a strong Diffie-Hellman group:**

```bash
sudo openssl dhparam -out /etc/nginx/dhparam.pem 2048
```

Add to your NGINX configuration:
```nginx
ssl_dhparam /etc/nginx/dhparam.pem;
```

## HTTP/2 and HTTP/3

**HTTP/2 Configuration:**

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;
    
    # HTTP/2 specific optimizations
    http2_push_preload on;              # Support for preload hints
    http2_max_concurrent_streams 128;   # Max concurrent HTTP/2 streams
    
    # ... rest of configuration
}
```

**HTTP/3 Configuration (NGINX 1.25.0+):**

```nginx
server {
    listen 443 ssl http2;
    listen 443 quic;  # HTTP/3 using QUIC
    server_name example.com;
    
    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;
    
    # HTTP/3 headers
    add_header Alt-Svc 'h3=":443"; ma=86400';
    
    # ... rest of configuration
}
```

## Security Best Practices

### Headers Security

```nginx
server {
    # ... server configuration ...
    
    # Security headers
    # Prevents browsers from MIME-sniffing (interpreting files as different MIME types than declared)
    add_header X-Content-Type-Options "nosniff" always;
    
    # Controls how the page can be embedded in iframes:
    # DENY: Cannot be displayed in frames
    # SAMEORIGIN: Only pages from same origin can frame this content
    # ALLOW-FROM uri: Only specified URI can frame this content
    add_header X-Frame-Options "SAMEORIGIN" always;
    
    # Enables browser's XSS filtering and prevents rendering of page if attack detected
    # Note: Modern browsers are phasing this out in favor of Content-Security-Policy
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Controls how much referrer information is included with requests
    # strict-origin-when-cross-origin: Send origin, path, and querystring when same origin,
    # only origin when cross-origin HTTPS→HTTPS, and no header when HTTPS→HTTP
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    
    # Controls which browser features can be used (formerly Feature-Policy)
    # Empty parentheses means feature is denied for all origins
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=(), interest-cohort=()";
    
    # HSTS (Strict Transport Security) - Forces clients to always use HTTPS
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    
    # Prevents leaking server version in headers and error pages (security by obscurity)
    # Hides information useful to attackers targeting specific NGINX versions
    server_tokens off;
}
```

### Rate Limiting

```nginx
# Define a limit zone based on client IP
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;

server {
    listen 80;
    server_name example.com;
    
    # Apply rate limiting to login page
    location /login {
        limit_req zone=mylimit burst=20 nodelay;
        
        # Proceed with normal processing
        proxy_pass http://backend/login;
    }
    
    # Apply a different rate limit to API
    location /api/ {
        limit_req zone=mylimit burst=5 nodelay;
        proxy_pass http://api-backend;
    }
}
```

**Rate limiting directives explained in detail:**
- `limit_req_zone`: Defines parameters for rate limiting.
  - `$binary_remote_addr`: Key to differentiate clients (using binary form of IP saves memory)
  - `zone=mylimit:10m`: Names the zone "mylimit" and allocates 10MB of memory for tracking client states
  - `rate=10r/s`: Limits each client to 10 requests per second (can also use r/m for per minute)

- `limit_req`: Applies the rate limiting zone to a location.
  - `zone=mylimit`: References the named zone defined earlier
  - `burst=20`: Allows a burst of 20 additional requests beyond the specified rate when traffic surges
  - `nodelay`: Processes excessive requests immediately up to the burst limit rather than queueing them
    - Without `nodelay`, excessive requests within burst limit would be delayed
    - With `nodelay`, they are processed immediately, but still limited by burst size

For context: a 10MB zone can hold about 160,000 IP addresses, while 1r/s with burst=5 allows a client one request per second, plus a burst of 5 additional requests processed immediately.

## Caching

Caching can significantly improve performance by storing frequently requested content in memory or on disk.

### Proxy Caching

```nginx
# Define cache path and parameters
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m use_temp_path=off;

server {
    listen 80;
    server_name example.com;
    
    location / {
        # Activate cache
        proxy_cache my_cache;
        
        # Cache valid for 10 minutes for 200 and 302 responses, 1 minute for 404
        proxy_cache_valid 200 302 10m;
        proxy_cache_valid 404 1m;
        
        # Bypass cache if requested by client
        proxy_cache_bypass $http_cache_control;
        
        # Use stale content in case of error or timeout
        proxy_cache_use_stale error timeout http_500;
        
        # Lock cache entry during population to prevent stampede
        proxy_cache_lock on;
        
        # Set expiration for browser caching
        expires 30d;
        add_header Cache-Control "public, no-transform";
        
        proxy_pass http://backend;
    }
}
```

**Cache control directives explained:**
- `proxy_cache_path`: Defines cache location and parameters
  - `levels=1:2`: Creates a two-level directory hierarchy to improve file lookup performance
  - `keys_zone=my_cache:10m`: Allocates 10MB of shared memory for cache keys (can store ~80,000 entries per MB)
  - `max_size=1g`: Sets maximum cache size to 1GB
  - `inactive=60m`: Items unused for 60 minutes are removed regardless of their validity
  - `use_temp_path=off`: Writes files directly to cache path instead of using a temp directory

- `proxy_cache`: Activates cache zone for the current context (server or location block)
  
- `proxy_cache_valid`: Sets caching time for different response codes
  - `200 302 10m`: Successful responses and redirects are cached for 10 minutes
  - `404 1m`: Not found responses are cached for 1 minute
  
- `proxy_cache_bypass`: Specifies conditions when caching should be bypassed
  - `$http_cache_control`: Bypasses cache when the client sends specific Cache-Control headers
  
- `proxy_cache_use_stale`: Allows serving stale content in specific scenarios like `error`, `timeout`, `updating`, or `http_500`
  
- `proxy_cache_lock`: When enabled, only one request at a time will populate the cache entry, preventing cache stampede
  
- `expires`: Sets Expires and Cache-Control headers in response to enable browser caching
  - `30d`: Tells browsers to cache content for 30 days
  
- `add_header Cache-Control`: Provides more detailed caching instructions to browsers
  - `public`: Response can be cached by any cache (browser, CDN, etc.)
  - `no-transform`: Prevents proxies from modifying the content (particularly images)

## Optimization Techniques

### Worker and connection directives explained:
- `worker_processes auto`: Automatically sets worker processes to match CPU cores. Each worker is single-threaded and handles connections independently. For specific resource allocation, you can set an explicit number like `worker_processes 4;`

- `worker_rlimit_nofile 65535`: Increases the limit on open files per worker, important for high-traffic sites. Should be higher than worker_connections

- `worker_connections 4096`: Maximum number of connections each worker can handle simultaneously. Total capacity = worker_processes × worker_connections

- `multi_accept on`: Allows a worker to accept all new connections at once rather than one at a time

- `use epoll`: Sets the connection processing method. Epoll is the most efficient for Linux, while other OS use kqueue (FreeBSD/macOS) or select/poll

### Connection tuning directives explained:
- `keepalive_timeout 65`: Number of seconds a keep-alive client connection will stay open. Lower values save resources but might increase reconnections

- `keepalive_requests 100`: Maximum number of requests allowed per keep-alive connection

- `client_body_timeout 12s`: Time NGINX waits for client to send the body of a request. Helps prevent slow client attacks

- `client_header_timeout 12s`: Time NGINX waits for client to send the complete request header

- `send_timeout 10s`: Time NGINX waits for client to receive data. Timer is reset when client receives data

- `client_body_buffer_size 16k`: Size of buffer for client request body. Requests larger than this are written to disk

- `client_max_body_size 10m`: Maximum allowed size of client request body. Requests exceeding this return 413 (Request Entity Too Large)

- `tcp_nopush on`: Optimizes the amount of data sent at once. Only works when sendfile is on

- `tcp_nodelay on`: Forces sending data immediately (no buffering). Useful for streaming and WebSocket connections

- `sendfile on`: Enables the use of sendfile for faster file transfers directly from disk to network socket

- `aio on`: Enables asynchronous I/O for improved performance with large files

- `directio 512`: Minimizes disk activity by using direct I/O for files larger than specified size (in this case 512 bytes)

### Compression directives explained:
- `gzip on`: Enables gzip compression to reduce transmission size

- `gzip_comp_level 5`: Sets compression level (1-9). Higher values use more CPU resources

- `gzip_min_length 256`: Minimum size file to compress (smaller files aren't worth compressing)

- `gzip_proxied any`: Compress responses for proxied requests regardless of headers

- `gzip_vary on`: Adds "Vary: Accept-Encoding" header to indicate content varies based on encoding

- `gzip_types`: List of MIME types to compress beyond the default text/html

- `gzip_disable "msie6"`: Disables compression for specified user agents (legacy IE had issues)

### Open file cache directives explained:
- `open_file_cache max=1000 inactive=20s`: Caches information about open files for up to 1000 files, removing files not accessed for 20 seconds

- `open_file_cache_valid 30s`: Specifies how often cache validity is checked

- `open_file_cache_min_uses 2`: Minimum number of requests for a file before it's cached

- `open_file_cache_errors on`: Enables caching of file lookup errors as well as successful lookups

**Content Security Policy directives explained:**
- `default-src 'self'`: Default fallback policy. Restricts resources to load only from the same origin

- `script-src 'self' https://trusted-cdn.com`: JavaScript can only be loaded from same origin or the specified CDN
  
- `img-src 'self' data: https://*.trusted-site.com`: Images can load from same origin, data: URIs, or specified domains

- `style-src 'self' 'unsafe-inline' https://trusted-cdn.com`: Stylesheets can load from same origin, be defined inline, or from the CDN
  - `'unsafe-inline'` allows inline styles but reduces security

- `font-src 'self' https://trusted-cdn.com`: Fonts can only load from same origin or specified CDN

- `object-src 'none'`: Blocks all plugins like Flash, Java, etc. (strong security practice)

- `frame-ancestors 'self'`: Controls which sites can embed your site in iframes (similar to X-Frame-Options)

- `base-uri 'self'`: Restricts URLs which can be used in base element (prevents base URL hijacking attacks)

- `form-action 'self'`: Restricts where forms on your site can submit to (prevents certain CSRF attacks)

A properly configured CSP significantly mitigates the risk of XSS attacks by controlling which resources can be executed. The `always` parameter ensures the header is sent with all response codes (including errors).

## Logging and Monitoring

### Access and error logging

```nginx
http {
    # ... other settings ...
    
    # Log format for access logs
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    
    # Access log configuration
    access_log /var/log/nginx/access.log main;
    
    # Error log configuration
    error_log /var/log/nginx/error.log warn;
}
```

**Logging directives and variables explained:**
- `log_format`: Defines a custom log format by combining variables:
  - `$remote_addr`: Client's IP address
  - `$remote_user`: Username provided for HTTP basic authentication
  - `$time_local`: Local time of the request in Common Log Format
  - `$request`: Full HTTP request line (e.g., "GET /path HTTP/1.1")
  - `$status`: HTTP response status code
  - `$body_bytes_sent`: Number of bytes sent to client excluding headers
  - `$http_referer`: Value of the Referer header (the origin page)
  - `$http_user_agent`: Value of the User-Agent header (browser identification)
  - `$http_x_forwarded_for`: Client IPs if request passed through proxies
  - `$request_time`: Request processing time in seconds
  - `$upstream_response_time`: Time taken by upstream server to respond

- `access_log`: Configures access log destination and format
  - `/var/log/nginx/example.com.access.log json_combined`: Path with format name
  - Adding `if=$loggable` makes logging conditional

- `error_log`: Configures error log destination and verbosity level
  - Levels (from most to least verbose): debug, info, notice, warn, error, crit, alert, emerg
  
- `map $status $loggable { ... }`: Creates a variable $loggable based on response status
  - `~^[23] 0;`: Matches 2xx and 3xx responses and sets variable to 0 (false)
  - `default 1;`: Sets variable to 1 (true) for all other responses

- `json_combined` format with `escape=json`: Creates valid JSON output for easier parsing by log analysis tools

The directive `access_log off` completely disables logging for specific locations like favicon or static files, saving disk space and improving performance.

### Monitoring with status codes

```nginx
server {
    listen 80;
    server_name example.com;
    
    location / {
        # ... other settings ...
        
        # Log all requests
        access_log /var/log/nginx/example.com.access.log;
    }
    
    location /health {
        # Health check endpoint
        access_log off;  # Disable logging for health checks
        return 200;
    }
    
    location /favicon.ico {
        # Favicon requests
        access_log off;  # Disable logging for favicon
        try_files $uri =204;  # Return 204 No Content if not found
    }
    
    error_page 404 /custom_404.html;  # Custom 404 error page
}
```

**Monitoring directives explained:**
- `access_log /var/log/nginx/example.com.access.log;`: Logs all requests to the specified access log file

- `location /health { ... }`: Defines a location for health checks
  - `access_log off;`: Disables access logging for this location to reduce log noise
  - `return 200;`: Responds with HTTP 200 OK

- `location /favicon.ico { ... }`: Special handling for favicon requests
  - `access_log off;`: Disables access logging for this location
  - `try_files $uri =204;`: Returns HTTP 204 No Content if the favicon is not found, preventing 404 errors

- `error_page 404 /custom_404.html;`: Serves a custom 404 error page for better user experience and SEO

## Common Use Cases with Sample Configurations

### Serving static files

```nginx
server {
    listen 80;
    server_name static.example.com;
    
    root /var/www/static;
    index index.html index.htm;
    
    location / {
        # Cache static files for 30 days
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }
}
```

### Reverse proxy to an application server

```nginx
server {
    listen 80;
    server_name app.example.com;
    
    location / {
        proxy_pass http://app_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Load balancing between multiple servers

```nginx
upstream backend {
    server app1.example.com;
    server app2.example.com;
}

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### SSL termination with Let's Encrypt

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    
    # Redirect all HTTP requests to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    location / {
        root /var/www/html;
        index index.html;
    }
}
```

### HTTP/2 and HTTP/3 configuration

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;
    
    # HTTP/2 specific optimizations
    http2_push_preload on;
    http2_max_concurrent_streams 128;
    
    location / {
        root /var/www/html;
        index index.html;
    }
}

server {
    listen 443 quic;
    server_name example.com;
    
    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;
    
    # HTTP/3 headers
    add_header Alt-Svc 'h3=":443"; ma=86400';
    
    location / {
        root /var/www/html;
        index index.html;
    }
}
```

## Troubleshooting

### Common issues and solutions

- **Issue**: NGINX fails to start or reload configuration
  - **Solution**: Check configuration syntax with `nginx -t`. Fix any reported errors.

- **Issue**: 502 Bad Gateway error
  - **Solution**: Check if the backend server is running and accessible. Verify the `proxy_pass` directive.

- **Issue**: SSL certificate errors
  - **Solution**: Ensure the certificate files exist and have correct permissions. Check the `ssl_certificate` and `ssl_certificate_key` directives.

- **Issue**: Static files not found (404 errors)
  - **Solution**: Verify the `root` directive points to the correct directory. Check file permissions.

- **Issue**: High server load or slow response times
  - **Solution**: Optimize worker and connection settings. Consider enabling caching. Check for slow backend responses.

- **Issue**: NGINX returns the wrong server block
  - **Solution**: Check the `server_name` directive. Ensure the correct server block matches the requested hostname.

### Debugging tips

- Use `error_log` with `debug` level to get detailed debugging information.
- Check the access and error logs for clues about request processing and errors.
- Use tools like `curl` or `wget` to test requests and inspect responses.
- For upstream server issues, check the health of the backend servers and their logs.

## Major Production Errors and Recovery

This section details critical production errors that can severely impact your service, including real-world scenarios, impact analysis, and proven recovery strategies.

### 500 Internal Server Error

**Scenario 1: Application Code Failure**
- **Incident**: During a peak shopping period, our e-commerce platform began serving 500 errors to approximately 70% of customers.
- **Impact**: 43% drop in conversion rate, estimated revenue loss of $27,000 per hour.
- **Root Cause**: A code deployment introduced a memory leak in the PHP application, causing worker processes to exhaust available memory.
- **Solution**:
  ```bash
  # First, verify the issue in NGINX and application logs
  tail -f /var/log/nginx/error.log
  tail -f /var/log/php/php-fpm.log
  
  # Implement immediate mitigation by increasing PHP memory limit
  sed -i 's/memory_limit = 128M/memory_limit = 256M/' /etc/php/7.4/fpm/php.ini
  systemctl restart php7.4-fpm
  
  # Roll back the problematic deployment
  cd /var/www/html
  git revert HEAD --no-edit
  
  # Verify services have recovered
  systemctl status nginx php7.4-fpm
  ```
- **Prevention**: Implemented pre-deployment memory profiling and gradual rollout strategy with automated canary analysis.

**Scenario 2: Database Connection Exhaustion**
- **Incident**: After database maintenance, all API endpoints began returning 500 errors.
- **Impact**: Complete service outage for 17 minutes, affecting 100% of users.
- **Root Cause**: Connection pool settings were incorrectly configured, causing connections to be held open without being released.
- **Solution**:
  ```bash
  # Check NGINX and application connection status
  netstat -anp | grep ESTABLISHED | wc -l
  
  # Identify connection leaks
  netstat -anp | grep ESTABLISHED | grep mysql
  
  # Adjust connection pool settings in the application
  vim /etc/app/config.json  # Reduce max connections and add timeout
  
  # Restart application servers with new settings
  systemctl restart app-server
  
  # Monitor recovery
  watch -n1 "netstat -anp | grep ESTABLISHED | wc -l"
  ```
- **Prevention**: Implemented connection pool monitoring with alerts for unusual growth patterns and automatic circuit breaking.

### 502 Bad Gateway

**Scenario: Upstream Server Overload**
- **Incident**: Following a marketing campaign, our service began intermittently responding with 502 errors.
- **Impact**: 30% of requests failing during peak periods, resulting in negative social media mentions and customer complaints.
- **Root Cause**: The Node.js application servers were overwhelmed, causing request timeouts to NGINX.
- **Solution**:
  ```bash
  # Increase timeout values in NGINX configuration
  vim /etc/nginx/conf.d/upstream.conf
  
  # Add these directives
  proxy_connect_timeout 300s;
  proxy_send_timeout 300s;
  proxy_read_timeout 300s;
  
  # Implement request queuing
  limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
  
  # Apply the rate limiting
  location /api/ {
      limit_req zone=api_limit burst=20 nodelay;
      proxy_pass http://backend_servers;
  }
  
  # Test and reload NGINX
  nginx -t && systemctl reload nginx
  ```
- **Prevention**: Implemented auto-scaling based on CPU and request queue metrics, and added a CDN for static content.

### 504 Gateway Timeout

**Scenario: Database Query Timeouts**
- **Incident**: Users reported inability to complete checkout process, with requests hanging for 30+ seconds before failing.
- **Impact**: 60% cart abandonment rate, double the normal rate.
- **Root Cause**: A missing database index caused product inventory queries to perform full table scans.
- **Solution**:
  ```bash
  # Identify slow queries
  mysql -e "SHOW FULL PROCESSLIST" | grep -i inventory
  
  # Create missing index (through database migration)
  mysql -e "CREATE INDEX idx_product_inventory ON products(inventory_id);"
  
  # Increase NGINX timeouts temporarily while the database recovers
  vim /etc/nginx/conf.d/timeouts.conf
  # Add:
  proxy_read_timeout 120s;
  
  # Reload NGINX
  nginx -t && systemctl reload nginx
  
  # Monitor query performance
  watch -n5 "mysql -e 'SHOW FULL PROCESSLIST' | grep -i inventory | wc -l"
  ```
- **Prevention**: Implemented query performance monitoring, automatic slow query detection, and required database index review for all schema changes.

### 403 Forbidden (WAF Blocking)

**Scenario: False Positive WAF Rules**
- **Incident**: After enabling ModSecurity WAF, legitimate customer requests were blocked with 403 errors.
- **Impact**: 15% of mobile users unable to access the service, increasing customer support volume by 300%.
- **Root Cause**: Overly aggressive WAF rule was blocking certain mobile user-agent strings.
- **Solution**:
  ```bash
  # Identify the blocking rules
  grep -i denied /var/log/modsec_audit.log | tail -50
  
  # Create a custom exception rule
  cat << EOF > /etc/nginx/modsec/custom-rules.conf
  SecRule REQUEST_HEADERS:User-Agent "@contains MobileApp/2.1" \
    "id:1000001,\
    phase:1,\
    pass,\
    nolog,\
    ctl:ruleRemoveById=941100"
  EOF
  
  # Test and reload
  nginx -t && systemctl reload nginx
  
  # Monitor false positive rate
  tail -f /var/log/modsec_audit.log | grep -i denied
  ```
- **Prevention**: Implemented a WAF testing environment with real traffic replay and rule tuning based on false positive analysis.

### Network-Level Failures

**Scenario: DNS Resolution Failure**
- **Incident**: NGINX began failing to proxy requests to third-party API services despite no changes to our configuration.
- **Impact**: Payment processing failed for 40% of transactions, resulting in lost orders.
- **Root Cause**: Our DNS provider experienced an outage, causing DNS resolution failures for upstream hosts.
- **Solution**:
  ```bash
  # Verify DNS resolution issues
  dig payment-api.partner.com
  
  # Add static DNS entries to /etc/hosts as temporary fix
  echo "203.0.113.54 payment-api.partner.com" >> /etc/hosts
  
  # Create local DNS cache with dnsmasq
  apt-get update && apt-get install -y dnsmasq
  
  # Configure dnsmasq with external DNS providers
  cat << EOF > /etc/dnsmasq.conf
  no-resolv
  server=8.8.8.8
  server=1.1.1.1
  EOF
  
  # Enable and start service
  systemctl enable dnsmasq && systemctl start dnsmasq
  
  # Update resolv.conf to use local DNS
  echo "nameserver 127.0.0.1" > /etc/resolv.conf
  ```
- **Prevention**: Implemented redundant DNS providers, local DNS caching, and periodic DNS resolution health checks.

### TLS Certificate Expiry

**Scenario: Expired SSL Certificate**
- **Incident**: At midnight, all secure connections began failing with browser certificate warnings.
- **Impact**: 95% drop in traffic as users encountered scary security warnings. Brand reputation damage on social media.
- **Root Cause**: SSL certificate expired and auto-renewal failed due to DNS verification issues.
- **Solution**:
  ```bash
  # Verify certificate expiration
  openssl x509 -enddate -noout -in /etc/nginx/ssl/example.com.crt
  
  # Generate emergency certificate with Let's Encrypt
  certbot certonly --standalone -d example.com -d www.example.com
  
  # Update NGINX configuration with new certificate
  vim /etc/nginx/conf.d/ssl.conf
  # Update paths:
  # ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
  # ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
  
  # Test and reload
  nginx -t && systemctl reload nginx
  ```
- **Prevention**: Implemented certificate expiration monitoring with 30/14/7 day alerts, automated renewal testing, and documented emergency certificate procedures.

### Out of File Descriptors

**Scenario: File Descriptor Exhaustion**
- **Incident**: During high traffic period, NGINX began rejecting connections with "too many open files" errors.
- **Impact**: Intermittent 503 errors affecting approximately 25% of users.
- **Root Cause**: Default system limits for file descriptors were too low for our traffic volume.
- **Solution**:
  ```bash
  # Check current limits
  cat /proc/$(cat /var/run/nginx.pid)/limits | grep "open files"
  
  # Increase system-wide limits temporarily
  echo "fs.file-max = 2097152" >> /etc/sysctl.conf
  sysctl -p
  
  # Update NGINX service file limits
  mkdir -p /etc/systemd/system/nginx.service.d
  cat << EOF > /etc/systemd/system/nginx.service.d/limits.conf
  [Service]
  LimitNOFILE=500000
  EOF
  
  # Reload systemd and restart NGINX
  systemctl daemon-reload
  systemctl restart nginx
  ```
- **Prevention**: Added file descriptor usage to monitoring dashboards, and adjusted worker_connections and worker_processes based on load testing results.
