# Nginx Load Balancing Methods

This document outlines various load balancing methods available in Nginx and provides configuration examples for each approach.

## Table of Contents
- [1. Round Robin (Default)](#1-round-robin-default)
- [2. Least Connections](#2-least-connections)
- [3. IP Hash (Sticky Sessions)](#3-ip-hash-sticky-sessions)
- [4. Weighted Load Balancing](#4-weighted-load-balancing)
- [5. Health Checks](#5-health-checks)
- [6. Least Time](#6-least-time)
- [7. Consistent Hashing](#7-consistent-hashing)
- [8. Dynamic Load Balancing](#8-dynamic-load-balancing)
- [9. Proxy Protocol for Load Balancers](#9-proxy-protocol-for-load-balancers)
- [10. Failover](#10-failover)
- [Implementation Guidelines](#implementation-guidelines)

## 1. Round Robin (Default)

**How it works:** Requests are distributed evenly across all servers in the upstream block.

**When to use:** Suitable for stateless applications or when all backend servers have equal capacity.

**Example:**
```nginx
upstream app_servers {
    server 192.168.101.14:5024;
    server 192.168.101.15:5024;
}
```

## 2. Least Connections

**How it works:** Routes requests to the server with the fewest active connections.

**When to use:** Useful when the backend servers have varying response times or when traffic patterns vary.

**Example:**
```nginx
upstream app_servers {
    least_conn;
    server 192.168.101.14:5024;
    server 192.168.101.15:5024;
}
```

## 3. IP Hash (Sticky Sessions)

**How it works:** Maps requests to servers based on the client's IP address.

**When to use:** Useful for applications that require session persistence without external session storage.

**Example:**
```nginx
upstream app_servers {
    ip_hash;
    server 192.168.101.14:5024;
    server 192.168.101.15:5024;
}
```

## 4. Weighted Load Balancing

**How it works:** Assigns weights to servers to distribute traffic unevenly based on their capacity.

**When to use:** Use when servers have different resource capabilities or configurations.

**Example:**
```nginx
upstream app_servers {
    server 192.168.101.14:5024 weight=3;
    server 192.168.101.15:5024 weight=1;
}
```

## 5. Health Checks

**How it works:** Monitors backend server health and removes unhealthy servers from the load balancing pool.

**When to use:** Critical for high-availability systems.

**Example (requires Nginx Plus):**
```nginx
upstream app_servers {
    server 192.168.101.14:5024;
    server 192.168.101.15:5024;
    health_check;
}
```

**Note:** Basic health checks are available in open-source Nginx using passive health monitoring:
```nginx
upstream app_servers {
    server 192.168.101.14:5024 max_fails=3 fail_timeout=30s;
    server 192.168.101.15:5024 max_fails=3 fail_timeout=30s;
}
```

## 6. Least Time

**How it works:** Routes requests to the server with the lowest average response time and least number of active connections.

**When to use:** Useful for applications where response time is critical.

**Example (requires Nginx Plus):**
```nginx
upstream app_servers {
    least_time header;
    server 192.168.101.14:5024;
    server 192.168.101.15:5024;
}
```

## 7. Consistent Hashing

**How it works:** Maps requests to servers based on a hash of a key (e.g., URL, header, or cookie).

**When to use:** Use for distributed caching or applications like content delivery networks (CDNs).

**Example:**
```nginx
upstream app_servers {
    hash $request_uri;
    server 192.168.101.14:5024;
    server 192.168.101.15:5024;
}
```

## 8. Dynamic Load Balancing

**How it works:** Dynamically updates backend servers based on DNS or an external API.

**When to use:** Useful in cloud environments or when servers scale dynamically.

**Example:**
```nginx
upstream app_servers {
    server backend1.example.com;
    server backend2.example.com;
}
```

## 9. Proxy Protocol for Load Balancers

**How it works:** Preserves the client's original IP when requests pass through additional load balancers (e.g., AWS ELB).

**When to use:** When chaining multiple load balancers or preserving client information is critical.

**Example:**
```nginx
upstream app_servers {
    server 192.168.101.14:5024;
    server 192.168.101.15:5024;
}

location / {
    proxy_pass http://app_servers;
    proxy_protocol on;
}
```

## 10. Failover

**How it works:** Sends requests to the primary server and uses a backup server only if the primary is down.

**When to use:** Ensures high availability with a primary-backup setup.

**Example:**
```nginx
upstream app_servers {
    server 192.168.101.14:5024 max_fails=3 fail_timeout=30s;
    server 192.168.101.15:5024 backup;
}
```

## Implementation Guidelines

### Basic Nginx Load Balancer Configuration

Here's a complete example showing how to configure Nginx as a load balancer:

```nginx
http {
    upstream backend {
        # Choose one of the load balancing methods above
        server 192.168.101.14:5024;
        server 192.168.101.15:5024;
    }

    server {
        listen 80;
        server_name example.com;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

### Common Parameters for Upstream Servers

- **weight**: Defines server weight for load balancing
- **max_fails**: Number of failed attempts before marking server as unavailable
- **fail_timeout**: Time period to mark a server unavailable and check again
- **backup**: Marks server as backup (only used when primary servers are down)
- **down**: Marks server as permanently unavailable

### Important Considerations

1. **Session Persistence**: Choose IP hash or cookie-based solutions for applications requiring session persistence.
2. **Server Capacity**: Use weighted distribution when servers have different capacities.
3. **High Availability**: Implement health checks and failover for critical systems.
4. **Performance**: For optimal performance, consider using least connections or least time (Nginx Plus).
5. **Logging and Monitoring**: Set up proper logging to monitor load balancer performance.

### Nginx vs. Nginx Plus

Some advanced features like active health checks and least time algorithm require Nginx Plus (commercial version). The open-source version still offers robust load balancing capabilities with passive health monitoring.
