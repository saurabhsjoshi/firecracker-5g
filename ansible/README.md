# Ansible Deployment

## Supported Operating System
- This Ansible script has only been tested on Ubuntu 20.04 LTS host and guest 

## Pre-requisites
- Start an Ubuntu VM with the username `user` and password `pass`

- Install SSH server on the VM:
   ```commandline
   sudo apt-get install openssh-server && \
   sudo systemctl start ssh
   ```
   
- Find the IP address of the VM using `ip addr`

- Copy the SSH id on the host:
   ```commandline
   ssh-copy-id user@<VM-IP>
   sudo apt-get install sshpass
   ```
- Replace the IP address in inventory file with your VM's IP
   ```yaml
   [server]
   server ansible_host=192.168.0.146
   ```
- Replace the `sudo` password in `host_vars/server.yaml`

- Install external roles and collections
  ```commandline
  ansible-galaxy install -p external/ -r requirements.yml
  ansible-galaxy collection install -p external/ -r requirements.yml 
  ```

- Run playbook
   ```commandline
   ansible-playbook 5g-firecracker.yml
   ```

- Stop and delete VM(s)
  ```commandline
  ansible-playbook purge.yml
  ```
