# k8s-eks-cloudformation
This project creates:

- Two public subnets containing
    - Internet Gateway
    - NAT Gateway
- Two private subnets contining
    - EKS nodes
- EKS cluster
- ECR repository

There did not seem to be a complete example of deploying all required components of EKS with CloudFormation. This example includes:

- OIDC Provider
- EBS CSI Add-on and associated IAM Roles for Service Accounts (IRSA) and policy
- Core DNS Add-on
- Kube Proxy Add-on

[When you provision a cluster, Amazon EKS installs VPC CNI automatically.](https://aws.github.io/aws-eks-best-practices/networking/vpc-cni/#deploy-vpc-cni-managed-add-on)

Both JSON and YAML examples are the same deployment, and were provided to show examples of both.

## References
Some of the more tricky elements were writing the IAM role's trust policy while stripping the leading https:// from the [AWS::EKS::Cluster OpenIdConnectIssuerUrl](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-eks-cluster.html#aws-resource-eks-cluster-return-values) attributee. 

Writing the AssumeRolePolicyDocument as a [Fn::Sub](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-sub.html) was part of this solution.

[The trick is to know that the AssumeRolePolicyDocument value in a template is specified as json. As such, we can perform a !Sub on any value we pass to it, as long as the result is a valid json string.](https://bambooengineering.io/constraining-eks-pod-iam-roles-using-cloudformation/)

The [ThumbprintList for the OIDCProvider is hardcoded](https://gist.github.com/riccardomc/a3891356b09516ab3f3b79a12e9b13e1), as the certificate is valid until 06/29/2034. We can get the complete [AWS::EKS::Cluster CertificateAuthorityData](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-eks-cluster.html#aws-resource-eks-cluster-return-values) but would then need to [calculate the thumbprint using OpenSSL](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc_verify-thumbprint.html) and an external process.