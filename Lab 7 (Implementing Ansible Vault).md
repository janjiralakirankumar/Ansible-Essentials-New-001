## Lab 7: Implementing Ansible Vault

### Task 1: Encrypting and Managing Playbooks with Ansible Vault

1. **Switch to the Labs Directory:**
    ```sh
    cd ~/labs
    ```

2. **Create a Sample Playbook (`implement-vault.yml`):**
    ```yaml
    ---
    - hosts: all
      tasks:
        - file:
            path: /home/ec2-user/test.conf
            state: touch
            owner: ec2-user
            group: ec2-user
            mode: 0644
    ```

3. **Encrypt the Playbook:**
    ```sh
    ansible-vault encrypt implement-vault.yml
    ```

4. **Verify Encryption:**
    ```sh
    cat implement-vault.yml
    ```

5. **View Encrypted Playbook Contents in Plaintext:**
    ```sh
    ansible-vault view implement-vault.yml
    ```

6. **Execute the Encrypted Playbook:**
    ```sh
    ansible-playbook --ask-vault-pass implement-vault.yml
    ```

7. **Edit the Encrypted Playbook:**
    ```sh
    ansible-vault edit implement-vault.yml
    ```

8. **Execute the Playbook After Editing:**
    ```sh
    ansible-playbook --ask-vault-pass implement-vault.yml
    ```

9. **Change the Vault Password:**
    ```sh
    ansible-vault rekey implement-vault.yml
    ```

10. **Verify Encryption After Changing Password:**
    ```sh
    cat implement-vault.yml
    ```

11. **Decrypt the Playbook:**
    ```sh
    ansible-vault decrypt implement-vault.yml
    ```

12. **Verify Plaintext Content:**
    ```sh
    cat implement-vault.yml
    ```

### Task 2: Loops with Ansible Playbooks

1. **Create a Playbook with Loops (`looplab.yml`):**
    ```yaml
    ---
    - hosts: all
      become: yes
      tasks:
        - name: Creating users
          user:
            name: "{{ item }}"
            state: present
          with_items:
            - userX
            - userY
            - userZ
    ```

2. **Execute the Playbook:**
    ```sh
    ansible-playbook looplab.yml
    ```

3. **Verify Users with Ansible Ad-Hoc Command:**
    ```sh
    ansible all -a "tail -n 3 /etc/passwd"
    ```

### Task 3: Tags with Ansible Playbooks

1. **Create a Playbook with Tags (`tagslabs.yml`):**
    ```yaml
    ---
    - hosts: all
      become: yes
      user: ec2-user
      connection: ssh
      gather_facts: no
      tasks:
        - name: Install telnet
          yum: pkg=telnet state=latest
          tags:
            - packages
        - name: Verifying telnet installation
          raw: yum list installed | grep telnet > /home/ec2-user/pkg.log
          tags:
            - logging
    ```

2. **Execute the Playbook:**
    ```sh
    ansible-playbook tagslabs.yml
    ```

3. **Run the Playbook with Specific Tags:**
    ```sh
    ansible-playbook -t "logging" tagslabs.yml
    ansible-playbook -t "packages" tagslabs.yml
    ```

### Task 4: Prompts with Ansible Playbooks

1. **Create a Playbook with Prompts (`promptlab.yml`):**
    ```yaml
    ---
    - hosts: all
      become: yes
      user: ec2-user
      connection: ssh
      vars_prompt:
        - name: pkginstall
          prompt: Which package do you want to install?
          default: telnet
          private: no
      tasks:
        - name: Install the package specified
          yum: pkg={{ pkginstall }} state=latest
    ```

2. **Execute the Playbook:**
    ```sh
    ansible-playbook promptlab.yml
    ```

3. **Verify Installed Package:**
    ```sh
    ssh ec2-user@<managed_node_private_ip>
    rpm -qa | grep httpd
    ansible all -m "command" -a "rpm -qa | grep httpd"
    ```

### Task 5: Using the Until Function

1. **Create a Playbook Using `until` (`untillab.yml`):**
    ```yaml
    ---
    - hosts: all
      become: yes
      connection: ssh
      user: ec2-user
      tasks:
        - name: Install Apache Web Server
          yum:
            name: httpd
            state: latest
        - name: Verify Status of Service
          shell: systemctl status httpd
          register: result
          until: result.stdout.find("active (running)") != -1
          retries: 5
          delay: 10
    ```

2. **Execute the Playbook:**
    ```sh
    ansible-playbook untillab.yml
    ```

3. **Start the httpd Service on the Managed Node:**
    ```sh
    ssh ec2-user@<managed_node_private_ip>
    sudo service httpd start
    sudo service httpd status
    ```

