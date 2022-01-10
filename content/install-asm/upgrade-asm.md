---
title: "Upgrade ASM"
weight: 3
---
This section is not part of the workshop, so you don't need to run it just yet. It's more about the process to have in place for your _Day 2 Operations_ with ASM.

Download new version of `asmcli`:
```Bash
curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.12 > ~/asmcli
chmod +x ~/asmcli
```

Grab the current ASM version before upgrading to the new version:
```
OLD_ASM_VERSION=$(kubectl get deploy -n istio-system -l app=istiod -o jsonpath={.items[*].metadata.labels.'istio\.io\/rev'}'{"\n"}')
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

FIXME - Update the asm/istio labels of your namespaces

```Bash
kubectl rollout restart deployments
```

Once you have verified that your workloads and ASM is working properly, you could complete the upgrade of ASM by removing the the components of the old version:
```Bash
kubectl delete Service,Deployment,HorizontalPodAutoscaler,PodDisruptionBudget istiod-$OLD_ASM_VERSION -n istio-system --ignore-not-found=true
kubectl delete IstioOperator installed-state-$OLD_ASM_VERSION -n istio-system
```

Official resources:
- [ASM Plan an upgrade](https://cloud.google.com/service-mesh/docs/unified-install/plan-upgrade)