# Ansible Deployment

## Pre-requisites
- Start an Ubuntu VM with the username `user` and password `pass`

- Install SSH server on the VM:
   ```commandline
   sudo apt-get install openssh-server
   sudo systemctl enable ssh
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
   [my-vm]
   control-plane-1 ansible_host=192.168.0.146
   ```
- Replace the `sudo` password in `host_vars/control-plane-1.yaml`

- Run playbook
   ```commandline
   ansible-playbook setup_playbook.yml
   ```
