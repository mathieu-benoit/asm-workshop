---
title: "Include OnlineBoutique in Mesh"
weight: 1
---
In this section, you will include OnlineBoutique in Mesh.

```Bash
ASM_VERSION=$(kubectl get deploy -n istio-system -l app=istiod -o jsonpath={.items[*].metadata.labels.'istio\.io\/rev'}'{"\n"}')
kubectl label namespace $ONLINEBOUTIQUE_NAMESPACE istio-injection- istio.io/rev=$ASM_VERSION --overwrite
kubectl rollout restart deployments -n $ONLINEBOUTIQUE_NAMESPACE
```

FIXME - add the section with the link with the ingress gateway: virtualservice, gateway, etc.

Official resources:
- [ASM - Injecting sidecar proxies](https://cloud.google.com/service-mesh/docs/proxy-injection)