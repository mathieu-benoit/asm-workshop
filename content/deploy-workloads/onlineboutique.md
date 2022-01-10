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

Rk: wait for onlinboutique release tag > v0.3.4 to have all the profiler, trace, debugging, etc. environment variables disabled in the Kubernetes manifest under the release folder.

Official resources:
- [Tutorial - Deploying the Online Boutique sample application](https://cloud.google.com/service-mesh/docs/onlineboutique-install-kpt)