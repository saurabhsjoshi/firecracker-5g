Initial setup:

start an ubuntu VM

In the VM:
   Install and start SSH server:
   $ sudo apt-get install openssh-server
   $ sudo systemctl enable ssh
   $ sudo systemctl start ssh

   Take note of the IP address using:
   $ ipaddr

On the Host: 
   ssh-copy-id <username>@<VM-IP>
   sudo apt-get install sshpass

   add the following line in /etc/ansible/hosts: 
   <VM-IP> ansible_ssh_pass=<password> ansible_ssh_user=<username> ansible_become_password=<password>


To run:
$ ansible-playbook  setup_playbook.yml