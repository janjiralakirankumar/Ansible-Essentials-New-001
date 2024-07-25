## Lab 5: Implementing Ansible Variables

### Task 1: Configuring Packages in Ansible Using Variables

#### 1. Create the Ansible Playbook:

Create a file named `implement-vars.yml` with the following content:

```
vi implement-vars.yml
```
```yaml
---
- hosts: '{{ hostname }}'
  become: yes
  vars:
    hostname: all
    package1: httpd
    destination: /var/www/html/index.html
    source: /home/ec2-user/lab5/file/index.html  # Control node path
  tasks:
    - name: Install defined package
      yum:
        name: '{{ package1 }}'
        update_cache: yes   # Refresh the cache before applying changes
        state: latest
    
    - name: Start desired service
      service:
        name: '{{ package1 }}'
        state: started
    
    - name: Copy required index.html to the document folder for httpd
      copy:
        src: '{{ source }}'
        dest: '{{ destination }}'
```

2. **Create the `index.html` File:**

Create a file named `index.html` with the following content:

```
vi index.html
```
```html
<html>
  <body>
  <h1>Welcome to CloudThat</h1>
  <h1>Welcome to variables</h1>
  </body>
</html>
```

#### 3. Run the Ansible Playbook:

```sh
ansible-playbook implement-vars.yml
```

#### 4. Verify the Changes:

- View the page using the public IP address of the VM.

### Task 2: Implementing Ansible Variables Using `--extra-vars` Option

#### 1. Create a New `index1.html` File:

Create a file named `index1.html` with the following content:

```
vi index1.html
```
```html
<html>
  <body>
  <h1>This is the alternate Home Page</h1>
  <img width="100%" src="https://cdn-labgd.nitrocdn.com/DgsEbCQFApREClXUXMwcDAPWJfHtBIby/assets/images/optimized/rev-f4df46d/content.cloudthat.com/consulting/wp-content/uploads/2023/11/30110123/Banner__Homepage_-Superstar-Award1.webp">
  </body>
</html>
```

#### 2. Run the Ansible Playbook with Extra Variables:

```sh
ansible-playbook implement-vars.yml --extra-vars "source=/home/ec2-user/lab5/file/index1.html"
```

### Task 3: Configuring Variables as a Separate File and Implementing Ansible Playbook

#### 1. Create the Ansible Playbook:

Create a file named `implement-vars1.yml` with the following content:

```
vi implement-vars1.yml
```
```yaml
---
- hosts: '{{ hostname }}'
  become: yes
  vars_files:
    - myvariables.yml
  tasks:
    - name: Install defined package
      yum:
        name: '{{ package1 }}'
        update_cache: yes
        state: latest
    
    - name: Start desired service
      service:
        name: '{{ package1 }}'
        state: started
    
    - name: Copy required index.html to the document folder for httpd
      copy:
        src: '{{ source }}'
        dest: '{{ destination }}'
```

#### 2. Create the Variables File:

Create a file named `myvariables.yml` with the following content:

```
vi myvariables.yml
```
```yaml
---
hostname: all
package1: httpd
destination: /var/www/html/index.html
source: /home/ec2-user/lab5/file/index.html
```

#### 3. Create or Update the `index.html` File:

Ensure `index.html` contains:

```
vi index.html
```
```html
<html>
  <body>
  <h1>Welcome to CloudThat</h1>
  <h1>Welcome to variables</h1>
  </body>
</html>
```

#### 4. Run the Ansible Playbook:

```sh
ansible-playbook implement-vars1.yml
```

#### ============================= END of LAB  =============================
