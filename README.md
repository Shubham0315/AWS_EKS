# AWS_EKS (Elastic Kubernetes Service)

- EKS is managed K8S offering.
- As we know K8S cluster has 2 components Control plane and data plane (single or multiple). For High availability of clusters, its suggested to create 3 worker and 3 master node architecture (6 EC2 instances)
  
- After creation of 6 instances, we've to install configurations on master nodes like API server, etcd, scheduler, CCM, CM (control plane components) which are user facing components. Users will be talking to these components for any request from where request goes to data plane. After that, as a devops engineer we've to install and configure data plane components after which we've to join worker nodes to control plane.
  - This process is very tedious and error prone.

- Suppose, our one of the three master node goes down due to reasons like certificate expiry, API server is down, etcd getting crashed, scheduler not working. Even if our cluster is created, we need to handle all of these issues. This can make life of devops engineer very hectic if we've many clusters. Thats why we need to setup monitoring. 
 - Thus AWS created EKS service which is managed control plane (not managed data plane) which is High Availability cluster (HA)
 - AWS EKS when requested for K8S cluster, it will give us highly available K8S cluster with control plane components.
 - It also provides easy way to integrate worker nodes with control plane
 - While attaching worker nodes, EKS gives 2 options - Use EC2 instances, fargate (server or serverless computes). Fargate are like lambda but lambda is for small workloads or quick actions. Fargate are responsible for running containers.
 - So if we use Fargate with EKS, we dont have to worry about any resources. EKS takes care of control plane, fargate takes care of worker nodes.
 - Fargate and EKS control plane are highly available services of AWS. This way we can build robust and highly stable k8s cluster using EKS.


EKS = Control Plane, Fargate = Data Plane
-

- We can also use EC2 but we need to ensure HA. We've to install instances with auto scaling, we've to setup monitoring. This is not the case in fargate which itself takes care of worker nodes.

- EKS is fully managed control plane of K8S using which we increase work efficiency. Here we dont need to worry about certificate expiry, API slowness, etcd getting crashed.
- If we're on AWS, there are 2 ways to install K8S :- Create VM and use tools like kops, kubeadm to setup K8S cluster without EKS or go with EKS.

- Initially there was on premises services, then people moved to cloud to create everything by themselves and then they moved towards cloud managed services. 

---------------------------------------------------------------------------------------------------------------------------------------------

 # How to Deploy application on AWS EKS Cluster and allow users to access it?

- Lets say we have 3 master nodes (M1,M2,M3) and 2 worker nodes (W1, W2)
- Suppose, our app is deployed onto pod creating pod.yml on one of the worker node W2 (we've created pod.yml and deployed it).
- Now this app inside pod will have cluster IP. So this pod can be accessed from anywhere in cluster (M1,M2,M3,W1) but end user cannot access it.
- So now first we'll create service for pod (create service.yml and deploy the service). Service has 3 options cluster IP(already using), Node port, LB mode exposing to public
  - Using cluster IP, the pod will be accessible only within from cluster
  - In node port, if we convert service to node-port model, the pod can be accessed from any of the IP addresses of EC2 instances. But K8S Cluster will be within VPC which will have public subnet and private subnet. Apps are always deployed in private subnet. So in nodeport mode apps will be accessible to people having access to private subnet only.
  - For outside people to access it, we need LB. LB creates Elastic IP address using which users can access but it is very costly.
  
- Here best option apart from 3 modes is go with **_*"Ingress Resource"*_**
- Lets have master node and worker node. In master we've API server. There will be a pod on worker node. Also service will be there on worker which will be cluster IP or nodeport. On worker we have pod inside which app is deployed and also service is deployed there. The service we'll restrict to cluster IP or Node Port mode both supported by ingress. 
- On worker we'll create ingress resource which will allow user to access app inside EKS cluster (Ingress routes traffic inside EKS cluster)

- Devops engineer will write _**"ingress.yml"**_ file where they define allow user to access app ON specific website , forward the request to service from where request goes to pod. This configuration is written in ingress resource. We'll deploy ingress resource using kubectl.
- Now ingress is deployed.
  
- But there has to be someone who must take user request from outside to worker node as user cannot access anything inside worker as everything inside nodes will be in private subnet. But here user can access things in public subnet. If we place LB in public subnet, user request goes to LB and then it will go inside pod.
- So there is _**"Ingress Controller"**_ which is supported by all LBs. Request from user can be taken to public subnet inside nodes but there has to be LB which takes request to private subnet and access the app. Typically all LB support Ingress Controllers.
- As soon as we create ingress resource, ingress controller will watch for it and it creates ALB (Application Load Balancer) controller for us which watches ingress resources and whenever ingress resource is created, ALB controller creates ALB env for us using which user can talk to app LB from where request will go to our app.

_**e.g:- If we're using nginx app. There will be nginx ingress controller which we deploy in K8S cluster and whenever nginx ingress controller finds ingress resource, nginx controller will either create nginx LB or if LB is already there it will configure the LB for the rules mentioned in ingress resource.**_

- In **ingress.yml** we can define, who should access ingress resource (IAM roles).

- DevOps engineer along with pod and service will create ingress for every resource or pod which need access from external world. There will be one ingress controller which will watch for all ingress resources and it will configure LB. External person will talk to LB and from LB which is in public subnet request will go through service to the pod.

---------------------------------------------------------------------------------------------------------------------------------------------

# Practical Demo

- Before creating EKS, we've to install below on laptop - kubectl, eksctl and AWS CLI to interact with EKS cluster. Kubectl is used to interact with K8S cluster created on EKS and eksctl is a command line utility to create EKS Cluster.
- Also download AWS CLI and configure it.
- For demo, use root account.
- To create Cluster, we can do from UI or CLI. In organizations we can use eksctl CLI to create EKS cluster
  - Choose fargate if our organization doesnt have specific requirements as worker node to be on RHEL only or any other, it need to have speceific things. Now use fargate
  - Command :-  **eksctl create cluster --name demo-cluster --region us-east-1 --fargate**

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/c26d32bd-6b6e-46ca-856c-6aefd64ae8b0)

