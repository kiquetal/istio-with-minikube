# Steps to install Istio and deploy httpbin

## 1. Install Istio

### 1.1. Add the Istio Helm repository

```bash
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```

### 1.2. Create a namespace for Istio

```bash
kubectl create namespace istio-system
```

### 1.3. Install the Istio base chart

```bash
helm install istio-base istio/base -n istio-system --wait
```

### 1.4. Install the Istiod control plane

```bash
helm install istiod istio/istiod -n istio-system --wait

### 1.5. Verify istioctl and kubectl compatibility

```bash
istioctl version
```
```

## 2. Install the Istio Gateway

```bash
kubectl create namespace istio-ingress
helm install istio-ingress istio/gateway -n istio-ingress --wait
```

## 3. Enable sidecar injection

```bash
kubectl create namespace apps
kubectl label namespace apps istio-injection=enabled
```

## 4. Deploy httpbin

### 4.1. Create the httpbin deployment and service

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: httpbin
  namespace: apps
  labels:
    app: httpbin
    service: httpbin
spec:
  ports:
  - name: http
    port: 8000
    targetPort: 80
  selector:
    app: httpbin
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpbin
  namespace: apps
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpbin
      version: v1
  template:
    metadata:
      labels:
        app: httpbin
        version: v1
    spec:
      containers:
      - image: docker.io/kennethreitz/httpbin
        name: httpbin
        ports:
        - containerPort: 80
EOF
```

### 4.2. Create a Gateway and VirtualService for httpbin

```bash
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: httpbin-gateway
  namespace: apps
spec:
  selector:
    istio: ingressgateway # use istio default ingress gateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
  namespace: apps
spec:
  hosts:
  - "*"
  gateways:
  - httpbin-gateway
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: httpbin
        port:
          number: 8000
EOF
```

## 5. Test the httpbin service

### 5.1. Port-forward the ingress gateway

```bash
kubectl port-forward -n istio-ingress "$(kubectl get pod -n istio-ingress -l app=istio-ingress -o jsonpath='{.items[0].metadata.name}')" 8080:80
```

### 5.2. Access the httpbin service

Now you can access the httpbin service in your browser at [http://localhost:8080](http://localhost:8080) or with curl:

```bash
curl http://localhost:8080/headers
```

## 6. Troubleshooting

### 6.1. Set istio-proxy logging level to debug

To set the logging level of the istio-proxy to debug for a specific pod (e.g., httpbin), you can use the following command. This change is not persistent across pod restarts.

```bash
kubectl exec -it "$(kubectl get pod -n apps -l app=httpbin -o jsonpath='{.items[0].metadata.name}')" -n apps -c istio-proxy -- curl -X POST "http://localhost:15000/logging?level=debug"
```
