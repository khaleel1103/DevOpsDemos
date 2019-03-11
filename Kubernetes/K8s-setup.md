
#### Setup Kubernetes (K8s) Cluster on AWS

1. Create an IAM user with Route53, EC2 and S3 full access
1. Create Ubuntu EC2 instance
1. install AWSCLI
   ```sh 
    curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
    apt install unzip python
    unzip awscli-bundle.zip
    #sudo apt-get install unzip - if you dont have unzip in your system
    ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
    ```
    
1. Install kubectl
   ```sh
   curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
   ```
1. Install kops on ubuntu instance:
   ```sh
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```
1. Setup IAM programatic access to on Ubuntu server
   ```sh 
     aws configure
    ```
1. create an S3 bucket 
   ```sh
    aws s3 mb s3://cluster.k8s.valaxy.in
   ```
1. Expose environment variable:
   ```sh 
    export KOPS_STATE_STORE=s3://cluster.k8s.valaxy.in
   ```
1. Create sshkeys before creating cluster
   ```sh
    ssh-keygen
   ```
1. Create kubernetes cluster defenations on S3 bucket 
   ```sh 
    kops create cluster --cloud=aws --zones=ap-southeast-1b --name=southeast.k8s.valaxy.in --dns-zone=valaxy.in --dns private
    ```
1. Create kubernetes cluser
    ```sh 
      kops update cluster southeast.k8s.valaxy.in --yes
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
1. Deploying Nginx Container
    ```sh 
      kubectl run sample-nginx --image=nginx --replicas=2 --port=80
      kubectl get pods
      kubectl get deployments
   ```
   
1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them:
   ```sh 
    kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
    kubectl get services -o wide
    ```