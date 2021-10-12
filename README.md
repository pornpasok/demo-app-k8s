# demo-app-k8s

### Workshop Kubernetes (k8s) Demo App
GitHub/GitLab + Jenkins + k8s (EKS)

### IAM CI/CD v1 Overview
![IAM CI/CD v1](images/iam_v1.png)

- Jenkins: https://cicd.tono.io/
- Nexus: https://repository.tono.io:8081/
- SonarQube: https://sq.tono.io/
- Monitoring: https://monitor-dev.tono.io/

### Update kubeconfig EKS Dev
1.Install aws cli

2.Install kubectl

3.Insert aws credentials to ~/.aws/credentials
```
[tono-eks-dev-workshop]
aws_access_key_id=xxx
aws_secret_access_key=xxx
```

4.Update kubeconfig (~/.kube/config) with aws cli
```
aws eks update-kubeconfig --name tono-eks-dev \
--region ap-southeast-1 \
--profile tono-eks-dev-workshop
```

5.Install Lens (Options)
https://k8slens.dev/

6.Test 
```
kubectl get node
```
- Open Lens add tono-eks-dev cluster

## Ref:
- DO280
https://www.redhat.com/en/services/training/do280-red-hat-openshift-administration-II-operating-production-kubernetes-cluster
- DO288
https://www.redhat.com/en/services/training/do288-red-hat-openshift-development-ii-containerizing-applications
