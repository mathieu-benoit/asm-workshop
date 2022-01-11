---
title: "Deploy Ingress Gateway"
weight: 1
---

In this section, you will deploy the Ingress Gateway in its own namespace as you will do for any other workload.

```Bash
export INGRESS_GATEWAY_NAMESPACE=asm-ingress
export INGRESS_GATEWAY_NAME=asm-ingressgateway
export INGRESS_GATEWAY_LABEL="asm: ingressgateway"
kubectl create namespace $INGRESS_GATEWAY_NAMESPACE
ASM_VERSION=$(kubectl get deploy -n istio-system -l app=istiod -o jsonpath={.items[*].metadata.labels.'istio\.io\/rev'}'{"\n"}')
kubectl label namespace $INGRESS_GATEWAY_NAMESPACE istio-injection- istio.io/rev=$ASM_VERSION --overwrite
```

```Bash
cat <<EOF | kubectl apply -n $INGRESS_GATEWAY_NAMESPACE -f -
apiVersion: v1
kind: Service
metadata:
  name: ${INGRESS_GATEWAY_NAME}
spec:
  type: LoadBalancer
  selector:
    ${INGRESS_GATEWAY_LABEL}
  ports:
  - port: 80
    name: http
  - port: 443
    name: https
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${INGRESS_GATEWAY_NAME}
spec:
  selector:
    matchLabels:
      ${INGRESS_GATEWAY_LABEL}
  template:
    metadata:
      annotations:
        # Select the gateway injection template (rather than the default sidecar template)
        inject.istio.io/templates: gateway
      labels:
        # Set a unique label for the gateway. This is required to ensure Gateways can select this workload
        ${INGRESS_GATEWAY_LABEL}
        # Enable gateway injection. If connecting to a revisioned control plane, replace with "istio.io/rev: revision-name"
        sidecar.istio.io/inject: "true"
    spec:
      containers:
      - name: istio-proxy
        image: auto # The image will automatically update each time the pod starts.
---
# Set up roles to allow reading credentials for TLS
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: ${INGRESS_GATEWAY_NAME}
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ${INGRESS_GATEWAY_NAME}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: ${INGRESS_GATEWAY_NAME}
subjects:
- kind: ServiceAccount
  name: default
EOF
```

Check that the pod is properly deployed and that you got the associated public IP:
```Bash
kubectl wait --for=condition=available --timeout=600s deployment --all -n $INGRESS_GATEWAY_NAMESPACE
kubectl get svc $INGRESS_GATEWAY_NAME -n $INGRESS_GATEWAY_NAMESPACE -o jsonpath="{.status.loadBalancer.ingress[*].ip}"
```

Official resources:
- [Istio - Installing Gateways](https://istio.io/latest/docs/setup/additional-setup/gateway)
- [Docs - ASM Installing and upgrading gateways](https://cloud.google.com/service-mesh/docs/gateways)