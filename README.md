# Automate Node.js Application Deployment

## Project Overview
This project demonstrates how to automate the deployment of a Node.js application using Ansible on an AWS EC2 instance running Linux. The process includes creating a server, setting up necessary technologies, creating a dedicated Linux user, and deploying the application under that user.

## Technologies Used
- **Ansible**: For automating application deployment
- **Node.js**: Backend application runtime
- **AWS EC2**: Cloud instance hosting the application
- **Linux**: Operating system for the EC2 instance

## Project Objectives
1. Provision an AWS EC2 instance.
2. Configure the instance by installing Node.js and npm using Ansible.
3. Create a Linux user specifically for deploying and running the Node.js application.
4. Deploy and run the Node.js application using Ansible tasks.

## Prerequisites
1. An AWS account with permissions to create EC2 instances.
2. SSH access to the EC2 instance with a private key file.
3. Installed tools:
   - Ansible
   - AWS CLI
4. A Node.js application packaged as `nodejs-app-1.0.0.tgz`.

## File Structure
```
.
├── ansible.cfg
├── deploy-node.yaml
├── hosts
├── nodejs-app-1.0.0.tgz
```

## Ansible Configuration Files

### 1. `ansible.cfg`
```ini
[defaults]
host_key_checking = False
```

### 2. `hosts`
```ini
[ec2]
44.222.168.148 ansible_python_interpreter=/usr/bin/python3.9

[ec2:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_user=ec2-user
```

## Ansible Playbook

### `deploy-node.yaml`
```yaml
---
- name: Install node and npm
  hosts: 44.222.168.148
  tasks:
    - name: yum update
      yum: update_cache=yes update_only=yes
    - name: install nodejs and npm
      yum: 
        name: nodejs
        state: present
    - name: Verify Node.js installation
      command: node --version
    - name: Verify npm installation
      command: npm --version

- name: Create new linux user for node app
  hosts: 44.222.168.148
  tasks:
    - name: Ensure the admin group exists
      group:
        name: admin
        state: present
    - name: Create linux user
      user:
        name: nodeadmin
        comment: Node Administrator
        group: admin

- name: Deploy nodejs app
  hosts: 44.222.168.148
  become: True
  become_user: nodeadmin
  tasks:
    - name: Unpack the the nodejs file
      unarchive: 
        src: /root/ansible/nodejs-app-1.0.0.tgz
        dest: /home/nodeadmin
    - name: Install dependencies
      npm:
        path: /home/nodeadmin/package
    - name: Start the application
      command:
        chdir: /home/nodeadmin/package/app
        cmd: node server
      async: 1000
      poll: 0
    - name: Ensure app is running
      shell: ps aux | grep node
      register: app_status
    - debug: msg={{app_status.stdout_lines}}
```

## Steps to Deploy

### 1. Create an AWS EC2 Instance
- Use the AWS Management Console or CLI to launch a new EC2 instance.
- Use an existing key pair or generate a new one for SSH access.
- Note the public IP address of the instance.

### 2. Configure Ansible
- Update `hosts` file with the EC2 instance IP address.
- Ensure the private key file path and `ansible_user` are correctly set.

