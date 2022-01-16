---
title: "Install ASM"
weight: 2
---

Download `asmcli`:
```Bash
curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.12 > ~/asmcli
chmod +x ~/asmcli
```

Enable Managed ASM on your current project:
```Bash
gcloud container hub mesh enable
```

Run the `asmcli install` command:
```Bash
ASM_CHANNEL=rapid
ASM_LABEL=asm-managed
export ASM_VERSION=$ASM_LABEL-$ASM_CHANNEL
~/asmcli install \
  --project_id $PROJECT_ID \
  --cluster_name $GKE_NAME \
  --cluster_location $ZONE \
  --enable-all \
  --managed \
  --channel $ASM_CHANNEL \
  --use_managed_cni
```

Apply the following Mesh configs (`distroless` container image for the proxy and Cloud Tracing):
```Bash
cat <<EOF | kubectl apply -n istio-system -f -
apiVersion: v1
kind: Namespace
metadata:
  name: istio-system
---
apiVersion: v1
data:
  mesh: |-
    enableTracing: true
    defaultConfig:
      image:
        imageType: distroless
      tracing:
        stackdriver:{}
kind: ConfigMap
metadata:
  name: istio-${ASM_VERSION}
EOF
```

Ensure that all deployments are up and running:
```Bash
kubectl get controlplanerevision -n istio-system
kubectl get dataplanecontrols
kubectl get daemonset istio-cni-node -n kube-system
kubectl wait --for=condition=available --timeout=600s deployment --all -n asm-system
```

Get the version of the ASM Control Plane:
```Bash
kubectl get configmap env-$ASM_VERSION -n istio-system -o yaml | grep "TAG"
```

Resources:
- [ASM Release Notes](https://cloud.google.com/service-mesh/docs/release-notes)
- [Configure Managed Anthos Service Mesh](https://cloud.google.com/service-mesh/docs/managed/service-mesh)
- [Managed ASM Release Channel](https://cloud.google.com/service-mesh/docs/managed/release-channels)