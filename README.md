# Building Platforms on top of Kubernetes

[KubeCon Shanghai 2023 Schedule](https://sched.co/1PTHH)

![1695799566039](https://github.com/salaboy/kubecon-china-2023/assets/271966/9f03f106-d80f-4ba0-b880-4b8b796b5e17)

This short tutorial will use KServe, Knative, Istio, and Dapr to build a simple but powerful platform. 


## Installation

Create a KinD Cluster by Running: 

```shell
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

```shell
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

You can list all the application services and the inference model by querying the Knative Services in the cluster: 

```shell
> kubectl get ksvc
NAME                     URL                                                        LATESTCREATED                  LATESTREADY                    READY   REASON
agenda-service           http://agenda-service.default.127.0.0.1.sslip.io           agenda-service-00001           agenda-service-00001           True    
c4p-service              http://c4p-service.default.127.0.0.1.sslip.io              c4p-service-00001              c4p-service-00001              True    
frontend                 http://frontend.default.127.0.0.1.sslip.io                 frontend-00001                 frontend-00001                 True    
notifications-service    http://notifications-service.default.127.0.0.1.sslip.io    notifications-service-00001    notifications-service-00001    True    
sklearn-iris-predictor   http://sklearn-iris-predictor.default.127.0.0.1.sslip.io   sklearn-iris-predictor-00001   sklearn-iris-predictor-00001   True    

```

If you list the pods before using the application, you should see that the `c4p-service` will be downscaled to zero if it is not used. 

```shell
> kubectl get pods
NAME                                                       READY   STATUS    RESTARTS      AGE
agenda-service-00001-deployment-9c5c6dcc4-nv9jj            3/3     Running   1 (19h ago)   19h
conference-kafka-0                                         1/1     Running   0             19h
conference-postgresql-0                                    1/1     Running   0             19h
conference-redis-master-0                                  1/1     Running   0             19h
flagd-6bbdc5d999-8xpdn                                     1/1     Running   0             19h
frontend-00001-deployment-687d5c985f-t7c8w                 3/3     Running   0             19h
notifications-service-00001-deployment-9d85979d-767vp      3/3     Running   1 (19h ago)   19h
sklearn-iris-predictor-00001-deployment-779857d577-552qr   2/2     Running   0             19h
```

You will also notice that all application Pods have three containers running because we use Dapr for the application's services to connect to the infrastructure (Redis and Kafka). So, one container for the application logic, one for Knative Serving (queue-proxy, for auto-scaling), and one for Dapr Sidecar to inject application-level APIs. 
The Agenda, Notification, and Frontend services include no dependencies on Redis or Kafka in the [source code](https://github.com/salaboy/platforms-on-k8s/tree/v2.0.0/conference-application).  

Interacting with the inference model is easy because it is automatically exposed for application developers to consume on an HTTP or HTTPS endpoint. We are using the [SKlearn model provided by Kserve](https://kserve.github.io/website/0.7/modelserving/v1beta1/sklearn/v2/#testing-deployed-model) 

If you have [`httpie`](https://httpie.io/) installed, you can send a request by running: 

```shell
http POST http://sklearn-iris-predictor.default.127.0.0.1.sslip.io/v2/models/sklearn-iris/infer < resources/inputs-v2.json
```
You should see: 

```
HTTP/1.1 200 OK
content-length: 206
content-type: application/json
date: Wed, 27 Sep 2023 23:56:26 GMT
server: istio-envoy
x-envoy-upstream-service-time: 2

{
    "id": "327bf30f-e1d4-4cb5-b1da-da4279d87c45",
    "model_name": "sklearn-iris",
    "model_version": null,
    "outputs": [
        {
            "data": [
                1,
                1
            ],
            "datatype": "INT32",
            "name": "output-0",
            "parameters": null,
            "shape": [
                2
            ]
        }
    ],
    "parameters": null
}

```


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


