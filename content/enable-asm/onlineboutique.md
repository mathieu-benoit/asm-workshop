---
title: "Enable ASM for OnlineBoutique"
weight: 1
---
In this section, you will enable ASM for OnlineBoutique.

Inject the Istio/ASM proxy within the OnlineBoutique namespace:
```Bash
ASM_VERSION=$(kubectl get deploy -n istio-system -l app=istiod -o jsonpath={.items[*].metadata.labels.'istio\.io\/rev'}'{"\n"}')
kubectl label namespace $ONLINEBOUTIQUE_NAMESPACE istio-injection- istio.io/rev=$ASM_VERSION --overwrite
kubectl rollout restart deployments -n $ONLINEBOUTIQUE_NAMESPACE
```

Test that 
```Bash

```

Route traffic through the OnlineBoutique's `frontend` app via the Ingress Gateway:
```Bash

```

Official resources:
- [ASM - Injecting sidecar proxies](https://cloud.google.com/service-mesh/docs/proxy-injection)