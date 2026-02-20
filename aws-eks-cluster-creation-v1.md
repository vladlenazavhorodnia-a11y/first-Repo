				EKS cluster creation:


1. Prerequisites:
	1.1. Cluster IAM Role AmazonEKSAutoClusterRole (EKS)
		1.1.1. Trust relationship:

---------------------------------------------------------
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "eks.amazonaws.com"
            },
            "Action": [
                "sts:AssumeRole",
                "sts:TagSession"
            ]
        }
    ]
}
---------------------------------------------------------


		1.1.2. Attached IAM Policies:
			1.1.2.1. AmazonEKSBlockStoragePolicy
			1.1.2.2. AmazonEKSClusterPolicy
			1.1.2.3. AmazonEKSComputePolicy
			1.1.2.4. AmazonEKSLoadBalancingPolicy
			1.1.2.5. AmazonEKSNetworkingPolicy

	1.2. Node IAM Role AmazonEKSNodeRole (EC2)
		1.2.1. Trust relationship:

---------------------------------------------------------
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
---------------------------------------------------------


		1.2.2. Attached IAM Policies:
			1.2.2.1. AmazonEC2ContainerRegistryPullOnly
			1.2.2.2. AmazonEKS_CNI_Policy
			1.2.2.3. AmazonEKSWorkerNodePolicy


	1.3 VPC and Subnets configuration.
		1.3.1. VPC CIDR (10.10.0.0/16)
			1.3.1.1. DNS Hostnames: Enabled
			1.3.1.2. DNS Support: Enabled
		1.3.2. Subneting
			1.3.2.1. Public subnet "A" (10.10.1.0/24)
				1.3.2.1.1. Tags:
					kubernetes.io/role/elb = 1
					kubernetes.io/cluster/your-cluster-name = shared
				1.3.2.1.2. Add Public IP: Actions/Edit subnet settings/turn "ON" Enable auto-assign public IPv4 address
			1.3.2.2. Public subnet "B" (10.10.2.0/24)
				1.3.2.2.1. Tags:
					kubernetes.io/role/elb = 1
					kubernetes.io/cluster/your-cluster-name = shared
				1.3.2.2.2. Add Public IP: Actions/Edit subnet settings/turn "ON" Enable auto-assign public IPv4 address
			1.3.2.3. Public subnet "C" (10.10.3.0/24)
				1.3.2.3.1. Tags:
					kubernetes.io/role/elb = 1
					kubernetes.io/cluster/your-cluster-name = shared
				1.3.2.3.2. Add Public IP: Actions/Edit subnet settings/turn "ON" Enable auto-assign public IPv4 address

		1.3.3 Networking components:
			1.3.3.1. Internet Gateway (IGW): created and attached to VPC
			1.3.3.2. Route Table: created 0.0.0.0/0->IGW. Subnet Association.


2. Cluster Deployment.
	2.1. EKS Auto Mode Cluster Creation
	2.2. OIDC Provider Association
	2.3. Add-ons Configuration:
		2.3.1 VPC CNI
		2.3.2. CoreDNS
		2.3.3. kube-core
	2.4. Node Group Creatin: choose IAM role from 1.2

	2.5. Access to Cluster from Ubuntu: Add IAM User to access list in EKS Cluster (if in ubuntu system you are using access and secret keys for IAM user and attach Policy AmazonEKSClusterAdminPolicy)
	2.6. Node Group IAM Role Creation (AmazonEKSLoadBalancerControllerRole):
		2.6.1. Trust Relationship:

---------------------------------------------------------

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::002369671013:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/4A50CF2042BDFEFCE3810D8E32986006"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "oidc.eks.us-east-1.amazonaws.com/id/4A50CF2042BDFEFCE3810D8E32986006:aud": "sts.amazonaws.com",
                    "oidc.eks.us-east-1.amazonaws.com/id/4A50CF2042BDFEFCE3810D8E32986006:sub": "system:serviceaccount:kube-system:aws-load-balancer-controller"
                }
            }
        }
    ]
}

---------------------------------------------------------
		
		2.6.2. Attached Policies:
			2.6.2.1. AWSLoadBalancerControllerIAMPolicy
			AmazonEC2FullAccess


3. Ubuntu Cluster connection

aws eks update-kubeconfig --region us-east-1 --name cluster


4. EKS Cluster Configuring
	4.1. Cert-Manager
		4.1.1. "kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.3/cert-manager.yaml"

		4.1.2. to check: "kubectl get pods -n cert-manager"  

	4.2. Ingress controller creation:



# Применяем наш "Золотой манифест"
kubectl apply -f ingress-controller-v4.yaml

# Проверяем, что он сел на ноды (благодаря tolerations)
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller

# Запускаем создание балансировщика
kubectl apply -f test-ingress.yaml
















