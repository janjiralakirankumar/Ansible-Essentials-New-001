## Lab 9: Implementing Ansible Roles

### Task 1: Implementing Ansible Roles

1. **Uninstall httpd (if installed):**
    ```sh
    ansible-playbook /home/ec2-user/ansible-labs/uninstall-apache-pb.yml
    ```

2. **Install `tree` utility (for viewing directory structure):**
    ```sh
    sudo yum install tree -y
    ```

3. **Create Role Directories and Files:**

    ```sh
    mkdir role-labs && cd role-labs
    mkdir webrole dbrole && cd dbrole
    mkdir tasks
    ```

4. **Create `main.yml` for `dbrole`:**
    ```yaml
    ---
    - name: Install MariaDB server package
      yum:
        name: mariadb-server
        state: present
    - name: Start MariaDB Service
      service:
        name: mariadb
        state: started
        enabled: true
    ```

5. **Create `index.html` file in `webrole`:**
    ```html
    <html>
      <body>
      <div align="center">
      <h1>We are performing the Roles Lab</h1>
      <img width="50%" src="https://miro.medium.com/v2/resize:fit:678/1*MtmOHEt8ZX7s5KxV6bFSUg.png">
      </div>
      </body>
    </html>
    ```

6. **Create `main.yml` for `webrole`:**
    ```yaml
    ---
    - name: install httpd
      yum:
        name: httpd
        update_cache: yes
        state: latest

    - name: uploading default index.html for host
      copy:
        src: files/index.html
        dest: /var/www/html/index.html

    - name: Setting up attributes for file
      file:
        path: /var/www/html/index.html
        owner: apache
        group: apache
        mode: 0644

    - name: start httpd
      service:
        name: httpd
        state: started
    ```

7. **Verify Role Structure:**
    ```sh
    cd ../..
    tree
    ```

8. **Create the Playbook to Implement Roles (`implement-roles.yml`):**
    ```yaml
    ---
    - hosts: all
      become: yes
      roles:
        - webrole
        - dbrole
    ```

9. **Execute the Playbook:**
    ```sh
    ansible-playbook implement-roles.yml
    ```

10. **Check Home Page on Browser:**
    Access the public DNS of the managed node to view the page with the message "We are performing the Roles Lab".

### Task 2: Installing Java through Ansible Galaxy Roles

1. **Install Java Role from Ansible Galaxy:**
    ```sh
    ansible-galaxy install geerlingguy.java
    ```

2. **Create a Playbook to Install Java (`implement-java.yml`):**
    ```yaml
    ---
    - hosts: all
      become: yes
      roles:
        - geerlingguy.java
    ```

3. **Check Java Installation:**
    ```sh
    ansible all -m command -a "java -version"
    ```

4. **Execute the Playbook:**
    ```sh
    ansible-playbook implement-java.yml
    ```

5. **Verify Java Installation:**
    ```sh
    ansible all -a "java -version"
    ```
