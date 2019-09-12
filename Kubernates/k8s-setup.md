
#### Setup Kubernetes (K8s) Cluster on AWS


1. Create Ubuntu EC2 instance
1. install AWSCLI
   ```sh 
    curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
    apt install unzip python
    unzip awscli-bundle.zip
    #sudo apt-get install unzip - if you dont have unzip in your system
    ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
    ```
    4update:  pip install awscli --upgrade --user
	
1. Install kubectl
   ```sh
   curl -LO https://storage.googleapis.com/kubernetes-release/release/
	$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
   chmod +x ./kubectl
   sudo mv ./kubectl /usr/local/bin/kubectl
   ```
1. Create an IAM user/role  with Route53, EC2, IAM and S3 full access
   ```sh
   Setup IAM user
	In order to build clusters within AWS we'll create a dedicated IAM user for kops. This user requires 
	API credentials in order to use kops. Create the user, and credentials, using the AWS console.

	The kops user will require the following IAM permissions to function properly:

	AmazonEC2FullAccess
	AmazonRoute53FullAccess
	AmazonS3FullAccess
	IAMFullAccess
	AmazonVPCFullAccess

	You can create the kops IAM user from the command line using the following:

	aws iam create-group --group-name kops
	aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name kops
	aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name kops
	aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name kops
	aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name kops
	aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name kops
	aws iam create-user --user-name kops
	aws iam add-user-to-group --user-name kops --group-name kops
	aws iam create-access-key --user-name kops

	You should record the SecretAccessKey and AccessKeyID in the returned JSON output, and then use them below:

	# configure the aws client to use your new IAM user
	aws configure           # Use your new access and secret key here
	aws iam list-users      # you should see a list of all your IAM users here

	# Because "aws configure" doesn't export these vars for kops to use, we export them now
	export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
	export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)
   ```
1. Attach IAM role to ubuntu server

    #### Note: If you create IAM user with programmatic access then provide Access keys. 
   ```sh 
     aws configure
    ```
1. Install kops on ubuntu instance:
   ```sh
    curl -LO https://github.com/kubernetes/kops/releases/download/
	 $(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```
1. Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
1. create an S3 bucket 
   ```sh
     aws s3 mb s3://clusters.k8s.alohapro.tk
   ```
1. Expose environment variable:
   ```sh 
     export KOPS_STATE_STORE=s3://clusters.k8s.alohapro.tk
   ```
1. Create sshkeys before creating cluster
   ```sh
    ssh-keygen
   ```
1. Create kubernetes cluster definitions on S3 bucket 
   ```sh 
    kops create cluster --cloud=aws --zones=us-east-1d --name=useast1.k8s.alohapro.tk --dns-zone=alohapro.tk --dns public
   ```
	diffrent ways:
   ```sh 
	kops create cluster 
    --master-count 1
    --bastion
    --cloud=aws 
    --zones=us-east-1d 
    --name=useast1.k8s.alohapro.tk 
    --dns-zone=alohapro.tk 
    --dns public

	  # Create a cluster in AWS
	  
	  kops create cluster --name=kubernetes-cluster.example.com \
	  --state=s3://kops-state-1234 --zones=eu-west-1a \
	  --node-count=2
	  
	  # Create a cluster in AWS that has HA masters.  This cluster
	  # will be setup with an internal networking in a private VPC.
	  # A bastion instance will be setup to provide instance access.
	  
	  export NODE_SIZE=${NODE_SIZE:-m4.large}
	  export MASTER_SIZE=${MASTER_SIZE:-m4.large}
	  export ZONES=${ZONES:-"us-east-1d,us-east-1b,us-east-1c"}
	  export KOPS_STATE_STORE="s3://my-state-store"
	  kops create cluster k8s-clusters.example.com \
	  --node-count 3 \
	  --zones $ZONES \
	  --node-size $NODE_SIZE \
	  --master-size $MASTER_SIZE \
	  --master-zones $ZONES \
	  --networking weave \
	  --topology private \
	  --bastion="true" \
	  --yes
	  
	  # Create cluster in GCE.
	  # This is an alpha feature.
	  export KOPS_STATE_STORE="gs://mybucket-kops"
	  export ZONES=${MASTER_ZONES:-"us-east1-b,us-east1-c,us-east1-d"}
	  export KOPS_FEATURE_FLAGS=AlphaAllowGCE
	  
	  kops create cluster kubernetes-k8s-gce.example.com
	  --zones $ZONES \
	  --master-zones $ZONES \
	  --node-count 3
	  --project my-gce-project \
	  --image "ubuntu-os-cloud/ubuntu-1604-xenial-v20170202" \
	  --yes
	  # Create manifest for a cluster in AWS
	  kops create cluster --name=kubernetes-cluster.example.com \
	  --state=s3://kops-state-1234 --zones=eu-west-1a \
	  --node-count=2 --dry-run -oyaml

	MICRO

	 kops create cluster \
		   --state "s3://state.chat.poeticoding.com" \
		   --zones "us-east-1d,us-east-1f"  \
		   --master-count 3 \
		   --master-size=t2.micro \
		   --node-count 2 \
		   --node-size=t2.micro \
		   --name chat.poeticoding.com \
		   --yes \
		   --cloud=aws


	export KOPS_STATE_STORE="s3://clusters.k8s.alohapro.tk"
	export KOPS_CLUSTER_NAME=dev.k8s.alohapro.tk

	 kops create cluster \
		   --state "s3://clusters.k8s.alohapro.tk"  \
		   --zones "eu-central-1a"  \
		   --master-count 1 \
		   --master-size=t2.micro \
		   --node-count 2 \
		   --node-size=t2.micro \
		   --name=$KOPS_CLUSTER_NAME  \
		   --cloud=aws \
		   --dns-zone=alohapro.tk  \
		   --dns public
		   
	```
	else

	https://www.poeticoding.com/create-a-high-availability-kubernetes-cluster-on-aws-with-kops/

    
1. Create kubernetes cluser
    ```sh 
      kops update cluster useast1.k8s.alohapro.tk  --yes
     ```
1. Validate your cluster 
     ```sh 
      kops validate cluster
    ```

1. To list nodes
   ```sh 
     kubectl get nodes 
   ```

#### Deploying Nginx container on Kubernetes 

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

1. Deploying Nginx Container
    ```sh 
      kubectl run sample-nginx --image=nginx --replicas=2 --port=80
      kubectl get pods
      kubectl get deployments
   ```
   
1. Delete Nginx Container
    ```sh 
	  kubectl delete deployment sample-nginx
    ```
   
1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them:
   ```sh 
    kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
    kubectl get services -o wide
    ```
 1. To delete cluster
    ```sh
     kops delete cluster useast1.k8s.alohapro.tk  --yes
    ```

### Another Helpful command
```sh
 kubectl cluster-info
 kubectl config current-context
```



