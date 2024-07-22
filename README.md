```markdown
# Ansible Labs

## Login to AWS Console

### Lab 1: Installation and Configuration of Ansible

1. **Launch EC2 Instance:**
    - Launch a RHEL 9 machine in `us-east-1`.
    - Choose `t2.micro`.
    - In the security group, allow SSH (22) and HTTP (80) for all incoming traffic.
    - Add Tag Name: `Ansible-ControlNode`.

2. **Set Hostname:**
    - Once the EC2 instance is up & running, SSH into it and set the hostname as 'Control-Node'.
    ```sh
    sudo hostnamectl set-hostname Control-Node
    ```
    - Exit and login again to see the new hostname or type `bash` to open another shell which shows the new hostname.

3. **Update the Package Repository:**
    ```sh
    sudo yum check-update
    ```

4. **Install Python:**
    ```sh
    sudo yum install python3-pip wget
    python3 --version
    sudo pip3 install --upgrade pip
    ```

5. **Install AWS CLI, Boto, Boto3, and Ansible:**
    ```sh
    sudo pip3 install awscli boto boto3
    sudo pip3 install ansible==4.10.0
    pip show ansible
    ```

6. **Configure AWS CLI:**
    ```sh
    aws configure
    ```
    - Add AWS Access Key and AWS Secret Access Key.

7. **Create the Playbook for Creating Managed Nodes:**
    - Create a file named `ec2-playbook.yml`:
    ```yaml
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

8. **Run the Playbook:**
    ```sh
    ansible-playbook ec2-playbook.yml
    ```

9. **Add Managed Node IP Addresses:**
    - Edit the hosts file:
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
    - SSH into each node and set the hostnames:
    ```sh
    ssh ec2-user@<Replace Node 1 IP>
    sudo hostnamectl set-hostname managed-node-1
    exit

    ssh ec2-user@<Replace Node 2 IP>
    sudo hostnamectl set-hostname managed-node-2
    exit
    ```

12. **Check Managed Nodes:**
    - Use the ping module to check if the managed nodes are able to interpret the Ansible modules:
    ```sh
    ansible all -m ping
    ```
```
