# Ingress on AWS EKS ( AWS Load Balancer Controller )

## AWS Load Balancer Controller

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=modern-jazz-mushrooms --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=ap-northeast-2 --set vpcId=<VPC_ID> --set image.repository=<REGISTRY_URL>/eks/aws-load-balancer-controller --set image.tag=v3.5.0 --set enable-shield=false --set enable-waf=false --set enable-waf-v2=false
```

https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html

https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html

## Deployment

```bash
## CREATE AN IAM OIDC PROVIDER
eksctl utils associate-iam-oidc-provider \
    --region <region-code> \
    --cluster <your-cluster-name> \
    --approve

## IAM POLICY FOR THE LBC
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v3.5.0/docs/install/iam_policy.json

## CREATE IAM POLICY FOR THE LBC
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json

## CREATE AN IAM ROLE AND KUBERNETES SERVICE ACCOUNT
eksctl create iamserviceaccount \
--cluster=<cluster-name> \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=ARN_OF_ABOVE_IAM_POLICY \
--override-existing-serviceaccounts \
--region <region-code> \
--approve

## ADD EKS CHART REPO TO HELM
helm repo add eks https://aws.github.io/eks-charts

## HELM INSTALL COMMANDS FOR CLUSTER WITH IRSA
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=<CLUSTER_NAME> --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=<AWS_REGION> --set vpcId=<VPCID> --set image.repository=<REGISTRY_URL>/eks/aws-load-balancer-controller --set image.tag=v3.5.0 --set enable-shield=false --set enable-waf=false --set enable-waf-v2=false
```

### Install eksctl

```bash
# for ARM systems, set ARCH to: `arm64`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
```

### Install Helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
```
