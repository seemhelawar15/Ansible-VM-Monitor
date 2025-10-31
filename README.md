**1 #Ansible Project To Monitor VMs Health**
Step 1: Update the System
sudo apt update && sudo apt upgrade -y

Step 2: Add the Ansible PPA
Ansible provides an official maintained PPA (for latest versions):
sudo add-apt-repository --yes --update ppa:ansible/ansible

Step 3: Install Ansible
sudo apt install ansible -y

**2 # Install AWS CLI**
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws configure

**3#Create a Tagging Script for tagging the ec2 instances:**
#!/bin/bash

#Fetch instance IDs that match Environment=dev and Role=web
instance_ids=$(aws ec2 describe-instances \
  --filters "Name=tag:Environment,Values=dev" "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text)

#Sort instance IDs deterministically
sorted_ids=($(echo "$instance_ids" | tr '\t' '\n' | sort))

#Rename instances sequentially
counter=1
for id in "${sorted_ids[@]}"; do
  name="web-$(printf "%02d" $counter)"
  echo "Tagging $id as $name"
  aws ec2 create-tags --resources "$id" \
    --tags Key=Name,Value="$name"
  ((counter++))
done



**4 #Create config file for host checking of inventory file**
ansible.cfg
[defaults]
inventory = ./inventory/aws_ec2.yaml
host_key_checking = False

[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null


**5 #Dynamic Inventory**
vi inventory/aws_ec2.yaml 
plugin: amazon.aws.aws_ec2
regions:
  - ap-south-1
filters:
  tag:Environment: dev
  instance-state-name: running
compose:
  ansible_host: public_ip_address
keyed_groups:
  - key: tags.Name
    prefix: name
  - key: tags.Environment
    prefix: env                               

**6 # Step 1: Install venv module if not already present**
sudo apt install python3-venv -y

**7 # Step 2: Create a virtual environment**
python3 -m venv ansible-env

**8 # Step 3: Activate it**
source ansible-env/bin/activate

**9 # Step 4: Install required Python packages**
pip install boto3 botocore docker

ansible-galaxy collection install amazon.aws

ansible-inventory -i inventory/aws_ec2.yaml --graph

