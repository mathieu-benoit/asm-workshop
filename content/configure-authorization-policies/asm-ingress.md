---
title: "Configure AuthorizationPolicy for Ingress Gateway"
weight: 1
---
In this section we will configure `AuthorizationPolicy` for the Ingress Gateway namespace.

Deploy fine granular `AuthorizationPolicy` per app:
```Bash
cat <<EOF | kubectl apply -n $INGRESS_GATEWAY_NAMESPACE -f -
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
spec:
  {}
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: ${INGRESS_GATEWAY_NAME}
spec:
  selector:
    matchLabels:
      app: ${INGRESS_GATEWAY_NAME}
  rules:
  - to:
      - operation:
          ports: ["8080"]
EOF
```