## Lab 1: Installation and Configuration of Ansible

### Task-0: The first step is to `Manually Launch an Ansible-ControlNode` with the below configuration:

#### Step-01: Pre-requisites:

In EC2 Dashboard, and under "Network & Security," `create a key pair` and a `security group.`

1. Create key pair with name: `Ansible-Keypair-YourName`
2. Create security group with name: `Ansible-SG-YourName`
   (Include Ports: `22 [SSH],` `80 [HTTP]` for all incoming traffic.)

Once you are ready, While Manually Launching an `Ansible-ControlNode`, select the above `Ansible-Keypair-YourName` and `Ansible-SG-YourName`

#### Step-02: Steps for Manually launching the Anchor EC2 Instance

* **Region:** North Virginia (us-east-1).
* **Use tag Name:** `Ansible-ControlNode`
* **AMI Type and OS Version:** `Red Hat Enterprise Linux 9 (HVM)`
* **Instance type:** `t2.micro`
* Choose the existing Keypair with the Name: `Ansible-Keypair-YourName`
* In security groups, Choose the existing security group with name: `Ansible-SG-YourName`
* **Configure Storage:** 10 GiB
* Click on `Launch Instance.` to launch it.
---------------------------------------------------------------------
### Task-1: Install AWS CLI, Ansible and Dependencies and Create a Playbook, Ececute to launch 2 Managed Nodes.

Once the `Ansible-ControlNode` is up and running. Now, SSH into the machine using `MobaXterm` or `Putty` with the username `ubuntu` and continue Task-1:

[Click here](https://mobaxterm.mobatek.net/download-home-edition.html) to download MobaXterm (**Note:** Choose `Installer Edition` and install on your Laptop) 

1. **Set Hostname:**
Set the hostname as 'Control-Node'.
    ```sh
    sudo hostnamectl set-hostname Control-Node
    bash
    ```

2. **Update the Package Repository:**
    ```sh
    sudo yum check-update
    ```

3. **Install Python:**
    ```sh
    sudo yum install python3-pip wget unzip
    python3 --version
    sudo pip3 install --upgrade pip
    ```

4. **Configure AWS CLI:**
    Installing `AWS CLI`, boto, boto3.
```
sudo pip install boto boto3
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
Now, Check the AWS CLI Version
```
aws --version
```
5. **Updating and Installing Ansible:**
```
sudo yum update
```
```
pip3 install ansible==4.10.0
```
```
ansible --version
```
6. **For Authentication with AWS we need to provide `IAM User's CLI Credentials`**
```
aws configure
```
#### Credentials Example:
| **Access Key ID** | **Secret Access Key** |
| ----------------- | --------------------- |
| AKIAXMWJXSSHRD27T6SC | H4Vh0U5oenKfmJ/+FEUcbaGbDjcnGAmZvQLX7zTT |

---------------------------------------------------------------------
#### Once configured, do a smoke test to check if your credentials are valid and got the access to AWS account.

You can check using any one command or both.
```
aws s3 ls
```
(Or)
```
aws iam list-users
```
---------------------------------------------------------------------

7. **Create the Playbook for Creating Managed Nodes:**

Create a New file named `ec2-playbook.yml`:
```
vi ec2-playbook.yml
```

```  
---
- hosts: localhost
  connection: local

  tasks:
    - name: Execute curl command to get token
      shell: "curl -X PUT 'http://169.254.169.254/latest/api/token' -H 'X-aws-ec2-metadata-token-ttl-seconds: 21600'"
      register: TOKEN

    - name: Get region of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' http://169.254.169.254/latest/meta-data/placement/region/"
      register: region

    - name: Get AMI ID of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' http://169.254.169.254/latest/meta-data/ami-id"
      register: ami_id

    - name: Get keypair of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' http://169.254.169.254/latest/meta-data/public-keys/| cut -c 3-100 "
      register: kp

    - name: Get Instance Type of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' http://169.254.169.254/latest/meta-data/instance-type"
      register: instance_type


    - name: Get subnet id of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' -v http://169.254.169.254/latest/meta-data/network/interfaces/macs/$(curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' -v http://169.254.169.254/latest/meta-data/network/interfaces/macs)/subnet-id"
      register: subnet

    - name: Get security group of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' -v http://169.254.169.254/latest/meta-data/network/interfaces/macs/$(curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' -v http://169.254.169.254/latest/meta-data/network/interfaces/macs)/security-group-ids/"
      register: secgrp


    - name: Generate SSH keypair
      openssh_keypair:
        force: yes
        path: /home/ec2-user/.ssh/id_rsa

    - name: Get the public key
      shell: cat /home/ec2-user/.ssh/id_rsa.pub
      register: pubkey

    - name: Create EC2 instance
      ec2:
        key_name: "{{ kp.stdout }}"
        group_id: "{{ secgrp.stdout }}"
        instance_type: "{{ instance_type.stdout }}"
        image: "{{ ami_id.stdout }}"         # "ami-0931978297f275f71"
        wait: true
        region: "{{ region.stdout }}"
        instance_tags:
          Name: "{{ item }}"
        vpc_subnet_id: "{{ subnet.stdout }}"
        assign_public_ip: yes
        user_data: |
           #!/bin/bash
           echo "{{ pubkey.stdout }}" >> /home/ec2-user/.ssh/authorized_keys
      register: ec2var
      loop:
          - managed-node-1
          - managed-node-2

    - name: Make ansible directory
      file:
        path: /etc/ansible
        state: directory
      become: yes

    - debug:
        msg: "{{ ec2var.results[0].instances[0].private_ip }}"

    - debug:
        msg: "{{ ec2var.results[1].instances[0].private_ip }}"
  ```
   
9. **Run the Playbook:**
    ```sh
    ansible-playbook ec2-playbook.yml
    ```

10. **Add Managed Node IP Addresses:**
* Edit the hosts file:
    ```sh
    sudo vi /etc/ansible/hosts
    ```
    - Add the private IP addresses:
    ```plaintext
    node1 ansible_ssh_host=node1-private-ip ansible_ssh_user=ec2-user
    node2 ansible_ssh_host=node2-private-ip ansible_ssh_user=ec2-user
    ```
    - Example:
    ```plaintext
    node1 ansible_ssh_host=172.31.14.113 ansible_ssh_user=ec2-user
    node2 ansible_ssh_host=172.31.2.229 ansible_ssh_user=ec2-user
    ```

10. **List All Managed Node IP Addresses:**
    ```sh
    ansible all --list-hosts
    ```

11. **Set Hostnames for Managed Nodes:**
* SSH into each node and set the hostnames:
  - ssh ec2-user@<Replace Node 1 IP>
    ```sh
    sudo hostnamectl set-hostname managed-node-1
    bash
    ```
    exit from managed-node-1
    
  - ssh ec2-user@<Replace Node 2 IP>
    ```sh
    sudo hostnamectl set-hostname managed-node-2
    bash
    ```
    exit from managed-node-2

12. **Check Managed Nodes:**
    - Use the ping module to check if the managed nodes are able to interpret the Ansible modules:
    ```sh
    ansible all -m ping
    ```
