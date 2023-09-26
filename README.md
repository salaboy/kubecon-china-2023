# Building Platforms - Devs & Data Scientist

This short tutorial will use KServe, Knative, Istio, and Dapr to build a simple but powerful platform. 


## Installation

Create a KinD Cluster by Running: 


```
cat <<EOF | kind create cluster --name dev --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 31080 # expose port 31380 of the node to port 80 on the host, later to be use by kourier or contour ingress
    listenAddress: 127.0.0.1
    hostPort: 80
EOF

```
Install all the demo components running:

```
./install/install.sh
```

This script installs and configures: 
- [Knative Serving](https://knative.dev)
- [Istio](https://istio.io/)
- [Dapr](https://dapr.io)
- [KServe](https://kserve.github.io/website/)
- [Conference Application](https://github.com/salaboy/platforms-on-k8s)
- Inference Model

## Accessing the Application

You can access the application by pointing your browser to this address [http://frontend.default.127.0.0.1.sslip.io](http://frontend.default.127.0.0.1.sslip.io).




## Clean up

Delete the KinD cluster by running: 
```
kind delete clusters dev
```
