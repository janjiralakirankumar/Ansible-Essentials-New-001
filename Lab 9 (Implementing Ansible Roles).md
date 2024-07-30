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

---

## Self Practice: Configuring HTTPS Listener in Windows

**Note:** Replace placeholders with actual values based on your setup.

1. **Create a Certificate:**
    ```powershell
    New-SelfSignedCertificate -DnsName "DNS Name" -CertStoreLocation Cert:\LocalMachine\My
    ```

2. **Add Ports to Security Group:**
    Ensure ports `5985` (winrm-http) and `5986` (winrm-https) are open in the security group of the Windows server.

3. **Create HTTPS Listener:**
    ```powershell
    winrm create winrm/config/Listener?Address=*+Transport=HTTPS '@{Hostname="DNS Name"; CertificateThumbprint="Thumbprint"}'
    ```

4. **Add Firewall Rule for Port 5986:**
    ```powershell
    netsh advfirewall firewall add rule name="Windows Remote Management (HTTPS-In)" dir=in action=allow protocol=TCP localport=5986
    ```

5. **Check the Listener and Service:**
    ```powershell
    winrm e winrm/config/Listener
    winrm get winrm/config
    ```

6. **Set Basic Authentication:**
    ```powershell
    Set-Item -Force WSMan:\localhost\Service\auth\Basic $true
    ```

**On Ansible Control Node:**

1. **Update Inventory File:**
    ```ini
    [windows]
    windows ansible_host=172.31.44.232 ansible_password=YOUR_PASSWORD ansible_connection=winrm ansible_port=5986 ansible_user=administrator ansible_winrm_server_cert_validation=ignore ansible_winrm_transport=basic
    ```

2. **Ping Windows Server:**
    ```sh
    ansible windows -m win_ping
    ```
