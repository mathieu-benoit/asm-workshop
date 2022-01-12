---
title: "Configure AuthorizationPolicy for OnlineBoutique"
weight: 2
---
In this section we will configure `AuthorizationPolicy` for the OnlineBoutique namespace.

Deploy fine granular `AuthorizationPolicy` per app:
```Bash
cat <<EOF | kubectl apply -n $ONLINEBOUTIQUE_NAMESPACE -f -
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
spec:
  {}
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: adservice
spec:
  selector:
    matchLabels:
      app: adservice
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/frontend"]
    to:
      - operation:
          paths: ["/hipstershop.AdService/GetAds"]
          methods: ["POST"]
          ports: ["9555"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: cartservice
spec:
  selector:
    matchLabels:
      app: cartservice
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/frontend", "cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/checkoutservice"]
    to:
      - operation:
          paths: ["/hipstershop.CartService/AddItem", "/hipstershop.CartService/GetCart", "/hipstershop.CartService/EmptyCart"]
          methods: ["POST"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: checkoutservice
spec:
  selector:
    matchLabels:
      app: checkoutservice
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/frontend"]
    to:
      - operation:
          paths: ["/hipstershop.CheckoutService/PlaceOrder"]
          methods: ["POST"]
          ports: ["5050"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: currencyservice
spec:
  selector:
    matchLabels:
      app: currencyservice
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/frontend", "cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/checkoutservice"]
    to:
      - operation:
          paths: ["/hipstershop.CurrencyService/Convert", "/hipstershop.CurrencyService/GetSupportedCurrencies"]
          methods: ["POST"]
          ports: ["7000"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: emailservice
spec:
  selector:
    matchLabels:
      app: emailservice
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/checkoutservice"]
    to:
      - operation:
          paths: ["/hipstershop.EmailService/SendOrderConfirmation"]
          methods: ["POST"]
          ports: ["8080"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: frontend
spec:
  selector:
    matchLabels:
      app: frontend
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/loadgenerator", "cluster.local/ns/${INGRESS_GATEWAY_NAMESPACE}/sa/${INGRESS_GATEWAY_NAME}"]
    to:
      - operation:
          ports: ["8080"]
          methods: ["GET", "POST"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: paymentservice
spec:
  selector:
    matchLabels:
      app: paymentservice
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/checkoutservice"]
    to:
      - operation:
          paths: ["/hipstershop.PaymentService/Charge"]
          methods: ["POST"]
          ports: ["50051"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: productcatalogservice
spec:
  selector:
    matchLabels:
      app: productcatalogservice
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/frontend", "cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/checkoutservice", "cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/recommendationservice"]
    to:
      - operation:
          paths: ["/hipstershop.ProductCatalogService/GetProduct", "/hipstershop.ProductCatalogService/ListProducts"]
          methods: ["POST"]
          ports: ["3550"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: recommendationservice
spec:
  selector:
    matchLabels:
      app: recommendationservice
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/frontend"]
    to:
      - operation:
          paths: ["/hipstershop.RecommendationService/ListRecommendations"]
          methods: ["POST"]
          ports: ["8080"]
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: shippingservice
spec:
  selector:
    matchLabels:
      app: shippingservice
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/frontend", "cluster.local/ns/${ONLINEBOUTIQUE_NAMESPACE}/sa/checkoutservice"]
    to:
      - operation:
          paths: ["/hipstershop.ShippingService/GetQuote", "/hipstershop.ShippingService/ShipOrder"]
          methods: ["POST"]
          ports: ["50051"]
EOF
```