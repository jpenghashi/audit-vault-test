# audit-vault-test

This is a step-by-step tutorial based on the [comment](https://github.com/hashicorp/vault-helm/issues/142#issuecomment-1151353386) for enabling logrotate for Vault Audit logs deployed in Kubernetes environment. 
Note that, this is not considered as official nor secure in any way.  It is only used for tutorial or demo purposes and it is advisable to remove everything that was done right after the tutorial is completed.

## Step up k8s cluster
I performed the steps with my EKS cluster running on T3.small by utilizing the [Terraform tutorial](https://developer.hashicorp.com/terraform/tutorials/kubernetes/eks). If you already have a running k8s cluster on your end, you may skip this step.

## Build your dockerfile and upload it to AWS ECR
- Go to your AWS Console Create a Public Repository under [AWS ECR](https://aws.amazon.com/ecr/) (Note: this repo will be publicly accessible, do not host anything that you wish to remain private). 
- Select ‘View Push Commands’ under AWS ECR, and follow the prompts for docker login (Step 1 in the screenshot below). <img width="766" alt="Screenshot 2024-04-11 at 11 10 12 PM" src="https://github.com/jpenghashi/audit-vault-test/assets/86845444/13bc84d7-738e-4b88-baf1-7a8516e15c36">
- ```git clone https://github.com/HanseMerkur/vault-logrotate.git``` and go into vault-logrotate directory (Note, this repo is from third party. The libraries used in the repo is considfered as outdated and should be used for demo purposes only.)
- ```export DOCKER_DEFAULT_PLATFORM=linux/amd64``` to make sure you specify the right architecture for your build in the environment variable. As T3.small is running on amd64, here I specify that arch accordingly.
- Follow Steps 2-4 in your `View Push Commands`
- Validate in AWS ECR Console that the image has been pushed successfully
- Copy the Image URI from AWS ECR Console (you will need this later)
  
Note: If you are not running EKS, Alternatively, you may Push it to [DockerHub](https://docs.docker.com/get-started/04_sharing_app/), or install the Docker image locally on the machine where the k8s cluster is running on. 

## Configmap
- Peek a look at configmap.yaml in this repo, and notice that the size is defined as 1M, meaning that the audit log is set to rotate when it exceeds 1 Megabyte.
- ```kubectl apply -f configmap.yaml```. Note that, the configmaps are k8s namespace specific.

## Change overrride-values.yaml
- Go to line 28, replace <Image URI> with the Image URI from the last step when you build your dockerfile.
- In line 32, there is a CRONJOB that is set to run the auditlog-rotator every 1 minute. You may change the frequency here.
- ```helm install vaultaudit hashicorp/vault -f override-values.yaml```
- Run ```kubectl get pods``` and ensure that vaultaudit-0 pod is running with 2/2.
- In the auditlogrotator container, you should observe that the audit log rotator is running:
```
>>> kubectl logs vaultaudit-0 auditlog-rotator
Starting vault logrotation with "*/1 * * * *"
Starting logrotation
Finished logrotation
```

## Enable Vault Audit Device
- Exec into the vault container inside vaultaudit-0 pod ```kubectl exec -it vaultaudit-0 -c vault -- sh```
- Initlialize Vault ```vault operator init -key-shares=1 -key-threshold=1```, and Unseal Vault with ```vault operator unseal <Unseal Key>``` and Login with ```vault login <token>```
- Enable the Audit Device ```vault audit enable file file_path=/vault/audit/vault.log```. Note that, the path /vault/audit/vault.log is defined in the configmap. If you wish to enable in another path, change the configmap accordingly and re-apply.
- Write a kv secret, and keep reading KV secrets continuously, and observe /vault/audit/vault.log within the vault container in vaultaudit-0 pod, you should see that the audit log rotates whenever the vault.log exceeds 1M:
```
/vault/audit $ ls -lh
total 2M     
drwxrws---    2 root     vault      16.0K Apr 12 07:06 lost+found
-rw-------    1 vault    vault       1.1M Apr 12 07:20 vault.log
/vault/audit $ ls -lh
total 1M     
drwxrws---    2 root     vault      16.0K Apr 12 07:06 lost+found
-rw-------    1 vault    vault          0 Apr 12 07:21 vault.log
```

## Delete everything
- ```helm delete vaultaudit```
- ```kubectl delete configmap vaultaudit-config```
- ```kubectl delete pvc audit-vaultaudit-0 data-vaultaudit-0```
- Delete the Image from AWS ECR Console
- ```docker images``` and delete the images that was built locally.

   





