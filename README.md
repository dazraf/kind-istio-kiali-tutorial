# Tutorial 

This tutorial will guide you through every step needed to:
- Create a local Kubernetes cluster using kind
- Install Istio and all required observability tools
- Deploy the BookInfo sample app
- Generate traffic and visualize service dependencies in Kiali

## Prerequisites & Installation
This guide assumes a Linux system. If you are missing any tools, follow these steps:

### 1. Install Docker
Follow the official instructions: https://docs.docker.com/engine/install/
Or, for Ubuntu:
```sh
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```
Log out and back in for group changes to take effect.

### 2. Install kubectl
```sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
rm kubectl
```

### 3. Install kind
```sh
curl -Lo ./kind "https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64"
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

### 4. Install istioctl
```sh
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
sudo cp bin/istioctl /usr/local/bin/
cd ..
```

---

## 1. Create a kind Cluster
Creates a local Kubernetes cluster named `istio-demo` using kind.
```sh
kind create cluster --name istio-demo
```

---

## 2. Install Istio
Installs Istio with the demo profile and enables automatic sidecar injection in the default namespace.
```sh
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled
```

---

## 3. Deploy the BookInfo Sample App
Deploys the BookInfo microservices demo application to your cluster.
```sh
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.26/samples/bookinfo/platform/kube/bookinfo.yaml
```

---

## 4. Verify the Deployment
Check that all BookInfo services and pods are running and ready.
```sh
kubectl get services
kubectl get pods
kubectl wait --for=condition=Ready pods --all --timeout=180s
```

---

## 5. Install Observability Addons
Kiali (for service graph) and Prometheus (for metrics) are required for full visualization.
```sh
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.26/samples/addons/kiali.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.26/samples/addons/prometheus.yaml
```
Wait for the pods to be ready:
```sh
kubectl wait --for=condition=Ready pods -l app=kiali -n istio-system --timeout=120s
kubectl wait --for=condition=Ready pods -l app=prometheus -n istio-system --timeout=120s
```

---

## 6. Access the Kiali Dashboard
Port-forward the Kiali service and open the dashboard in your browser.
```sh
kubectl port-forward svc/kiali -n istio-system 20001:20001 &
```
Then open [http://localhost:20001/kiali](http://localhost:20001/kiali) in your browser. (Default login is usually `admin/admin` or no login required.)

---

## 7. Generate Traffic for BookInfo
Kiali visualizes service dependencies based on real traffic. To generate traffic:

### Option 1: Manual (Browser)
Port-forward the productpage service and visit it in your browser:
```sh
kubectl port-forward svc/productpage 9080:9080 &
```
Open [http://localhost:9080/productpage](http://localhost:9080/productpage) and refresh the page several times.

### Option 2: Automated (Terminal)
Run this command to continuously generate traffic:
```sh
while true; do curl -s http://localhost:9080/productpage > /dev/null; sleep 1; done
```

After a minute or two, check the Kiali Traffic Graph for a live view of service interactions.

---

## 8. Cleanup
To delete everything and free resources:
```sh
kind delete cluster --name istio-demo
```

---

## Troubleshooting
- If Kiali shows errors about Prometheus, ensure Prometheus is installed and running in the `istio-system` namespace.
- If BookInfo pods are not ready, check pod logs for errors: `kubectl logs <pod-name>`
- For more details, see the [Istio BookInfo docs](https://istio.io/latest/docs/examples/bookinfo/).