### 3. Run the Playbook
- Navigate to the project directory.
- Execute the Ansible playbook:
  ```bash
  ansible-playbook -i hosts deploy-node.yaml

  PLAY [Install node and npm] **************************************************************************************************************************

  TASK [Gathering Facts] *******************************************************************************************************************************
  ok: [44.222.168.148]
  
  TASK [yum update] ************************************************************************************************************************************
  ok: [44.222.168.148]
  
  TASK [install nodejs and npm] ************************************************************************************************************************
  ok: [44.222.168.148]
  
  TASK [Verify Node.js installation] *******************************************************************************************************************
  changed: [44.222.168.148]
  
  TASK [Verify npm installation] ***********************************************************************************************************************
  changed: [44.222.168.148]
  
  PLAY [Create new linux user for node app] ************************************************************************************************************
  
  TASK [Gathering Facts] *******************************************************************************************************************************
  ok: [44.222.168.148]
  
  TASK [Ensure the admin group exists] *****************************************************************************************************************
  changed: [44.222.168.148]
  
  TASK [Create linux user] *****************************************************************************************************************************
  changed: [44.222.168.148]
  
  PLAY [Deploy nodejs app] *****************************************************************************************************************************
  
  TASK [Gathering Facts] *******************************************************************************************************************************
  [WARNING]: Module remote_tmp /home/nodeadmin/.ansible/tmp did not exist and was created with a mode of 0700, this may cause issues when running as
  another user. To avoid this, create the remote_tmp dir with the correct permissions manually
  ok: [44.222.168.148]
  
  TASK [Unpack the the nodejs file] ********************************************************************************************************************
  changed: [44.222.168.148]
  
  TASK [Install dependencies] **************************************************************************************************************************
  changed: [44.222.168.148]
  
  TASK [Start the application] *************************************************************************************************************************
  changed: [44.222.168.148]
  
  TASK [Ensure app is running] *************************************************************************************************************************
  changed: [44.222.168.148]
  
  TASK [debug] *****************************************************************************************************************************************
  ok: [44.222.168.148] => {
      "msg": [
          "root        1043  0.0  0.0      0     0 ?        I<   09:22   0:00 [xfs-inodegc/xvd]",
          "nodeadm+   37287  0.0  1.5 245140 15536 ?        S    16:03   0:00 /usr/bin/python3.9 /var/tmp/ansible-tmp-1733241814.902588-42560-56985118023843/async_wrapper.py j67089433426 1000 /var/tmp/ansible-tmp-1733241814.902588-42560-56985118023843/AnsiballZ_command.py _",
          "nodeadm+   37288  0.0  1.6 245140 15856 ?        S    16:03   0:00 /usr/bin/python3.9 /var/tmp/ansible-tmp-1733241814.902588-42560-56985118023843/async_wrapper.py j67089433426 1000 /var/tmp/ansible-tmp-1733241814.902588-42560-56985118023843/AnsiballZ_command.py _",
          "nodeadm+   37289  0.0  2.3 244968 22444 ?        S    16:03   0:00 /usr/bin/python3.9 /var/tmp/ansible-tmp-1733241814.902588-42560-56985118023843/AnsiballZ_command.py",
          "nodeadm+   37306  0.0  5.7 606400 55932 ?        Sl   16:03   0:00 node server",
          "ec2-user   37371  0.0  0.3 222952  3200 pts/1    Ss+  16:03   0:00 /bin/sh -c sudo -H -S -n  -u nodeadmin /bin/sh -c 'echo BECOME-SUCCESS-quprlyaiunycwimydzaabcotbucnzvwu ; /usr/bin/python3.9 /var/tmp/ansible-tmp-1733241815.6741688-42570-252260115051510/AnsiballZ_command.py' && sleep 0",
          "root       37390  0.0  0.8 233296  7844 pts/1    S+   16:03   0:00 sudo -H -S -n -u nodeadmin /bin/sh -c echo BECOME-SUCCESS-quprlyaiunycwimydzaabcotbucnzvwu ; /usr/bin/python3.9 /var/tmp/ansible-tmp-1733241815.6741688-42570-252260115051510/AnsiballZ_command.py",
          "root       37392  0.0  0.2 233296  2500 pts/2    Ss   16:03   0:00 sudo -H -S -n -u nodeadmin /bin/sh -c echo BECOME-SUCCESS-quprlyaiunycwimydzaabcotbucnzvwu ; /usr/bin/python3.9 /var/tmp/ansible-tmp-1733241815.6741688-42570-252260115051510/AnsiballZ_command.py",
          "nodeadm+   37393  0.0  2.2 244960 22204 pts/2    S+   16:03   0:00 /usr/bin/python3.9 /var/tmp/ansible-tmp-1733241815.6741688-42570-252260115051510/AnsiballZ_command.py",
          "nodeadm+   37394  0.0  0.3 222952  3216 pts/2    S+   16:03   0:00 /bin/sh -c ps aux | grep node",
          "nodeadm+   37395  0.0  0.2 223492  2812 pts/2    R+   16:03   0:00 ps aux",
          "nodeadm+   37396  0.0  0.2 222316  2052 pts/2    S+   16:03   0:00 grep node"
      ]
  }
  
  PLAY RECAP *******************************************************************************************************************************************
  44.222.168.148             : ok=14   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
  
  
  
  
    ```

### 4. Verify Deployment
- SSH into the EC2 instance:
  ```bash
  ssh -i ~/.ssh/id_rsa ec2-user@44.222.168.148
  ```
- Check the running processes to verify the Node.js application is active:
  ```bash
  ps aux | grep node
  ```


