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

Ensure that all deployments are up and running:
```Bash
kubectl wait --for=condition=available --timeout=600s deployment --all -n istio-system
kubectl wait --for=condition=available --timeout=600s deployment --all -n asm-system
```

_Not part of the workshop, but here below is the routine when you will need to run to [upgrade to a newer version of ASM](https://cloud.google.com/service-mesh/docs/unified-install/plan-upgrade):_
```Bash
# Grab the current ASM version before upgrading to the new version
OLD_ASM_VERSION=$(kubectl get deploy -n istio-system -l app=istiod -o jsonpath={.items[*].metadata.labels.'istio\.io\/rev'}'{"\n"}')
# Doownload the new version of the `asmcli` tool
curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.12 > ~/asmcli
chmod +x ~/asmcli
# Run the same `asmcli install` command we ran for the installation
~/asmcli install...
# Update the asm/istio labels of your namespaces
kubectl rollout restart deployments -n FIXME
# Once you have verified that your workloads and ASM is working properly, you could complete the upgrade of ASM by removing the the components of the old version
kubectl delete Service,Deployment,HorizontalPodAutoscaler,PodDisruptionBudget istiod-$OLD_ASM_VERSION -n istio-system --ignore-not-found=true
kubectl delete IstioOperator installed-state-$OLD_ASM_VERSION -n istio-system
```

Resources:
- [Install ASM](https://cloud.google.com/service-mesh/docs/unified-install/install)