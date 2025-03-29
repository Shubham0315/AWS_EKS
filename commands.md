To create EKS Cluster :- **eksctl create cluster --name $cluster-name --region us-east2 --fargate**

To delete EKS cluster :- **eksctl delete cluster --name $cluster-name --region us-east2**

To download kube-config file :- **aws eks update-kubeconfig --name demo-cluster --region us-east-1**

---------------------------------------------------------------------------------------------

Deployment Steps
-

1. Create Fargate Profile
- Command :- **eksctl create fargateprofile --cluster demo-cluster --region us-east-1 --name alb-sample-app --namespace game-2048**

2. Create Resources on EKS
- If we've defined pods, deployments, namespace, ingress, service in one file and apply it to kubectl
- Command :- **kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml**

3. Checking created Resources
- Pods Command :- **kubectl get pods -n $nameSpace**
- Service command :- **kubectl get svc -n $nameSpace**
- Ingress Command :- **kubectl get ingress -n game-2048**

4. To create IAM OIDC provider
- Command :- **eksctl utils associate-iam-oidc-provider --cluster demo-cluster --region us-east-1 --approve**

5. To create IAM policy
- Command :- **aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json**
- .json is file document from where policy is to be created

6. To create Role
- Command :- _**eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --region us-east-1 \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::975049937461:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve**_

7. Helm Commands
- Command1 :- **helm repo add eks https://aws.github.io/eks-charts**  ( to add eks to our repositories)
- Command2 :- **helm repo update eks**  ( to check for updates)

8. To install AWS ALB using helm charts
- Command :- **helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=vpc-0c19787adc3708b9b**
- Take VPC name from cluster - Networking

9. Check LB creation
- Command :-   **kubectl get deployment -n kube-system aws-load-balancer-controller**

10. To delete helm charts if required
- Command :- **helm delete aws-load-balancer-controller -n kube-system**

