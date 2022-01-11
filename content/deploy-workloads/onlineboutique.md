---
title: "Deploy OnlineBoutique"
weight: 2
---
In this section, you will deploy the [OnlineBoutique](https://github.com/GoogleCloudPlatform/microservices-demo) apps as-is, without any notion of Istio nor ASM, not yet.

```Bash
export ONLINEBOUTIQUE_NAMESPACE=onlineboutique
kubectl create namespace $ONLINEBOUTIQUE_NAMESPACE
curl -LO https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/kubernetes-manifests.yaml
kubectl apply -f kubernetes-manifests.yaml -n $ONLINEBOUTIQUE_NAMESPACE
```

Check that the pods are properly deployed and that you got the associated public IP:
```Bash
kubectl wait --for=condition=available --timeout=600s deployment --all -n $ONLINEBOUTIQUE_NAMESPACE
kubectl get svc frontend -n $ONLINEBOUTIQUE_NAMESPACE -o jsonpath="{.status.loadBalancer.ingress[*].ip}"
```

FIXME - add steps to replace redis by Memorystore

Official resources:
- [Tutorial - Deploying the Online Boutique sample application](https://cloud.google.com/service-mesh/docs/onlineboutique-install-kpt)