# IMESH
# Kubernetes Deployment Assignment

## Objective

Deploy the following applications:

- nginx
- httpbin
- echoserver

Each application:

- Runs as a Deployment
- Uses Kubernetes Service (ClusterIP)
- Includes Resource Requests and Limits
- Includes Readiness Probe
- Includes Liveness Probe

---

## Files Included

- nginx-deployment.yaml
- nginx-service.yaml
- httpbin-deployment.yaml
- httpbin-service.yaml
- echoserver-deployment.yaml
- echoserver-service.yaml
- commands.txt

---

## Deployment

```bash
kubectl apply -f .
```

---

## Verification

```bash
kubectl get deployments
kubectl get pods
kubectl get svc
kubectl get endpoints
```

---

## Internal Testing

```bash
kubectl run curl-test --image=curlimages/curl -it --rm -- sh

curl http://nginx-service

curl http://httpbin-service/get

curl http://echoserver-service
```

---

## Cleanup

```bash
kubectl delete -f .
```

# Assignment 2 - API Gateway

## Features Implemented

- Host-based Routing
- Path-based Routing
- URL Rewrite
- Header Injection
- Request Timeout
- Retry Policy
- Rate Limiting (10 requests/min)

## Backends

| Host | Backend |
|------|---------|
| web.company.local | nginx |
| api.company.local | httpbin |
| test.company.local | echoserver |

## Apply Files

```bash
kubectl apply -f gateway.yaml
kubectl apply -f httproute.yaml
kubectl apply -f header-filter.yaml
kubectl apply -f rewrite-filter.yaml
kubectl apply -f timeout.yaml
kubectl apply -f retry.yaml
kubectl apply -f ratelimit.yaml
```

## Verification

Use the commands in `verification-commands.txt`.
# Assignment 3 - TLS & Certificates

## Objectives

- Install cert-manager
- Create Self-Signed CA
- Create ClusterIssuer
- Create TLS Certificate
- Create Kubernetes TLS Secret
- Configure HTTPS Gateway

## Files

- selfsigned-ca.yaml
- clusterissuer.yaml
- certificate.yaml
- gateway-tls.yaml

## Installation

```bash
kubectl apply -f selfsigned-ca.yaml
kubectl apply -f clusterissuer.yaml
kubectl apply -f certificate.yaml
kubectl apply -f gateway-tls.yaml
```

## Verification

Run the commands in verification-commands.txt

# Assignment 4 – Networking Fundamentals

## Objective

Explain the complete traffic journey when a user accesses:

```
https://api.company.local/orders
```

The explanation covers DNS resolution, Gateway, Envoy Proxy, Kubernetes Service, kube-proxy/eBPF, Pod, Container, and the OSI layers involved in processing the request.

---

## Scenario

A user sends the following request:

```
https://api.company.local/orders
```

The request passes through multiple networking components before reaching the application running inside a Kubernetes Pod.

---

## Complete Traffic Journey

### Step 1 – DNS Resolution

- The browser receives the URL.
- DNS resolves `api.company.local` to the Gateway IP.
- The browser now knows where to send the request.

---

### Step 2 – HTTPS Connection

- Browser establishes a TCP connection on port **443**.
- TLS handshake is performed.
- The Gateway presents its TLS certificate.
- After certificate verification, an encrypted connection is established.

---

### Step 3 – Gateway

- The HTTPS request reaches the Envoy Gateway.
- The Gateway terminates TLS.
- It checks the hostname and path.
- The matching HTTPRoute is selected.

---

### Step 4 – Envoy Proxy

Envoy performs Layer 7 processing including:

- Host-based routing
- Path-based routing
- URL rewrite
- Header injection
- Retry policy
- Request timeout
- Rate limiting
- Logging and metrics

The request is then forwarded to the Kubernetes Service.

---

### Step 5 – Kubernetes Service

- The Service provides a stable virtual IP.
- It load balances traffic across healthy Pods.

---

### Step 6 – kube-proxy / eBPF

The Service forwards traffic using:

**kube-proxy**
- Uses iptables/IPVS rules
- Selects a healthy Pod

OR

**eBPF**
- Kernel-level packet forwarding
- Lower latency
- Better performance

---

### Step 7 – Pod

The selected Pod receives the request and forwards it to the application container.

---

### Step 8 – Container

The application processes:

```
GET /orders
```

It generates an HTTP response and sends it back through the same path.

---

## OSI Layer Explanation

### Layer 3 – Network Layer

Responsible for:

- IP addressing
- Packet routing
- Network communication

Protocols:
- IP
- ICMP

---

### Layer 4 – Transport Layer

Responsible for:

- TCP connection
- Port numbers
- Reliable data transfer

Protocol:
- TCP

---

### Layer 7 – Application Layer

Responsible for:

- HTTP/HTTPS
- Host routing
- URL routing
- Headers
- Authentication
- Rate limiting
- Retries
- Timeouts

Envoy Gateway operates at this layer.

---

## Architecture Diagram