### Task 6: Run Once with Ansible Playbooks

1. **Create a Playbook with `run_once` (`rolab.yml`):**
    ```yaml
    ---
    - hosts: all
      become: yes
      user: ec2-user
      connection: ssh
      gather_facts: no
      tasks:
        - name: Recording uptime
          raw: /usr/bin/uptime >> /home/ec2-user/uptime
          run_once: true
    ```

2. **Execute the Playbook:**
    ```sh
    ansible-playbook rolab.yml
    ```

3. **Verify the File Exists and Contains the Correct Content:**
    ```sh
    ansible all -a "cat /home/ec2-user/uptime"
    ```

4. **Change `run_once` to `false` and Re-Execute:**
    ```yaml
    # Edit rolab.yml and set run_once: false
    ansible-playbook rolab.yml
    ansible all -a "cat /home/ec2-user/uptime"
    ```

### Task 7: Blocks with Ansible Playbooks

1. **Create a Playbook with Blocks (`blklab.yml`):**
    ```yaml
    ---
    - hosts: all
      become: yes
      user: ec2-user
      connection: ssh
      gather_facts: no
      tasks:
        - block:
            - name: Install {{ web_package }} package
              yum:
                name: "{{ web_package }}"
                state: latest
          rescue:
            - name: Install {{ db_package }} package
              yum:
                name: "{{ db_package }}"
                state: latest
          always:
            - name: Start {{ db_service }} service
              service:
                name: "{{ db_service }}"
                state: started
      vars:
        web_package: http
        db_package: mariadb-server
        db_service: mariadb
    ```

2. **Execute the Playbook:**
    ```sh
    ansible-playbook blklab.yml
    ```

3. **Fix the Package Name and Re-Execute:**
    ```yaml
    # Edit blklab.yml and set web_package: httpd
    ansible-playbook blklab.yml
    ```

### Task 8: Working with Handlers

1. **Check for Existing Configuration File and Install httpd:**
    ```sh
    ls -l /etc/httpd/conf/httpd.conf
    ```

2. **Create Playbook to Install httpd (`install-httpd.yml`):**
    ```yaml
    ---
    - name: Install Apache Web Server
      hosts: all
      become: yes
      tasks:
        - name: Install httpd
          yum:
            name: httpd
            update_cache: yes
            state: latest
    ```

3. **Execute the Playbook:**
    ```sh
    ansible-playbook install-httpd.yml
    ```

4. **Prepare Configuration Files:**
    ```sh
    cp /etc/httpd/conf/httpd.conf ./httpd.conf
    cp ./httpd.conf ./httpd.conf.og
    # Edit httpd.conf and uncomment line for ErrorDocument
    ```

5. **Create Playbook to Stop httpd (`stophttpd.yml`):**
    ```yaml
    ---
    - name: Stop Apache Web Server
      hosts: all
      become: yes
      tasks:
        - name: Stop httpd
          service:
            name: httpd
            state: stopped
    ```

6. **Execute the Stop httpd Playbook:**
    ```sh
    ansible-playbook stophttpd.yml
    ```

7. **Create Playbook to Install and Configure Apache (`install-apache.yml`):**
    ```yaml
    ---
    - name: Install and Configure Apache
      hosts: all
      become: yes
      tasks:
        - name: Install httpd
          yum:
            name: httpd
            update_cache: yes
            state: latest
        - name: Upload custom index.html
          copy:
            src: "index.html"
            dest: "/var/www/html/index.html"
        - name: Setup file attributes
          file:
            path: /var/www/html/index.html
            owner: apache
            group: apache
            mode: 0644
        - name: Upload httpd.conf.og
          copy:
            src: "httpd.conf.og"
            dest: "/etc/httpd/conf/httpd.conf"
        - name: Start httpd
          service:
            name: httpd
            state: started
    ```

8. **Execute the Playbook:**
    ```sh
    ansible-playbook install-apache.yml
    ```

9. **Create Playbook with Handlers (`handler.yml`):**
    ```yaml
    ---
    - name: Handler Demo
      hosts: all
      become: yes
      tasks:
        - name: Copy httpd.conf
          copy:
            src: "httpd.conf"
            dest: /etc/httpd/conf/httpd.conf
          notify:
            - restart httpd
        - name: Upload custom error.html
          copy:
            src: "missing.html"
            dest: "/var/www/html/missing.html"
      handlers:
        - name: restart httpd
          service:
            name: httpd
            state: restarted
    ```

10. **Execute the Handler Playbook:**
    ```sh
    ansible-playbook handler.yml
    ```

11. **Re-Run the Playbook to Check Handler Execution:**
    ```sh
    ansible-playbook handler.yml
    ```
