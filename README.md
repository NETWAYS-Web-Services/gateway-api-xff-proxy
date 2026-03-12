# kgateway — Gateway API XFF & Proxy Protocol Examples

This repository contains example configurations for [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) using [kgateway](https://kgateway.dev/), demonstrating different approaches to client IP preservation when traffic passes through load balancers.

## Overview

Three configuration sets are provided:

| Directory | Purpose |
|-----------|---------|
| `default/` | Basic gateway setup without any special IP handling |
| `xff/` | X-Forwarded-For header injection via OpenStack load balancer annotation |
| `proxy/` | Proxy Protocol support via OpenStack load balancer annotation |

## Prerequisites

- A Kubernetes cluster with the [Gateway API CRDs](https://gateway-api.sigs.k8s.io/guides/#installing-gateway-api) installed
- [kgateway](https://kgateway.dev/) deployed in the cluster (controller: `kgateway.dev/kgateway`)
- kgateway custom CRDs: `GatewayParameters` and `ListenerPolicy` (`gateway.kgateway.dev/v1alpha1`)
- An OpenStack-backed load balancer (for `xff/` and `proxy/` configurations)

## Demo Application

Deploy `mendhak/http-https-echo` to the `default` Namespace and make it available via Service for later testing:

```sh
kubectl run gw-test --image mendhak/http-https-echo:latest
kubectl expose pod gw-test --port 80 --target-port 8080
``` 


## Usage

Apply the configuration set that matches your requirements:

```bash
# Basic gateway (no IP preservation)
kubectl apply -f default/

# X-Forwarded-For header injection
kubectl apply -f xff/

# Proxy Protocol
kubectl apply -f proxy/
```

### Verify the deployment

```bash
kubectl get gatewayclass
kubectl get gateway -n kgateway-system
kubectl get httproute -n default
```

## Configuration Details

### Default

A minimal Gateway and HTTPRoute to route traffic to the `gw-test` backend. No custom `GatewayClass` or `GatewayParameters` are needed.

### X-Forwarded-For (`xff/`)

Adds the OpenStack load balancer annotation `loadbalancer.openstack.org/x-forwarded-for: "true"` to the Gateway's underlying Service via a `GatewayParameters` resource. This causes the load balancer to inject an `X-Forwarded-For` header with the original client IP.

Resources created:
- `GatewayClass` — references the `xff-gateway-params` GatewayParameters
- `GatewayParameters` — injects the XFF annotation on the provisioned load balancer Service
- `Gateway` — HTTP listener on port 80
- `HTTPRoute` — routes to the `gw-test` backend

### Proxy Protocol (`proxy/`)

Enables [Proxy Protocol](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt) on the load balancer via the OpenStack annotation `loadbalancer.openstack.org/proxy-protocol: "true"`. A `ListenerPolicy` is applied to the Gateway listener to instruct kgateway to expect and parse Proxy Protocol headers.

Resources created:
- `GatewayClass` — references the `proxy-gateway-params` GatewayParameters
- `GatewayParameters` — injects the proxy protocol annotation on the provisioned load balancer Service
- `Gateway` — HTTP listener on port 80
- `HTTPRoute` — routes to the `gw-test` backend
- `ListenerPolicy` — enables Proxy Protocol parsing on the Gateway listener

## License

MIT
