## Lab 4: User and Directory Management

### Part 1: User and Directory Creation

#### 1. Create the Ansible Playbook:

Create a file named `putfile.yml` with the following content:

```
vi putfile.yml
```
```yaml
---
- hosts: all
  become: yes
  tasks:
    - name: Create a new user cloudthat
      user:
        name: cloudthat
    
    - name: Create a directory for the new user
      file:
        path: /home/cloudthat/test
        state: directory
```

#### 2. Run the Ansible Playbook:

```sh
ansible-playbook putfile.yml
```

#### 3. Verify User and Directory Creation:

Check the latest entries in `/etc/passwd` to verify user creation:

```sh
ansible all -m command -a "tail -n 2 /etc/passwd"
```
- List the directory to verify it was created:
```sh
ansible all -m command -a "ls -l /home/cloudthat" -b
```

### Part 2: Advanced User and Directory Management

1. **Create the Ansible Playbook:**

Create a file named `p2.yml` with the following content:

```
vi p2.yml
```
```yaml
---
- hosts: all
  become: yes
  tasks:
    - name: Create a new user test
      user:
        name: test
    
    - name: Create a directory for the new user
      file:
        path: /home/test/demo
        state: directory
    
    - name: Create a folder named ansible
      file:
        path: /home/test/ansible
        state: directory
    
    - name: Create a file within the ansible folder
      file:
        path: /home/test/ansible/hello.txt
        state: touch
    
    - name: Change owner, group, and permissions for the file within the ansible folder
      file:
        path: /home/test/ansible/hello.txt
        owner: root
        group: test
        mode: 0665
    
    - name: Add a block of text to the file hello.txt
      blockinfile:
        path: /home/test/ansible/hello.txt
        block: |
          This is line 1
          This is line 2
```

#### 2. Run the Ansible Playbook:
```sh
ansible-playbook p2.yml
```

#### 3. Verify File Contents:

Check the contents of `hello.txt` to ensure the block was added:

```sh
ansible all -a "sudo cat /home/test/ansible/hello.txt"
```

## Task 2: Uninstalling Apache Service

1. **Create the Ansible Playbook:**

Create a file named `service.yml` with the following content:

```
vi service.yml
```
```yaml
---
- hosts: all
  become: yes
  tasks:
    - name: Uninstall httpd
      yum:
        name: httpd
        state: absent
    
    - name: Download a file
      get_url:
        url: https://s3.ap-south-1.amazonaws.com/files.cloudthat.training/devops/ansible-essentials/sql_permissions.txt
        dest: /tmp/
    
    - name: Disable SELinux
      selinux:
        state: disabled
```

#### 2. Run the Ansible Playbook:

```sh
ansible-playbook service.yml
```

#### 3. Verify Actions:

1. Check if `httpd` has been uninstalled:

```sh
ansible all -m command -a "yum list httpd" -b
```

2. Verify the file was downloaded:

```sh
ansible all -m command -a "ls -l /tmp" -b
```

3. Check the SELinux status:

```sh
ansible all -m command -a "getenforce" -b
```

#### ============================== END of LAB ==============================
