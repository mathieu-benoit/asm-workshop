---
title: "Configure the public IP and DNS"
weight: 2
---


```Bash
export PUBLIC_IP_NAME=$clusterName-asm-ingressgateway # Name hard-coded there: https://github.com/mathieu-benoit/my-kubernetes-deployments/tree/main/namespaces/asm-ingress/ingress.yaml
gcloud compute addresses create $PUBLIC_IP_NAME \
    --global
export PUBLIC_IP_ADDRESS=$(gcloud compute addresses describe $PUBLIC_IP_NAME --global --format "value(address)")
echo ${PUBLIC_IP_ADDRESS}
```

```Bash
cat <<EOF > dns-spec.yaml
swagger: "2.0"
info:
  description: "Cloud Endpoints DNS"
  title: "Cloud Endpoints DNS"
  version: "1.0.0"
paths: {}
host: "frontend.endpoints.${PROJECT_ID}.cloud.goog"
x-google-endpoints:
- name: "frontend.endpoints.${PROJECT_ID}.cloud.goog"
  target: "${GCLB_IP}"
EOF
gcloud endpoints services deploy dns-spec.yaml
```