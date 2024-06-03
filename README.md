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

- EKS is fully managed control plane of K8S using which we increase efficiency. If we're on AWS, there are 2 ways to install K8S :- Create VM and use tools to setup K8S cluster without EKS or go with EKS.

*Initially there was on premises services, then people moved to cloud to create everything by themselves and then they moved towards cloud managed services*


