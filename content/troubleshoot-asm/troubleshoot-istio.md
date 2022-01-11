---
title: "Troubleshoot Istio/ASM"
weight: 2
---

```Bash
kubectl get events
```

```Bash
istioctl analyze -A
```

```Bash
istioctl proxy-status
```

```Bash
NAMESPACE=your-namespace
DEPLOYMENT_NAME=your-deployment-name
kubectl logs deployment/$DEPLOYMENT_NAME -c istio-proxy -n $NAMESPACE
```

```Bash
NAMESPACE=your-namespace
DEPLOYMENT_NAME=your-deployment-name
APP_LABEL=your-pod-app-label
istioctl proxy-config log -l app=$APP_LABEL --level none -n $NAMESPACE
kubectl logs deployment/$DEPLOYMENT_NAME -c istio-proxy -n $NAMESPACE
```

```Bash
NAMESPACE=your-namespace
APP_LABEL=your-pod-app-label
istioctl proxy-config clusters $(kubectl -n $NAMESPACE get pod -l app=$APP_LABEL -o jsonpath={.items..metadata.name}) \
    -n $NAMESPACE
```

```Bash
NAMESPACE=your-namespace
APP_LABEL=your-pod-app-label
istioctl proxy-config listeners $(kubectl -n $NAMESPACE get pod -l app=$APP_LABEL -o jsonpath={.items..metadata.name}) \
    -n $NAMESPACE
```

```Bash
ASM_VERSION=$(kubectl get deploy -n istio-system -l app=istiod -o jsonpath={.items[*].metadata.labels.'istio\.io\/rev'}'{"\n"}')
kubectl describe configmap istio-$ASM_VERSION -n istio-system
```

Official resources:
- [Troubleshooting ASM](https://cloud.google.com/service-mesh/docs/troubleshooting/troubleshoot-intro)