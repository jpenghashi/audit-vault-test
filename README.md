# audit-vault-test

This is a step-by-step tutorial based on the [comment](https://github.com/hashicorp/vault-helm/issues/142#issuecomment-1151353386) for enabling logrotate for Vault Audit logs deployed in Kubernetes environment. 
Note that, this is not considered as official nor secure in any way.  It is only used for tutorial or demo purposes and you should remove everything right after the tutorial is completed.

## Step up k8s cluster
I performed the steps with my EKS cluster running on T3.small by utilizing the [Terraform tutorial](https://developer.hashicorp.com/terraform/tutorials/kubernetes/eks). If you already have a running k8s cluster on your end, you may skip this step.

## Build your dockerfile and upload it to AWS ECS
- Go to your AWS Console Create a Public Repository under AWS ECR (Note: this repo will be publicly accessible, do not host anything that you wish to remain private). 
- Select ‘View Push Commands’ under AWS ECR, and follow the prompts for docker login (Step 1 in the screenshot below). <img width="766" alt="Screenshot 2024-04-11 at 11 10 12 PM" src="https://github.com/jpenghashi/audit-vault-test/assets/86845444/13bc84d7-738e-4b88-baf1-7a8516e15c36">
- git clone https://github.com/HanseMerkur/vault-logrotate.git and go into vault-logrotate directory (Note, this repo is from third party. The libraries used in the repo is considfered as outdated and should be used for demo purposes only.)
- export DOCKER_DEFAULT_PLATFORM=linux/amd64. Make sure you specify the right architecture for your build in the environment variable.
- Follow Steps 2-4 in your `View Push Commands`
- Validate in AWS ECR Console that the image has been pushed successfully

Note: If you are not running EKS, Alternatively, you may Push it to [DockerHub](https://docs.docker.com/get-started/04_sharing_app/), or install the Docker image locally on the machine where the k8s cluster is running on. 

