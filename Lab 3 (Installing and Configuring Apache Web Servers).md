## Lab 3: Installing and Configuring Apache Web Servers

1. **Create the Ansible Playbook:**
    - Create a file named `install-apache.yml` with the following content:
    ```yaml
    ---
    - name: This play will install Apache web servers on all the hosts
      hosts: all
      become: yes
      tasks:
        - name: Install httpd using yum
          yum:
            name: httpd
            update_cache: yes
            state: latest
        
        - name: Upload custom index.html to all hosts
          copy:
            src: "index.html"
            dest: "/var/www/html/index.html"
        
        - name: Set attributes for index.html
          file:
            path: /var/www/html/index.html
            owner: apache
            group: apache
            mode: 0644
        
        - name: Start the httpd service
          service:
            name: httpd
            state: started
    ```

2. **Create the `index.html` File:**
    - Create a file named `index.html` with the following content:
    ```html
    <html>
      <body>
      <h1>Welcome to CloudThat</h1>
      <img width="100%" src="https://github.com/user-attachments/assets/7045cd76-2b90-44da-a012-004382a4ef77">
      </body>
    </html>
    ```

3. **Run the Ansible Playbook:**
    ```sh
    ansible-playbook install-apache.yml
    ```

4. **Verify Apache Installation:**
    - Use `curl` to check if Apache is serving the `index.html` page:
    ```sh
    curl http://<private-ip-of-vm-1>
    curl http://<private-ip-of-vm-2>
    ```

    Replace `<private-ip-of-vm-1>` and `<private-ip-of-vm-2>` with the actual private IP addresses of your VMs.
