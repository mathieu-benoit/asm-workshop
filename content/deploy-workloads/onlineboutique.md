---
title: "Deploy OnlineBoutique"
weight: 2
---
In this section, you will deploy the [OnlineBoutique](https://github.com/GoogleCloudPlatform/microservices-demo) apps as-is, without any notion of Istio nor ASM, not yet.

```Bash
export ONLINEBOUTIQUE_NAMESPACE=onlineboutique
kubectl create namespace $ONLINEBOUTIQUE_NAMESPACE
curl -LO https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/kubernetes-manifests.yaml > ~/$WORKING_DIRECTORY/onlineboutique.yaml
kubectl apply -f ~/$WORKING_DIRECTORY/onlineboutique.yaml -n $ONLINEBOUTIQUE_NAMESPACE
```

Ensure that all deployments are up and running:
```Bash
kubectl wait --for=condition=available --timeout=600s deployment --all -n $ONLINEBOUTIQUE_NAMESPACE
ONLINEBOUTIQUE_PUBLIC_IP=$(kubectl get svc frontend-external -n $ONLINEBOUTIQUE_NAMESPACE -o jsonpath="{.status.loadBalancer.ingress[*].ip}")
curl -s http://${ONLINEBOUTIQUE_PUBLIC_IP}
```

In order to be more secure and have more resilience with the data stored in `redis`, we will instead leverage Memorystore (redis) instead:
```Bash
gcloud services enable redis.googleapis.com
gcloud redis instances create cart --size=1 --region=$REGION --zone=$ZONE --redis-version=redis_6_X
REDIS_IP=$(gcloud redis instances describe cart --region=$REGION --format='get(host)')
sed -i "s/value: \"redis-cart:6379\"/value: \"$REDIS_IP\"/g" ~/$WORKING_DIRECTORY/onlineboutique.yaml
kubectl apply -f ~/$WORKING_DIRECTORY/onlineboutique.yaml -n $ONLINEBOUTIQUE_NAMESPACE
```

Ensure that the solution is still working correctly with Memorystore (redis):
```Bash
curl -s http://${ONLINEBOUTIQUE_PUBLIC_IP}
```

From there, the `redis` container originally deployed could now be deleted:
```Bash
kubectl delete deployment redis -n $ONLINEBOUTIQUE_NAMESPACE
kubectl delete service redis -n $ONLINEBOUTIQUE_NAMESPACE
```
{{% notice note %}}
You can connect to a Memorystore (redis) instance only from GKE clusters that are in the same region and use the same network as your instance. You cannot connect to a Memorystore (redis) instance from a GKE cluster without VPC-native/IP aliasing enabled. For this you should create a GKE cluster with this option `--enable-ip-alias`.
{{% /notice %}}

Here is the high-level setup you you just accomplished with this section:
![ASM Security diagram](/images/onlineboutique-initial.png)

Resources:
- [Tutorial - Deploying the Online Boutique sample application](https://cloud.google.com/service-mesh/docs/onlineboutique-install-kpt)
- [Connecting to Memorystore (redis) from GKE](https://cloud.google.com/memorystore/docs/redis/connect-redis-instance-gke)