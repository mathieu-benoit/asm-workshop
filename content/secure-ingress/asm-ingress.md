---
title: "Setup Ingress Gateway"
weight: 3
---

{{%expand%}}
```yaml
apiVersion: v1
kind: Service
metadata:
  name: asm-ingressgateway
  annotations:
    cloud.google.com/neg: '{"ingress": true}'
    cloud.google.com/backend-config: '{"default": "asm-ingressgateway"}'
  labels:
    asm: ingressgateway
spec:
  ports:
  - name: status-port
    port: 15021
    protocol: TCP
    targetPort: 15021
  - name: http2
    port: 80
    targetPort: 8081
  - name: https
    port: 443
    targetPort: 8443
  selector:
    asm: ingressgateway
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: asm-ingressgateway
spec:
  selector:
    matchLabels:
      asm: ingressgateway
  template:
    metadata:
      annotations:
        inject.istio.io/templates: gateway
      labels:
        asm: ingressgateway
    spec:
      containers:
      - name: istio-proxy
        image: auto
---
apiVersion: cloud.google.com/v1
kind: BackendConfig
metadata:
  name: asm-ingressgateway
spec:
  healthCheck:
    requestPath: /healthz/ready
    port: 15021
    type: HTTP
  securityPolicy:
    name: SECURITY_POLICY
---
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: asm-ingressgateway
spec:
  domains:
    - "HOST_NAME"
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: asm-ingressgateway
  annotations:
    kubernetes.io/ingress.allow-http: "false"
    kubernetes.io/ingress.global-static-ip-name: "IP_NAME"
    networking.gke.io/managed-certificates: "asm-ingressgateway"
    kubernetes.io/ingress.class: "gce"
spec:
  defaultBackend:
    service:
      name: asm-ingressgateway
      port:
        number: 443
  rules:
  - http:
      paths:
      - path: /*
        pathType: ImplementationSpecific
        backend:
          service:
            name: asm-ingressgateway
            port:
              number: 443
---
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: asm-ingressgateway
spec:
  selector:
    asm: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
```
{{% /expand%}}


- Simple deployment?
- Secure/unprivileged deployment
- Rk: about internal or PSC for the endpoint
- Ref about the e-2-m doc