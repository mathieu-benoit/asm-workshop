---
title: "Deploy Ingress Gateway"
weight: 1
---

In this section, you will deploy the Ingress Gateway in its own namespace as you will do for any other workload.

```Bash
export INGRESS_GATEWAY_NAMESPACE=asm-ingress
kubectl create namespace $INGRESS_GATEWAY_NAMESPACE
ASM_VERSION=$(kubectl get deploy -n istio-system -l app=istiod -o jsonpath={.items[*].metadata.labels.'istio\.io\/rev'}'{"\n"}')
kubectl label namespace $INGRESS_GATEWAY_NAMESPACE istio-injection- istio.io/rev=$ASM_VERSION --overwrite
```

FIXME - grab and deploy deployment, service, etc.

Official resources:
- [Istio - Installing Gateways](https://istio.io/latest/docs/setup/additional-setup/gateway)
- [Docs - ASM Installing and upgrading gateways](https://cloud.google.com/service-mesh/docs/gateways)