```
                User Browser
                     │
                     ▼
              DNS Resolution
                     │
                     ▼
              Envoy Gateway
                     │
                     ▼
               HTTPRoute Match
                     │
                     ▼
                Envoy Proxy
                     │
                     ▼
            Kubernetes Service
                     │
                     ▼
            kube-proxy / eBPF
                     │
                     ▼
                   Pod
                     │
                     ▼
                Container
                     │
                     ▼
              HTTP Response
```

---

## Conclusion

The request flows from the browser to DNS, then to the Envoy Gateway where TLS is terminated and routing decisions are made. The request is forwarded through the Kubernetes Service, routed by kube-proxy or eBPF to a healthy Pod, processed by the application container, and finally returned to the client over the secure HTTPS connection.

# Assignment 5 – Troubleshooting

## Objective

Analyze common Kubernetes Gateway and networking issues by identifying:

- Possible root causes
- Commands to execute
- How to identify the issue
- Corrective actions

---

# Scenario 1

## Problem

```
Pods are Running
Service exists
Gateway exists

curl https://api.company.local

503 Service Unavailable
```

### Possible Root Causes

- Service has no endpoints
- Incorrect Service selector
- Pods are not Ready
- HTTPRoute is not attached
- Incorrect Service port
- Backend is unreachable

### Commands

```bash
kubectl get pods -n gateway-demo
kubectl get svc -n gateway-demo
kubectl get endpoints -n gateway-demo
kubectl get httproute -n gateway-demo
kubectl describe httproute app-route
kubectl describe gateway envoy-gateway
kubectl logs -n envoy-gateway-system deployment/envoy-gateway
```

### Identification

- Verify Service endpoints.
- Check Pod labels and Service selector.
- Confirm HTTPRoute attachment.
- Inspect Envoy logs.

### Corrective Action

- Fix Service selector.
- Ensure Pods are Ready.
- Correct Service ports.
- Update HTTPRoute configuration.

---

# Scenario 2

## Problem

```
Gateway is healthy

Application cannot be reached

Connection timed out
```

### Possible Root Causes

- Incorrect DNS
- Gateway not exposed
- Firewall blocking traffic
- NetworkPolicy restrictions
- Incorrect Gateway IP

### Commands

```bash
kubectl get gateway
kubectl get svc -A
kubectl describe svc envoy-gateway
kubectl get networkpolicy -A
nslookup api.company.local
ping api.company.local
curl -v https://api.company.local
```

### Identification

- Verify Gateway IP.
- Check DNS resolution.
- Review firewall and NetworkPolicy.

### Corrective Action

- Correct DNS.
- Expose Gateway properly.
- Allow required network traffic.

---

# Scenario 3

## Problem

```
HTTPS fails

certificate verify failed
```

### Possible Root Causes

- Expired certificate
- Wrong TLS Secret
- Invalid certificate
- Hostname mismatch
- Self-signed certificate not trusted

### Commands

```bash
kubectl get certificate
kubectl describe certificate gateway-cert
kubectl get secret gateway-tls
kubectl describe gateway envoy-gateway
openssl s_client -connect api.company.local:443
```

### Identification

- Verify certificate status.
- Check expiry date.
- Confirm DNS names.
- Verify TLS Secret.

### Corrective Action

- Renew certificate.
- Update TLS Secret.
- Configure trusted CA.
- Correct Gateway TLS configuration.

---

# Scenario 4

## Problem

```
One route works

Another returns

404 Not Found
```

### Possible Root Causes

- Missing HTTPRoute
- Incorrect hostname
- Wrong path
- Backend Service missing
- Route not attached

### Commands

```bash
kubectl get httproute
kubectl describe httproute
kubectl get gateway
kubectl describe gateway
kubectl get svc
```

### Identification

- Verify Host and Path.
- Check backend Service.
- Confirm HTTPRoute attachment.

### Corrective Action

- Correct Host or Path.
- Update HTTPRoute.
- Reapply configuration.

---

# Scenario 5

## Problem

```
Gateway becomes inaccessible after deploying a new HTTPRoute.
```

### Investigation Steps

1. Check the new HTTPRoute.
2. Verify Gateway status.
3. Review Gateway events.
4. Inspect Envoy logs.
5. Check for duplicate hostnames.
6. Validate YAML configuration.
7. Compare with the previous working route.

### Commands

```bash
kubectl get httproute
kubectl describe httproute
kubectl get gateway
kubectl describe gateway
kubectl get events
kubectl logs -n envoy-gateway-system deployment/envoy-gateway
kubectl rollout restart deployment envoy-gateway -n envoy-gateway-system
```

### Corrective Action

- Remove the faulty HTTPRoute.
- Fix YAML syntax.
- Resolve routing conflicts.
- Redeploy the corrected configuration.
- Restart Envoy Gateway if required.

---

# Conclusion

Effective troubleshooting in Kubernetes requires checking Gateway configuration, HTTPRoutes, Services, Endpoints, Pods, TLS certificates, DNS, and Envoy logs. Tools such as `kubectl`, `curl`, `nslookup`, and `openssl` help identify the root cause and restore normal application connectivity.
