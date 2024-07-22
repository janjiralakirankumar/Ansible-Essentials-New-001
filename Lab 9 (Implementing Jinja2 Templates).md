## Lab 9: Implementing Jinja2 Templates

### Steps:

1. **Create a Directory for Templates:**
    ```sh
    mkdir templates
    cd templates
    ```

2. **Create a Jinja2 Template File (`new-motd.j2`):**
    ```jinja
    System information is as below
    Operatingsystem: '{{ ansible_os_family }}'
    Architecture: '{{ ansible_architecture }}'
    DistributionVersion: '{{ ansible_distribution }}'
    IP Address: '{{ ansible_eth0.ipv4.address }}'
    ```

3. **Create a Playbook to Implement Jinja2 Templates (`implement-jinja2.yml`):**
    ```yaml
    ---
    - hosts: all
      become: yes
      tasks:
        - template:
            src: new-motd.j2
            dest: /etc/motd
            owner: ec2-user
            group: ec2-user
            mode: 0644
    ```

4. **Execute the Playbook:**
    ```sh
    ansible-playbook implement-jinja2.yml -b
    ```

5. **Verify the Template Implementation:**
    ```sh
    ansible all -m shell -a "cat /etc/motd"
    ```

### Summary:

This lab demonstrates how to use Jinja2 templates in Ansible to generate configuration files dynamically. The template file `new-motd.j2` is used to create a dynamic `/etc/motd` file on all hosts, which contains system information.
