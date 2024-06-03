# AWS_EKS (Elastic Kubernetes Service)

- It is managed K8S offering.
  *For High availability of clusters, its suggested to create 3 worker and 3 master node architecture (6 EC2 instances)*
  
- After creation of 6 instances, we've to install configurations on master nodes like API server, etcd, scheduler, CCM, CM (control plane components) which are user facing components. Users will be talking to these components for any request from where request goes to data plane. After that, we've to install and configure data plane components after which we've to join worker nodes to control plane.

- Suppose, our one of the three master node goes down due to reasons like certificate expiry, API server is down, etcd getting crashed, scheduler not working. Even if our cluster is created, we need to handle all of these issues. This can make life of devops engineer very hectic if we've many clusters. Thats why we need to setup monitoring. 
  Thus AWS created EKS service which is managed control plane (not managed data plane).

- If we request AWS for K8S cluster, it provide us HA(High Availability) cluster with respect to control plane components. Everything in master is taken care by AWS platform. So if API server goes down, etcd gets crashed, we can just take help of AWS which takes care those issues doesn't happen.
- It also allows easy way to attach worker nodes with control plane.

- When we request for cluster, EKS will install all the master node and its components (we dont know where master node location is). When we attach worker nodes, we can create EC2 instances by ourself or we can choose fargate (AWS serverless compute just like lambda but lambda is for small workloads). Fargate is for running containers. If we use fargate with EKS, we dont have to worry about anything. EKS takes care of control plane, fargate takes care of worker nodes.

   EKS = Control Plane, Fargate = Data Plane
  
  This way we can build robust and highly stable k8s cluster using EKS.

  *_We can also use EC2 but we need to ensure HA. We've to install instances with auto scaling, we've to setup monitoring. This is not the case in fargate which itself takes care of worker nodes._*


 # How to Deploy application on AWS EKS Cluster and allow users to access it?

- Lets say we have 3 master nodes (M1,M2,M3) and 2 worker nodes (W1, W2)
- Suppose, our app is deployed onto pod on one of the worker node (we've created pod.yml and deployed it).
- Now this app inside pod will have cluster IP. So this pod can be accessed from anywhere in cluster (M1,M2,M3,W1) but end user cannot access it.
- So now first we'll create service for pod (create service.yml and deploy the service). Service has 3 options cluster IP, Node port, LB
  LB creates Elastic IP address using which users can access but it is very costly.
- Here best option apart from 3 modes is go with **_*"Ingress Resource"*_**
  Lets have master node and worker node. In master we've API server. On worker we have pod inside which app is deployed and also service is there.
  The service we'll restrict to cluster IP or Node Port mode both supported by ingress.
  On worker we'll create ingress resource which will allow user to access app inside EKS cluster (Ingress routes traffic inside EKS cluster)
  Devops engineer writes "ingress.yml" file where write allow user to access app example.com/abc, forward the request to service from where request goes to pod. This configuration is written in ingress resource. We'll deploy ingress resource using kubectl.
  But there has to be someone who must take user from outside to worker node as everything inside nodes will be in private subnet.
  So there is "Ingress Controller". Request from user can be taken to public subnet inside nodes but there has to be LB which takes request to private subnet and access the app. Typically all LB support Ingress Controllers
  As soon as we create ingress resource, ingress controller will watch for it and it creates ALB for us which watches ingress resources and whenever ingress resource is created, ALB controller created ALB env for us using which user can talk to app LB from where request will go to our app.

e.g:- If we're using nginx app. There will be nginx ingress controller which we deploy in K8S cluster and whenever nginx ingress controller finds ingress resource, nginx controller will either create nginx LB or if LB is already there it will configure the LB for the rules mentioned in ingress resource.

- We can define in ingress.yml that who should access ingress resource 

- DevOps engineer along with pod and service will create ingress for every resource or pod which need access from external world
  There will be one ingress controller which will watch for all ingress resources and it will configure LB
  External person will talk to LB and from LB which is in public subnet request will go service through pod.


- EKS is fully managed control plane of K8S using which we increase efficiency. If we're on AWS, there are 2 ways to install K8S :- Create VM and use tools to setup K8S cluster without EKS or go with EKS.

*Initially there was on premises services, then people moved to cloud to create everything by themselves and then they moved towards cloud managed services*


