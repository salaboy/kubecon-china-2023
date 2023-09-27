# Building Platforms on top of Kubernetes

[KubeCon Shanghai 2023 Schedule](https://sched.co/1PTHH)

![1695799566039](https://github.com/salaboy/kubecon-china-2023/assets/271966/9f03f106-d80f-4ba0-b880-4b8b796b5e17)


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
<img width="800" alt="Screenshot 2023-09-28 at 07 39 37" src="https://github.com/salaboy/kubecon-china-2023/assets/271966/489f862f-c1e7-4207-a898-cb11e12d01f5">

## Clean up

Delete the KinD cluster by running: 
```
kind delete clusters dev
```

# Resources
- [Free step-by-step tutorials](https://github.com/salaboy/platforms-on-k8s/) (Chinese translations thanks to [@dustise](https://twitter.com/dustise) 🥳)
- [Platform Engineering on Kubernetes](https://www.salaboy.com/book/)
  - Chinese Translation coming in 2024 by [https://www.epubit.com/](https://www.epubit.com/)
- [TAG App Delivery Platforms White Paper](https://tag-app-delivery.cncf.io/whitepapers/platforms/) 
- [Building Bloomberg's ML Inference Platform Using KServe](https://www.bloomberg.com/company/stories/the-journey-to-build-bloombergs-ml-inference-platform-using-kserve-formerly-kfserving/ )
- [Provisioning and consuming Multi-Cloud Infrastructure](https://blog.crossplane.io/crossplane-and-dapr/) 
- [Dapr and Alibaba Cloud](https://blog.dapr.io/posts/2021/03/19/how-alibaba-is-using-dapr/)
- [Knative Functions - From code to service](https://www.youtube.com/watch?v=ExuyJUIl4CY)  
- [Red Light, Green Light: Traffic Security in the Service Mesh (Alexa Nicole Griffith & Zhenni Fu)](https://www.youtube.com/watch?v=f6jMix46ZD8)
- [Exploring ML Model Serving with KServe (with fun drawings) - Alexa Nicole Griffith, Bloomberg](https://www.youtube.com/watch?v=FX6naJLaq2Y)
- [The State & Future of Cloud Native Model Serving](https://www.youtube.com/watch?v=786VaGAfm6I)


