Ansible Project To Monitor VMs Health
🔹 Step 1: Update the System
sudo apt update && sudo apt upgrade -y
________________________________________
🔹 Step 2: Add the Ansible PPA
Ansible provides an official maintained PPA (for latest versions):
sudo add-apt-repository --yes --update ppa:ansible/ansible
________________________________________
🔹 Step 3: Install Ansible
sudo apt install ansible -y
________________________________________
# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws configure

#Create config file for host checking of inventory file
ansible.cfg
[defaults]
inventory = ./inventory/aws_ec2.yaml
host_key_checking = False

[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null


#Dynamic Inventory
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

# Step 1: Install venv module if not already present
sudo apt install python3-venv -y

# Step 2: Create a virtual environment
python3 -m venv ansible-env

# Step 3: Activate it
source ansible-env/bin/activate

# Step 4: Install required Python packages
pip install boto3 botocore docker

ansible-galaxy collection install amazon.aws

ansible-inventory -i inventory/aws_ec2.yaml --graph

