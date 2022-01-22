---
title: "Secure Memorystore (redis) access"
weight: 1
---
In this section, you will secure the access by TLS to the Memorystore (redis) instance from the OnlineBoutique's `cartservice` application.


Create a new instance with in-transit encryption enabled:
```Bash
export REDIS_TLS_NAME=redis-tls
gcloud redis instances create $REDIS_TLS_NAME --size=1 --region=$REGION --zone=$ZONE --redis-version=redis_6_x --transit-encryption-mode=SERVER_AUTHENTICATION
```

Once the new Memorystore (redis) instance created, capture the following variables:
```Bash
export REDIS_TLS_IP=$(gcloud redis instances describe $REDIS_TLS_NAME --region=$REGION --format='get(host)')
export REDIS_TLS_PORT=$(gcloud redis instances describe $REDIS_TLS_NAME --region=$REGION --format='get(port)')
gcloud redis instances describe $REDIS_TLS_NAME --region=$REGION --format='get(serverCaCerts[0].cert)' > ~/$WORKING_DIRECTORY/redis-cert.pem
```

Create the `Secret` with the Certificate Authority:
```Bash
export REDIS_TLS_CERT_NAME=redis-cert
kubectl create secret generic $REDIS_TLS_CERT_NAME --from-file=$REDIS_TLS_CERT_NAME.pem -n $ONLINEBOUTIQUE_NAMESPACE
```

Create the `ServiceEntry` and `DestinationRule`:
```Bash
INTERNAL_HOST=cart.memorystore-redis.onlineboutique
cat <<EOF | kubectl apply -n $ONLINEBOUTIQUE_NAMESPACE -f -
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: memorystore-redis-tls
spec:
  hosts:
  - ${INTERNAL_HOST}
  addresses:
  - $REDIS_IP/32
  endpoints:
  - address: $REDIS_IP
  location: MESH_EXTERNAL
  resolution: STATIC
  ports:
  - number: $REDIS_PORT
    name: tcp-redis
    protocol: TCP
EOF
cat <<EOF | kubectl apply -n $ONLINEBOUTIQUE_NAMESPACE -f -
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: memorystore-redis-tls
spec:
  exportTo:
  - '.'
  host: ${INTERNAL_HOST}
  trafficPolicy:
    tls:
      mode: SIMPLE
      caCertificates: /etc/certs/${REDIS_TLS_CERT_NAME}.pem
EOF
```



Check that the new internal endpoint is listed as `cart.memorystore-redis.onlineboutique - 6378 - outbound - EDS - memorystore-redis-tls.onlineboutique` by running this command:
```Bash
istioctl proxy-config clusters $(kubectl -n $ONLINEBOUTIQUE_NAMESPACE get pod -l app=cartservice -o jsonpath={.items..metadata.name}) -n $ONLINEBOUTIQUE_NAMESPACE
```