---
title: "Install ASM"
weight: 2
---

Download `asmcli`:
```Bash
curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.12 > ~/asmcli
chmod +x ~/asmcli
```

Run the `asmcli install` command:
```Bash
cat <<EOF > distroless-proxy.yaml
---
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  meshConfig:
    defaultConfig:
      image:
        imageType: distroless
EOF
~/asmcli install \
  --project_id $PROJECT_ID \
  --cluster_name $CLUSTER_NAME \
  --cluster_location $CLUSTER_ZONE \
  --enable-all \
  --option cloud-tracing \
  --option cni-gcp \
  --custom_overlay distroless-proxy.yaml
```

Official resources:
- [Install ASM](https://cloud.google.com/service-mesh/docs/unified-install/install)