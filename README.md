# Kubernetes Cluster Setup with Kops on AWS

This repository provides a step-by-step guide for deploying a Kubernetes cluster on AWS using Kops. Kops is a tool that simplifies the creation, scaling, and lifecycle management of Kubernetes clusters, automating many tasks such as provisioning infrastructure, setting up the control plane, and managing worker nodes.

## Prerequisites
Before you begin, ensure that you have the following installed:
- **Python3** (for AWS CLI and Kops installation)
- **AWS CLI** (for interacting with AWS)
- **kubectl** (for interacting with the Kubernetes cluster)
- **Kops** (for managing the Kubernetes cluster)


## Installation

To install AWS CLI, kubectl, and Kops, follow these steps:

Install kubectl, awscli, and other dependencies:
```bash
# Add the Kubernetes apt repository key
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

# Add the Kubernetes apt repository to your system
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list

# Update apt-get repository list
sudo apt-get update

# Install kubectl, awscli, and other dependencies
sudo apt-get install -y python3-pip apt-transport-https kubectl

# Install awscli using pip3
pip3 install awscli --upgrade

# Add awscli to PATH (if it's not already)
export PATH="$PATH:/home/ubuntu/.local/bin/"
```
Install Kops (our hero for today):

```bash
# Download the latest stable version of kops

curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

# Make the kops binary executable
chmod +x kops-linux-amd64

# Move the kops binary to a directory in your PATH
sudo mv kops-linux-amd64 /usr/local/bin/kops
```
Once installed, verify by running the following commands:

```bash
kubectl version --client
aws --version
kops version
```

## Create S3 Bucket
Kops requires an S3 bucket to store the cluster state. To create the S3 bucket, use the AWS CLI. Note that for the us-east-1 region, you don't need to specify the LocationConstraint as it's the default region.
For regions like us-east-1, you can create the S3 bucket with no LocationConstraint:
```bash
aws s3api create-bucket --bucket <your-bucket-name> --region us-east-1
```
For other regions (e.g., eu-north-1), you'll need to specify the LocationConstraint
```bash
aws s3api create-bucket --bucket <your-bucket-name> --region <region> --create-bucket-configuration LocationConstraint=<region>
```
**Note:** The LocationConstraint is only needed for certain AWS regions like eu-north-1, and is not necessary for us-east-1 since it's the default region.

## Create the Cluster
I used the kops create cluster command to define the Kubernetes cluster configuration. This included specifying node sizes, availability zones, instance types, and volume sizes.
Example:
```bash
kops create cluster \
  --name=k8s.local \
  --state=s3://<your-bucket-name> \
  --zones=us-east-1a \
  --node-count=2 \
  --node-size=t2.micro \
  --master-size=t2.micro \
  --master-volume-size=10 \
  --node-volume-size=10
```
## Update and Validate the Cluster
After creating the cluster configuration, I applied it using the following command:
```bash
kops update cluster --yes --state=s3://<your-bucket-name>
```
I then validated the cluster to ensure everything was running correctly:
```bash
kops validate cluster
```
## Production Considerations
For a production environment, I would consider:

High Availability: Distribute master nodes across multiple availability zones.

Auto-scaling: Automatically adjust the number of nodes based on traffic.

Monitoring and Logging: Integrate tools like Prometheus and Grafana for cluster health monitoring.

Security: Use Kubernetes RBAC and IAM roles for access control.

Basic Usage Example
After completing the setup, you can manage the cluster using kubectl. Here's an example of how to verify the nodes in your cluster:

```bash
kubectl get nodes
```
