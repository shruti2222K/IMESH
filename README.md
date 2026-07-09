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