- Here eksctl is creating everything for us which is there in UI like service roles, networking (public or private subnets). Here eksctl creates a public-private subnet for us. Within VPC it creates private and public subnets and in private subnet we will place our apps. To run this command and create cluster it takes 10-15 mins. Control plane will be ready once cluster is created
- If now we see in AWS Console, in our region, cluster will get created. It will have K8S version as well.

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/e1510a2b-7938-4b95-93dc-b2fb6a3ee527)

- Go inside cluster and in resources tab we'll get all resources available on our cluster. We dont need to go to CLI and type "kubectl get pods" to list pods on cluster.
- We can check pods in different namespaces by choosing them.
- This is an added advantage of using EKS as all the default resources and configurations are automatically created on our cluster, we dont have to mention explicitly. These kind of features are available on most of the K8S distributions but with the plane k8s unless we install dashboard by ourself, we dont get all these features.

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/96e9aea0-4ac1-4fa7-a9df-0d21712e6101)


- In overview section, we see _**"API Server Endpoint"**_ and _**"OpenID Connector provider URL"**_.
- With this k8s cluster we've created we can integrate any identity provider. Identity provider means where we create all users for our organization like LDAP. We can create users here. AWS allows us to attach any identity providers like IAM.

e.g:-   If we created pod who wants to talk to S3, if we dont integrate IAM identity provider, how we'll give access to this pod (role). When any AWS service wants to talk to other service, we need IAM role. Similarly if we create K8S pod, we can attach IAM role with that K8S service account so we can talk to any other AWS services.

- In Compute section, we can see fargate (EC2 not applied here as per requirement). If we create EC2 then we've to attach node groups to them, not applied here.
- In the end, there is fargate profile. By default now fargate profile is attached to default and kubesystem namespace, which means we can deploy pods only to these namespaces. If we've to deploy pod to any other namespace, we've to add other fargate profile.

Compute Section:-

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/4a1c5767-00d0-42eb-8880-78953cc85373)

Fargate profiles:-

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/437764af-9ea5-468c-ab57-0cd30c6c6850)

- In authentication section, we can add any identity providers

- In logging section, if we want to log all API server requests, we can do --> _**manage logging --> enable API server or any other logging as required --> save**_

![image](https://github.com/user-attachments/assets/32c08318-6070-4efe-aabc-39cd04676d6e)

- Now we want to download kube-config file from CLI of kubectl
  - Command :- _**aws eks update-kubeconfig --name demo-cluster --region us-east-1**_

- Now we can deploy actual application. First deploy the pod
  - For this we need to create fargate profile
  - Command :-  _**eksctl create fargateprofile --cluster demo-cluster --region us-east-1 --name alb-sample-app --namespace game-2048**
  - Fargate prfile is created as we're creating it in new namespace, not default one
  - Here we're creating fargate profile because here we're attaching namespace "game-2048". The above command creates fargate profile.
  - Now if we go to compute section, our fargate profile is visible with attached namespace. Now we can create instances in both default and new namespace created. If we want to deploy on any namespaces on fargate, we've to do the same process

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/3f607cbc-cda3-4bf9-a208-4f5c740488a3)

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/aa1fcdc4-8638-45eb-9598-5eb9da717ee6)

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/9bf3e7ba-436c-40f7-a10c-a10f9a79e5a3)

- Now to deploy our app on any namespace created on fargate. This command contains all configuration for deployment, service and ingress
  - Command :-  _**kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml**_
  - We can also see contents of the file. In the file its creating namespace, deployment, service used for app. In service we've to take care of target port is same as container port, also need to ensure proper labels and selectors are used in service and deployment. So service can discover the pods. Then in ingress, we've to route traffic to our cluster.
  - We inside ingress define couple of annotations so if someone tries to access our LB, ingressclass name will read ingress resource and when it finds the matching rules it will forward request to service named 2048
 
  - So request flow will be from Ingress - Service - Deployment - Pods in Namepsace defined.
  - So inside ingress we define service where request is forwarded and service forwards request to deployment/pod which is in the namespace configured.
  - Once above command is run, deployment, namespace, service and ingress will get created

  ![image](https://github.com/user-attachments/assets/8d1d8106-8076-4f4f-a0b6-eec65cd6e2f6)

  ![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/d15ae27e-7e51-4064-a75e-833e1c36bbde)

- As of now using above yml we only created pod, deployment, service and ingress.
- We've not created any ingress controller. There is nothing on our cluster which understand the resources created. So without ingress controllers our resources are useless.

- To see pods and services

  Command :- **kubectl get pods -n game-2048**  (-n means in which namespace we want to list resources)
             **kubectl get svc -n game-2048**

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/8fc41c4a-9b91-41ef-91c8-32a7ed74341b)

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/b7b750ee-7b62-43b9-a3e8-18692e0c4c47)

