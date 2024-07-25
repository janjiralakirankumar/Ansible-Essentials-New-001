## Lab 2: Exploring Ad-Hoc Commands

1. **Update Ansible Hosts File:**
    - Edit the hosts file to include localhost and set the connection as local:
    ```sh
    sudo vi /etc/ansible/hosts
    ```
    - Add the following line by pressing `INSERT`:
    ```plaintext
    localhost ansible_connection=local
    ```
    - Save the file using `ESCAPE + :wq!`.

    > **Note:** In real-life scenarios, Ansible control node can also be used as a Managed node, It’s a convenient way to perform local operations, test playbooks, and manage the local system itself. you can include `localhost` in the hosts inventory file.

2. **Get Memory Details:**
    - Run the following ad-hoc command to get memory details of the hosts:
    ```sh
    ansible all -m command -a "free -h"
    ```
    - Alternatively:
    ```sh
    ansible all -a "free -h"
    ```

3. **Create a User on All Nodes:**
    - Create a user named `ansible-new` on all nodes, including the control node:
    ```sh
    ansible all -m user -a "name=ansible-new" --become
    ```

4. **Check for New User:**
    - List all users on `node1` to verify that `ansible-new` is present:
    ```sh
    ansible node1 -a "cat /etc/passwd"
    ```

5. **List Directories in /home:**
    - Ensure the directory `ansible-new` is present in `/home` on `node2`:
    ```sh
    ansible node2 -a "ls /home"
    ```

6. **Change Permissions:**
    - Change the permission mode of the new home directory to `755`:
    ```sh
    ansible node1 -m file -a "dest=/home/ansible-new mode=755" --become
    ```

7. **Verify Permissions:**
    - Check if the permissions have been updated:
    ```sh
    ansible node1 -a "sudo ls -l /home"
    ```

8. **Create a File in New Directory:**
    - Create a new file named `demo.txt` in `/home/ansible-new/` on `node1`:
    ```sh
    ansible node1 -m file -a "dest=/home/ansible-new/demo.txt mode=600 state=touch" --become
    ```

9. **Verify File Creation:**
    - Check if `demo.txt` was created successfully:
    ```sh
    ansible node1 -a "sudo ls -l /home/ansible-new/"
    ```

10. **Add Content to File:**
    - Add a line of text to `demo.txt`:
    ```sh
    ansible node1 -b -m lineinfile -a 'dest=/home/ansible-new/demo.txt line="This server is managed by Ansible"'
    ```

11. **Verify Content Addition:**
    - Check the content of `demo.txt` to confirm the line was added:
    ```sh
    ansible node1 -a "sudo cat /home/ansible-new/demo.txt"
    ```

12. **Remove Line from File:**
    - Remove the added line from `demo.txt`:
    ```sh
    ansible node1 -b -m lineinfile -a 'dest=/home/ansible-new/demo.txt line="This server is managed by Ansible" state=absent'
    ```

13. **Verify Line Removal:**
    - Check the content of `demo.txt` to confirm the line was removed:
    ```sh
    ansible node1 -b -a "sudo cat /home/ansible-new/demo.txt"
    ```

14. **Copy File to Managed Node:**
    - Create a file `test.txt` and copy it to `/home/ansible-new/` on `node1`:
    ```sh
    touch test.txt
    echo "This file will be copied to managed node using copy module" >> test.txt
    ansible node1 -m copy -a "src=test.txt dest=/home/ansible-new/test" -b
    ```

15. **Verify File Copy:**
    - Check if `test.txt` was copied successfully:
    ```sh
    ansible node1 -b -a "sudo ls -l /home/ansible-new/test"
    ```

16. **Remove Localhost Entry:**
    - Edit the hosts file to remove the localhost entry:
    ```sh
    sudo vi /etc/ansible/hosts
    ```
    - Remove the line:
    ```plaintext
    localhost ansible_connection=local
    ```
    - Save the file using `ESCAPE + :wq!`.

## Playbook

### Create a Playbook:**

Create a file named `first.yml`:

    ```yaml
    vi first.yml
    ```
Copy paste the below code into it and save (:wq!)
    
    ```yaml
    ---
    - name: first play
      hosts: all
      become: yes
      tasks:
        - name: create a directory
          file:
            path: /test
            state: directory
        - name: create a new file
          file:
            path: demo.txt
            mode: 0664
            state: touch
    ```

3. **Run the Playbook:**
    ```sh
    ansible-playbook first.yml
    ```

4. **List Files:**
    - Verify the files created by the playbook:
    ```sh
    ansible all -m command -a "ls -l"
    ```
### ============================ END of LAB ============================
