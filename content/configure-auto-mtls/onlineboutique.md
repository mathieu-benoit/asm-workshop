---
title: "Configure auto mTLS STRICT for OnlineBoutique"
weight: 2
---

{{%expand%}}
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT
```
{{% /expand%}}