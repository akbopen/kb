# Step 1: Create EKS cluster
## 1. Create an Amazon VPC with public and private subnets that meets Amazon EKS requirements. 

```
aws cloudformation create-stack \
  --stack-name airformex-eks-vpc-stack \
  --template-url https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml
``` 
 
## 2. Create a cluster IAM role and attach the required Amazon EKS IAM managed policy to it. Kubernetes clusters managed by Amazon EKS make calls to other AWS services on your behalf to manage the resources that you use with the service.
### a. Copy the following contents to a file named `airformex-cluster-role-trust-policy.json`.
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### b. Create the role.
```
aws iam create-role \
  --role-name AirFormexEKSClusterRole \
  --assume-role-policy-document file://"airformex-cluster-role-trust-policy.json"
```

### c. Attach the required Amazon EKS managed IAM policy to the role.
```
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy \
  --role-name AirFormexEKSClusterRole
```
## 3. Open the Amazon EKS console at https://console.aws.amazon.com/eks/home#/clusters. 


# Step 2: Configure to communicate with cluster 
## 1. Create or update a kubeconfig file for cluster.

```
aws eks update-kubeconfig \
  --region ap-southeast-2 \
  --name AirFormex-EKS
  ```
## 2. Test configuration
```kubectl get svc```

# Step 3: Create IAM OpenID Connect (OIDC) provider 

Create an IAM OpenID Connect (OIDC) provider for your cluster so that Kubernetes service accounts used by workloads can access AWS resources. You only need to complete this step one time for a cluster.

1. Select the Configuration tab.
2. In the Details section, copy the value for OpenID Connect provider URL.
3. Open the IAM console at https://console.aws.amazon.com/iam/.
4. In the navigation panel, choose Identity Providers.
5. Choose Add Provider.
6. For Provider Type, choose OpenID Connect.
7. For Provider URL, paste the OIDC provider URL for your cluster from step two, and then choose Get thumbprint.
8. For Audience, enter sts.amazonaws.com and choose Add provider.

# Step 4: Create nodes
## 1. Create an IAM role for the Amazon VPC CNI plugin
### a. Copy the following contents to a file named cni-role-trust-policy.json. Replace <111122223333> (including <>) with your account ID and replace <XXXXXXXXXX45D83924220DC4815XXXXX> with the value after the last / of your OpenID Connect provider URL.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::213397327449:oidc-provider/oidc.eks.ap-southeast-2.amazonaws.com/id/C7AD155D771A5964308FAEC6830A0043"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.ap-southeast-2.amazonaws.com/id/C7AD155D771A5964308FAEC6830A0043:sub": "system:serviceaccount:kube-system:aws-node"
        }
      }
    }
  ]
}
```
### b. Create an IAM role for the Amazon VPC CNI plugin.
```
aws iam create-role \
  --role-name AirFormexEKSCNIRole \
  --assume-role-policy-document file://"cni-role-trust-policy.json"
```
### c. Attach the required Amazon EKS managed IAM policy to the role. 
```
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy \
  --role-name AirFormexEKSCNIRole
```
## 2. Associate the Kubernetes service account used by the VPC CNI plugin to the IAM role. 
```
aws eks update-addon \
  --cluster-name AirFormex-EKS \
  --addon-name vpc-cni \
  --service-account-role-arn arn:aws:iam::213397327449:role/AirFormexEKSCNIRole 
```
## 3. Create a node IAM role and attach the required Amazon EKS IAM managed policy to it.
### a. Copy the following contents to a file named `node-role-trust-policy.json`. 
```
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
```
### b. Create the node IAM role.
```
aws iam create-role \
  --role-name AirFormexEKSNodeRole \
  --assume-role-policy-document file://"node-role-trust-policy.json"
```
### c. Attach the required Amazon EKS managed IAM policies to the role. 
```
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy \
  --role-name AirFormexEKSNodeRole
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly \
  --role-name AirFormexEKSNodeRole
```
## 4. Open the Amazon EKS console at https://console.aws.amazon.com/eks/home#/clusters
## 5. Choose the name of the cluster that you created in Step 1.
## 6. Select the Configuration tab.
## 7. On the Configuration tab, select the Compute tab, and then choose Add Node Group.
## 8. On the Configure node group page, fill out the parameters accordingly, accept the remaining default values, and then choose Next.
- Name – Enter a unique name for your managed node group, `AirFormex-EKS-Nodegroup`.
- Node IAM role name – Choose `AirFormexEKSNodeRole`. In this getting started guide, this role must only be used for this node group and no other node groups.
## 9. On the Set compute and scaling configuration page, accept the default values and select Next.
## 10. On the Specify networking page, select an existing key pair to use for SSH key pair and then choose Next. 
```
aws ec2 create-key-pair --region ap-southeast-2 --key-name myKeyPair
```
## 11. On the Review and create page, review your managed node group configuration and choose Create.
## 12. After several minutes, the Status in the Node Group configuration section will change from Creating to Active. Don't continue to the next step until the status is Active.