- If we see service has cluster IP, type is node port but no external IP. So anybody within AWS VPC or anyone who has access to VPC they can talk to pod using node IP address followed by port. But user should access it, so we created ingress.
  - To list ingress :-   **kubectl get ingress -n game-2048**

- Ingress is created but no address is there. Address is required as we've to access the app from outside world. Address is not created as there is no LB or ingress controller.

![image](https://github.com/user-attachments/assets/73f9b1d9-eeba-4c71-b68c-6522bfe0a248)

- Now create ingress controller which will read ingress resource called "ingress-2048" which will create Load balancer for us with all configurations
- Creating LB like ALB will be of no use
- So configurations like target groups, ports, etc are created by ingress resource. All is taken care by ingress controller all we need is ingress resource.
- To deploy ALB controller, we've to create IAM OIDS provider or connector as ALB controller has to access application LB. ALB controller is K8S pod but it needs to talk to AWS resources for which it needs to have IAM integrated. So we need to create IAM OIDC provider.
- This is done in every organization
- In cluster - Authentication there will be OIDC provider as well

![image](https://github.com/user-attachments/assets/e7039295-5197-47e0-9959-c3c1451d39d6)

  - Command :-   _**eksctl utils associate-iam-oidc-provider --cluster demo-cluster --region us-east-1 --approve**_

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/c1c20d12-5a06-4892-93e9-10c17efd0132)

- Now we're trying to install ALB controller which is a pod for which we've to grant it access to services like ALB (role). This is because ALB ingress controller should create ALB for us for which it has to talk to AWS APIs. So here create IAM role for that.

Command :-  _**curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json**_

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/dd67672e-d24e-48e7-808c-ad2fbd677249)

- Then create IAM policy using below

Command :- _**aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json**_

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/8d9acb4a-295e-4d15-b71e-c9bb693e1bc6)

- Then to create role use below
  - When pod is running it will have service account. For the svc account we need role attached to it so that we can integrate with other AWS resources
  - Here IAM service account is getting created and it is also getting attached with role. So we use this account in our application.

  - Command :- _**eksctl create iamserviceaccount --cluster=demo-cluster --region us-east-1 --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::975049937461:policy/AWSLoadBalancerControllerIAMPolicy --approve**_

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/2e91f43e-8c15-4358-acda-c4eca770060a)

- Using above command we are attaching role to service account of our pod. Whenever pod is running, it has service account and service account needs role attached to integrate it with other AWS resources. We can use same service account in our application

-----------------------------------------------------------------------------------------------------------------

- Now to create ALB controller for which we will use helm charts which creates actual controller and it will use the created service account for running pod.

- Now install the helm chart

  - Command1 :- **helm repo add eks https://aws.github.io/eks-charts**  ( to add eks to our repositories)
  - Command2 :- **helm repo update eks**  ( to check for updates)

  ![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/6169eade-6a16-477c-8bd1-d147005bf870)

- Now use below to install helm charts. Install AWS LB Controller (ALB Controller)

  Command :- _**helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=demo-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1 --set vpcId=vpc-0c19787adc3708b9b**_

  ![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/f7ab4634-fd33-4c22-8781-18c92411a3b5)

- Now we've to verify if the ALB is created and there are at least 2 replicas of it

  Command :-   _**kubectl get deployment -n kube-system aws-load-balancer-controller**_
  
  ALB will create 2 replicas one one each in both availability zones. It will continuously watch for ingress resources and will create ALB resources in 2 zones

  ![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/518e2198-d098-49d0-8fec-247411f9b8cf)

- Now lets see if this AWS LB controller has created ALB or not. Go to EC2 and then go to Load balancer. LB is created by BL controller as we submitted an ingress resource  (kubectl get ingress -n game-2048). Running the command gives us address of ingress. This address is the LB that ingress controller has created watching the ingress resource. 

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/a502e8c9-6274-4596-a3da-e4933aba68d5)

- We can go to LB now. Inside LB we can check the IP address (DNS name) will be same as the IP address we got running above command

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/0e924225-b782-4e8f-93c8-2ca98a0e0f71)

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/8a67c5f7-09e0-458f-9499-adc37991b8e0)

# Application Interface for Users

- Now we've to wait till the LB state is active. Copy the URL (IP address of LB) and paste in browser to check if our application is accessible for end users.

![image](https://github.com/Shubham0315/AWS_EKS/assets/105341138/5fb1876d-3e36-4e68-941c-96c2ce547d2e)

