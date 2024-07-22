## Lab 6: Advanced Ansible Playbooks

### Task 1: Including Tasks and Conditional Execution

1. **Update Inventory File:**
    - Edit the `/etc/ansible/hosts` file to include:
    ```ini
    localhost ansible_connection=local
    ```

2. **Create the `first.yaml` Playbook:**
    - Create a file named `first.yaml` with the following content:
    ```yaml
    ---
    - hosts: localhost
      gather_facts: no
      become: yes
      tasks:
        - name: Install common packages
          yum:
            name: [wget, curl]
            state: present

        - name: Include task for httpd installation
          include_tasks: second.yaml
    ```

3. **Create the `second.yaml` Playbook:**
    - Create a file named `second.yaml` with the following content:
    ```yaml
    ---
    - name: Install the httpd package
      yum:
        name: httpd
        state: latest
        update_cache: yes

    - name: Start the httpd service
      service:
        name: httpd
        state: started
        enabled: yes
    ```

4. **Run the `first.yaml` Playbook:**
    ```sh
    ansible-playbook first.yaml
    ```

### Task 2: Conditional Task Execution

1. **Create the `conditions.yml` Playbook:**
    - Create a file named `conditions.yml` with the following content:
    ```yaml
    ---
    - name: Installing Httpd
      hosts: all
      become: yes
      tasks:
        - name: Install httpd on RedHat
          yum:
            name: httpd
            state: removed
          when: ansible_os_family == "RedHat"

        - name: Install nginx on Ubuntu
          apt:
            name: nginx
            state: present
          when: ansible_os_family == "Ubuntu"
    ```

2. **Run the `conditions.yml` Playbook:**
    ```sh
    ansible-playbook conditions.yml
    ```

### Task 3: Task Execution Based on Previous Task Results

1. **Create the `third.yaml` Playbook:**
    - Create a file named `third.yaml` with the following content:
    ```yaml
    ---
    - hosts: all
      gather_facts: no
      become: yes
      tasks:
        - name: Install common packages
          yum:
            name: [wget, curl]
            state: present
          register: out

        - name: List result of previous task
          debug:
            msg: "{{ out.rc }}"

        - name: Include task for httpd installation
          include_tasks: second.yaml
          when: out.rc == 0
    ```

2. **Run the `third.yaml` Playbook:**
    ```sh
    ansible-playbook third.yaml
    ```

### Task 4: Using Variables in Playbooks

1. **Create the `task1.yml` Playbook:**
    - Create a file named `task1.yml` with the following content:
    ```yaml
    ---
    - name: Demo Scripts
      hosts: all
      become: yes
      vars:
        list1:
          - yum
          - zip
          - unzip

        dict1:
          name: xyz
          age: 30

        os: windows

      tasks:
        - name: Simple Variables
          debug:
            msg: "Printing simple variable"

        - name: Print OS Variable
          debug:
            var: os

        - name: List Variables
          debug:
            msg:
              - item 0: "{{ list1[0] }}"
              - item 1: "{{ list1[1] }}"
              - item 2: "{{ list1[2] }}"

        - name: Print List Variable
          debug:
            var: list1

        - name: Dict Variables
          debug:
            msg:
              - "My name is {{ dict1['name'] }} and I am {{ dict1['age'] }} years old."

        - name: Print Dict Variable
          debug:
            var: dict1
    ```

2. **Run the `task1.yml` Playbook:**
    ```sh
    ansible-playbook task1.yml
    ```

3. **Create the `task2.yml` Playbook (Loop with `with_items`):**
    - Create a file named `task2.yml` with the following content:
    ```yaml
    ---
    - name: Install and Start the Service
      hosts: all
      become: yes
      vars:
        apps:
          - zip
          - unzip
          - httpd
          - vim
          - telnet

      tasks:
        - name: Installing Packages
          yum:
            name: "{{ item }}"
            state: present
          tags: i-loops
          with_items: "{{ apps }}"
    ```

4. **Run the `task2.yml` Playbook:**
    ```sh
    ansible-playbook task2.yml
    ```

5. **Create the `task3.yml` Playbook (Create Users Using Loop):**
    - Create a file named `task3.yml` with the following content:
    ```yaml
    ---
    - name: Create Set of Users
      hosts: all
      become: yes

      tasks:
        - name: Create Users from List
          user:
            name: "{{ item }}"
            state: present
          loop:
            - testuser1
            - testuser2
    ```

6. **Run the `task3.yml` Playbook:**
    ```sh
    ansible-playbook task3.yml
    ```

7. **Verify Users:**
    ```sh
    ansible all -a "tail -n 3 /etc/passwd"
    ```

### Task 5: Variables in External File

1. **Create the `task4.yml` Playbook:**
    - Create a file named `task4.yml` with the following content:
    ```yaml
    ---
    - name: Example External Variables File
      hosts: all
      vars_files:
        - ./variables.yml

      tasks:
        - name: Print the value of variable docker_version
          debug:
            msg: "{{ docker_version }}"

        - name: Print the value of group variable http_port
          debug:
            msg: "{{ http_port }}"

        - name: Print the value of host variable app_version
          debug:
            msg: "{{ app_version }}"
    ```

2. **Create the `variables.yml` File:**
    - Create a file named `variables.yml` with the following content:
    ```yaml
    ---
    docker_version: 20.10.2
    http_port: 2200
    app_version: v-1.1
    ```

3. **Run the `task4.yml` Playbook:**
    ```sh
    ansible-playbook task4.yml
    ```

### Task 6: User Management and Service Checks

1. **Create the `task6-1.yml` Playbook (Uninstall httpd):**
    - Create a file named `task6-1.yml` with the following content:
    ```yaml
    ---
    - hosts: all
      become: yes
      connection: ssh
      user: ec2-user
      tasks:
        - name: Uninstall Apache Web Server
          yum:
            name: httpd
            state: absent
    ```

2. **Run the `task6-1.yml` Playbook:**
    ```sh
    ansible-playbook task6-1.yml
    ```

3. **Create the `task6-2.yml` Playbook (Install httpd and Check Status):**
    - Create a file named `task6-2.yml` with the following content:
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

4. **Run the `task6-2.yml` Playbook:**
    ```sh
    ansible-playbook task6-2.yml
    ```

5. **Create the `task6-3.yml` Playbook (Install httpd and Start Service):**
    - Create a file named `task6-3.yml` with the following content:
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
        
        - name: Start Apache Web Server
          service:
            name: httpd
            state: started
        
        - name: Verify Status of Service
          shell: systemctl status httpd
          register: result
          until: result.stdout.find("active (running)") != -1
          retries: 5
          delay: 10
    ```

6. **Run the `task6-3.yml` Playbook:**
    ```sh
    ansible-playbook task6-3.yml
    ```
